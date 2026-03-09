# gRPC Interceptor Implementations

## Logging Interceptor (`internal/interceptor/logging.go`)

```go
package interceptor

import (
	"context"
	"time"

	"go.uber.org/zap"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
)

func Logging() grpc.UnaryServerInterceptor {
	logger, _ := zap.NewProduction()

	return func(
		ctx context.Context,
		req interface{},
		info *grpc.UnaryServerInfo,
		handler grpc.UnaryHandler,
	) (interface{}, error) {
		start := time.Now()

		// Extract request ID from incoming metadata
		requestID := "unknown"
		if md, ok := metadata.FromIncomingContext(ctx); ok {
			if vals := md.Get("x-request-id"); len(vals) > 0 {
				requestID = vals[0]
			}
		}

		resp, err := handler(ctx, req)

		code := codes.OK
		if err != nil {
			code = status.Code(err)
		}

		logger.Info("grpc request",
			zap.String("method", info.FullMethod),
			zap.String("request_id", requestID),
			zap.Duration("duration", time.Since(start)),
			zap.String("code", code.String()),
			zap.Error(err),
		)

		return resp, err
	}
}
```

## Recovery Interceptor (`internal/interceptor/recovery.go`)

```go
package interceptor

import (
	"context"
	"fmt"
	"runtime/debug"

	"go.uber.org/zap"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
)

func Recovery() grpc.UnaryServerInterceptor {
	logger, _ := zap.NewProduction()

	return func(
		ctx context.Context,
		req interface{},
		info *grpc.UnaryServerInfo,
		handler grpc.UnaryHandler,
	) (resp interface{}, err error) {
		defer func() {
			if r := recover(); r != nil {
				logger.Error("panic recovered",
					zap.String("method", info.FullMethod),
					zap.Any("panic", r),
					zap.ByteString("stack", debug.Stack()),
				)
				err = status.Errorf(codes.Internal, "internal server error: %v", fmt.Sprintf("%v", r))
			}
		}()
		return handler(ctx, req)
	}
}
```

## Auth Interceptor (`internal/interceptor/auth.go`)

```go
package interceptor

import (
	"context"
	"strings"

	"github.com/golang-jwt/jwt/v5"
	"google.golang.org/grpc"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/metadata"
	"google.golang.org/grpc/status"
)

// SkippedMethods lists RPC methods that bypass auth (e.g. health checks)
var SkippedMethods = map[string]bool{
	"/grpc.health.v1.Health/Check": true,
}

type contextKey string
const UserIDKey contextKey = "user_id"

func Auth(jwtSecret string) grpc.UnaryServerInterceptor {
	return func(
		ctx context.Context,
		req interface{},
		info *grpc.UnaryServerInfo,
		handler grpc.UnaryHandler,
	) (interface{}, error) {
		if SkippedMethods[info.FullMethod] {
			return handler(ctx, req)
		}

		md, ok := metadata.FromIncomingContext(ctx)
		if !ok {
			return nil, status.Error(codes.Unauthenticated, "missing metadata")
		}

		authHeaders := md.Get("authorization")
		if len(authHeaders) == 0 {
			return nil, status.Error(codes.Unauthenticated, "missing authorization header")
		}

		tokenStr := strings.TrimPrefix(authHeaders[0], "Bearer ")
		claims := jwt.MapClaims{}
		_, err := jwt.ParseWithClaims(tokenStr, claims, func(t *jwt.Token) (interface{}, error) {
			return []byte(jwtSecret), nil
		})
		if err != nil {
			return nil, status.Error(codes.Unauthenticated, "invalid token")
		}

		userID, ok := claims["sub"].(string)
		if !ok {
			return nil, status.Error(codes.Unauthenticated, "invalid token claims")
		}

		ctx = context.WithValue(ctx, UserIDKey, userID)
		return handler(ctx, req)
	}
}

// UserIDFromContext extracts user ID from context (set by Auth interceptor)
func UserIDFromContext(ctx context.Context) (string, bool) {
	v, ok := ctx.Value(UserIDKey).(string)
	return v, ok
}
```
