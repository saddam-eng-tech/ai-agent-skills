# PostgreSQL Repository Implementation

## `internal/repository/postgres/<entity>_repo.go`

```go
package postgres

import (
	"context"
	"errors"
	"fmt"

	"github.com/jmoiron/sqlx"
	_ "github.com/lib/pq"

	"<module>/internal/domain"
	"<module>/internal/service"
	domainerrors "<module>/pkg/errors"
)

type <entity>Repo struct {
	db *sqlx.DB
}

func New<Entity>Repo(db *sqlx.DB) *<entity>Repo {
	return &<entity>Repo{db: db}
}

func (r *<entity>Repo) Create(ctx context.Context, e *domain.<Entity>) (*domain.<Entity>, error) {
	query := `
		INSERT INTO <entity>s (id, user_id, title, description, price, status, created_at, updated_at)
		VALUES (:id, :user_id, :title, :description, :price, :status, :created_at, :updated_at)
		RETURNING *`

	rows, err := r.db.NamedQueryContext(ctx, query, e)
	if err != nil {
		return nil, fmt.Errorf("create <entity>: %w", err)
	}
	defer rows.Close()

	if rows.Next() {
		var created domain.<Entity>
		if err := rows.StructScan(&created); err != nil {
			return nil, err
		}
		return &created, nil
	}
	return nil, domainerrors.ErrNotFound
}

func (r *<entity>Repo) FindByID(ctx context.Context, id string) (*domain.<Entity>, error) {
	var e domain.<Entity>
	err := r.db.GetContext(ctx, &e,
		`SELECT * FROM <entity>s WHERE id = $1 AND deleted_at IS NULL`, id)
	if err != nil {
		if errors.Is(err, sql.ErrNoRows) {
			return nil, domainerrors.ErrNotFound
		}
		return nil, fmt.Errorf("find <entity> by id: %w", err)
	}
	return &e, nil
}

func (r *<entity>Repo) List(ctx context.Context, filter service.ListFilter) ([]*domain.<Entity>, error) {
	query := `
		SELECT * FROM <entity>s
		WHERE deleted_at IS NULL
		ORDER BY created_at DESC
		LIMIT $1 OFFSET $2`

	var items []*domain.<Entity>
	if err := r.db.SelectContext(ctx, &items, query, filter.Limit, filter.Offset); err != nil {
		return nil, fmt.Errorf("list <entity>s: %w", err)
	}
	return items, nil
}

func (r *<entity>Repo) Update(ctx context.Context, e *domain.<Entity>) (*domain.<Entity>, error) {
	query := `
		UPDATE <entity>s
		SET title = :title, description = :description, price = :price,
		    status = :status, updated_at = :updated_at
		WHERE id = :id AND deleted_at IS NULL
		RETURNING *`

	rows, err := r.db.NamedQueryContext(ctx, query, e)
	if err != nil {
		return nil, fmt.Errorf("update <entity>: %w", err)
	}
	defer rows.Close()

	if rows.Next() {
		var updated domain.<Entity>
		if err := rows.StructScan(&updated); err != nil {
			return nil, err
		}
		return &updated, nil
	}
	return nil, domainerrors.ErrNotFound
}

func (r *<entity>Repo) Delete(ctx context.Context, id string) error {
	// Soft delete — Carousell pattern: never hard delete marketplace items
	result, err := r.db.ExecContext(ctx,
		`UPDATE <entity>s SET deleted_at = NOW() WHERE id = $1 AND deleted_at IS NULL`, id)
	if err != nil {
		return fmt.Errorf("delete <entity>: %w", err)
	}
	rows, _ := result.RowsAffected()
	if rows == 0 {
		return domainerrors.ErrNotFound
	}
	return nil
}

// MustConnect creates a sqlx DB or panics — use only in main()
func MustConnect(dsn string) *sqlx.DB {
	db, err := sqlx.Connect("postgres", dsn)
	if err != nil {
		panic(fmt.Sprintf("failed to connect to postgres: %v", err))
	}
	db.SetMaxOpenConns(25)
	db.SetMaxIdleConns(5)
	return db
}
```

## `migrations/000001_create_<entity>s.sql`

```sql
-- +migrate Up
CREATE TABLE IF NOT EXISTS <entity>s (
    id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id      UUID NOT NULL,
    title        VARCHAR(255) NOT NULL,
    description  TEXT,
    price        NUMERIC(12, 2) NOT NULL DEFAULT 0,
    status       VARCHAR(50) NOT NULL DEFAULT 'active',
    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at   TIMESTAMPTZ  -- soft delete
);

CREATE INDEX idx_<entity>s_user_id    ON <entity>s(user_id);
CREATE INDEX idx_<entity>s_status     ON <entity>s(status);
CREATE INDEX idx_<entity>s_created_at ON <entity>s(created_at DESC);

-- +migrate Down
DROP TABLE IF EXISTS <entity>s;
```
