# Technical plan — Manager Library (React + Laravel)

How to build what [`spec.md`](../spec-python/spec.md) and [`constitution.md`](constitution.md) describe (both
identical in business terms to the reference spec). The schema lives in
[`data-model.md`](data-model.md) (also identical: the data model does not depend on the language). The shape of
the system is in [`architecture.md`](architecture.md).

## 1. Technical decisions

| # | Decision | Choice (Laravel) | Reference spec equivalent (Python) | Why |
|---|---|---|---|---|
| D-01 | Language and runtime | PHP 8.2 | Python 3.12 | Challenge stack |
| D-02 | HTTP framework | Laravel 11 | FastAPI | Contract in `api-contract.md` |
| D-03 | Persistence | Eloquent + native migrations over PostgreSQL 16 | SQLAlchemy 2 + Alembic | The schema is owned by migrations (ADR-001) |
| D-04 | Concurrency | `CHECK` range on `book` + row lock (`lockForUpdate`) inside the loan transaction | Version token + `CHECK` | Last copy: one 201, the other 409 (RNF-2, ADR-002) |
| D-05 | Dates | UTC instants (`timestamptz`); injectable clock for testing | `Clock` port | Test due dates without waiting 14 days |
| D-06 | Report | Single aggregated query over `loan_item`, grouped by frozen values | Aggregated read port | Stable and cheap (ADR-004) |
| D-07 | Value objects | `App\Domain\ValueObjects\{Isbn,Quantity,DateRange}`: plain PHP, no Eloquent | Same | No identity; live inside their owner's row |
| D-08 | Covers | Local disk (`storage/app/covers`) in development; the database stores only `cover_key` | Disk/S3 behind `CoverStorage` | Opaque key, never a path |
| D-09 | Passwords | Laravel's native hashing (bcrypt), `password_hash` hidden on the model | Argon2id behind `PasswordHasher` | Article IX |
| D-10 | Bootstrap | Command/seeder creates the initial admin from `ADMIN_USERNAME`/`ADMIN_PASSWORD` | Same | The database cannot produce the hash |
| D-11 | Authentication | **Laravel Sanctum** (personal access tokens), 8h expiration | JWT HS256, key in `JWT_SECRET` | Same contract (`Authorization: Bearer <token>`); Sanctum is Laravel's standard and avoids reinventing signing/verification |
| D-12 | Testing | PHPUnit/Pest: domain unit tests with no DB, feature tests against real Postgres | pytest with fakes + integration | Same test pyramid |

## 2. Environment configuration

| Variable | Required | Example |
|---|---|---|
| `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` | yes | Postgres 16, schema `library` |
| `DB_SCHEMA` | no (default `library,public`) | connection `search_path` |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | yes | set in `.env`, never in the repo |
| `COVERS_DIR` | no | `storage/app/covers` |
| `CORS_ORIGIN` | yes | `http://localhost:5173` |

## 3. Test strategy

| Level | What it tests | Against what |
|---|---|---|
| Domain | `Isbn` (13 digits), `Quantity` (> 0), `DateRange` (end ≥ start) | Nothing: plain PHP |
| Application/service | Use cases: lend, return, report | Real models against a test database |
| Integration | Migrations, constraints, paginated search | Real PostgreSQL |
| Contract | Single error body and status codes from `api-contract.md` | The running API |
| Concurrency | Two loans of the last copy | Real PostgreSQL, two simultaneous requests |

## 4. Build order

Skeleton and `/health` → initial migration with seed → domain value objects/entities → authentication (Sanctum)
→ books → members → loans (critical transaction) → returns and overdue → report → contract and concurrency
tests → React frontend. Details with evidence are in [`tasks.md`](tasks.md).
