# Tasks — Manager Library (React + Laravel)

Translation of [`spec-python/tasks.md`](../spec-python/tasks.md). A task is done when its **evidence** exists
and can be run (Article XII). Initial status for all: **pending**.

| # | Task | Depends on | Evidence (Laravel/React) |
|---|---|---|---|
| T-01 | Laravel skeleton and `GET /api/v1/health` | — | `curl /api/v1/health` → `{"status":"ok"}`; the project boots with `php artisan serve` or the container |
| T-02 | Initial migration: six tables, FKs, unique indexes, `CHECK`s and the five seeded categories | T-01 | `php artisan migrate`; the queries in `data-model.md` §10 return what is documented; no column has a `DEFAULT` |
| T-03 | Value objects: `Isbn`, `Quantity`, `DateRange` (plain PHP, no Eloquent) | T-01 | PHPUnit: 13-digit ISBN, quantity > 0, end ≥ start |
| T-04 | Domain entity `Book` (withdraw/restock, soft delete) | T-03 | Tests: never < 0, never > total |
| T-05 | Domain entities `Loan`/`LoanItem` (computed status, 14-day due date, max 3) | T-04 | Tests with a fixed clock: `open`, `overdue`, `closed` |
| T-06 | Domain entities `Member` and `User` (username normalization, valid role) | T-03 | `" Ana "` becomes `ana`; the `admin` role is rejected via the API |
| T-07 | **Authentication**: `LoginService`, `CreateLibrarianService`, ports + Sanctum adapter | T-06 | HU-01 and HU-02 pass against fakes; Bearer token valid for 8h |
| T-08 | **Books**: create, search, edit, deactivate, cover, ports + Eloquent repository | T-04 | HU-03 and HU-04 pass |
| T-09 | **Lend** (`LendBooksService`, transaction + `lockForUpdate`) | T-05, T-08 | HU-06 complete: limits, overdue, no copies available |
| T-10 | **Return** and **overdue** | T-09 | HU-07 and HU-09; returning twice → `ITEM_ALREADY_RETURNED` |
| T-11 | **Most-borrowed report** and its read port | T-09 | HU-10; the old title is kept after a rename (ADR-004) |
| T-12 | HTTP adapter: all routes from [`api-contract.md`](api-contract.md), single error body, role-based authorization | T-07, T-08, T-09, T-10, T-11 | A contract test walks every route and every status code from §2 |
| T-13 | Composition and bootstrap: single `AppServiceProvider` binds every port, initial admin from `.env` | T-12 | The system boots with an empty database and the admin can log in |
| T-14 | Database-level invariants: `ck_book_total_positive`, `ck_user_role` | T-02 | An `INSERT` via `psql` that violates them fails |
| T-15 | Architecture tests (Articles I, II and III) | T-13 | A test in `tests/Architecture/` scans `app/Domain` and `app/Application` for forbidden imports (Eloquent, Illuminate\*, Laravel\*) and fails if one is introduced |
| T-16 | Concurrency test: two loans of the last copy | T-09 | Two simultaneous requests: one 201, one 409 (RNF-2) |
| T-17 | `infra`: `docker compose up` boots an empty Postgres, the API and the frontend | T-13 | One command; the database starts with no tables until migrations run |
| T-18 | `tool`: demo seeder **through the API** (≥ 40 books, 10 members, 15 loans, 3 overdue) | T-13 | After running it, `/loans/overdue` returns 3 items and the report has rows |
| T-19 | `app` (React): login, catalog, members, lend, return, overdue, report | T-12 | Manual end-to-end walkthrough |
| T-20 | `page`: static public presentation site | — | Opens without the API; correct on mobile and desktop |

## Definition of done for the test

Whoever reaches T-13 with evidence has the **core** done (equivalent to T-17 of the reference spec: auth +
books + loans + returns + report + bootstrap with admin). T-14 to T-20 are what separates good work from
outstanding work. SDD performance is evaluated, not the number of tasks.
