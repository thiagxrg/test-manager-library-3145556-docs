# ADR-001 · The schema is owned by the application's migrations

- **Status:** accepted
- **Context:** the `infra` repository boots a database engine. DDL could end up split between a container
  startup script and the application's migrations; with two owners, nobody knows which schema actually runs
  in production.
- **Decision:** **all DDL lives in Laravel's native migrations** (`database/migrations/`): tables, indexes,
  constraints and the category seed. `infra` only creates the empty engine (`docker-compose.yml`, `db`
  service with no SQL init volume), the user and the volume.
- **Rejected alternatives:** SQL scripts in `infra/db/init/` (a second source of truth); `Schema::create`
  fired at app boot instead of versioned migrations (no history, no rollback).
- **Consequences:** a new index or `CHECK` is created in the **same** migration that uses it. A schema changed
  by hand on the engine is a defect, not a variant.
- **How it's verified:** `infra` contains no `CREATE TABLE`; the queries in `data-model.md` §10 match what
  `php artisan migrate` creates.
