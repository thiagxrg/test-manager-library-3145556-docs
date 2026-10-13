# Constitution — Manager Library (React + Laravel)

Translation of [`spec-python/constitution.md`](../spec-python/constitution.md) to the real stack of the
challenge. The principles are the same; what changes is **how they are verified** in Laravel/PHP. Where the
stack forces a pragmatic trade-off, it is declared explicitly — never hidden.

> **Note on Article XI (deviation):** the reference spec requires documentation and user-facing messages in
> Spanish, code/columns/routes in English. For this fork, by explicit decision of the candidate, **all
> documentation and all user-facing API messages are written in English** instead. Business rules, data model
> and HTTP contract are unaffected — only the language of prose and error text changes.

---

## Article I — The domain depends on nothing

Pure invariants (`Isbn`, `Quantity`, `DateRange`) live in `app/Domain/ValueObjects/`: plain PHP, no Eloquent, no
Laravel, no third-party libraries. `App\Domain\Entities\{Book,Member,User,Loan,LoanItem}` are also plain PHP
classes with zero framework dependency — they do not extend Eloquent's `Model` and know nothing about
PostgreSQL, HTTP or Sanctum. They are unit-tested with PHPUnit without a database, a server or a container.

## Article II — The core declares its ports; the world implements them

The application layer (`app/Application/Services/*`) only knows the domain. Everything it needs from the
outside (repositories, clock, password hashing, token issuing, cover storage) is expressed as a **port it
defines itself** (`app/Application/Ports/*`, PHP interfaces). No framework or ORM type appears in a port's
signature.

## Article III — A single composition point

A single provider, `app/Providers/AppServiceProvider.php`, is where a port meets its adapter
(`$this->app->bind(BookRepository::class, EloquentBookRepository::class)`). No infrastructure is instantiated
outside of it.

## Article IV — SOLID is not decorative

| Principle | Concrete application |
|---|---|
| **SRP** | One application service = one use case (`LendBooksService` lends, nothing else) |
| **OCP** | Swapping cover storage (disk → S3) only adds a class implementing `CoverStorage` |
| **LSP** | Test fakes replace the real adapter without changing expectations |
| **ISP** | Small ports: `Clock`, `PasswordHasher`, `TokenIssuer`, `CoverStorage`, one repository interface per aggregate |
| **DIP** | Ports are defined by the layer that consumes them (`Application`), implemented by `Infrastructure` |

## Article V — The schema is owned by the migrations

The `infra` repository boots an **empty** engine (`docker-compose.yml`, no SQL init volume). All DDL lives in
Laravel's native migrations (`database/migrations/`), the equivalent of Alembic in the reference spec. See
[ADR-001](adr/adr-001-schema-ownership.md).

## Article VI — Invariants the engine can enforce, the engine enforces

`CHECK`, `UNIQUE` and `FK` are declared in migrations, not only in code: `ck_book_available_range`,
`ck_book_total_positive`, `ck_user_role`, a partial unique index on `isbn` over active books. These are already
in the initial migration — not left pending, unlike the reference spec which marks them as a later task (T-18).

## Article VII — Derived state is computed, never stored

Loan status (`open`, `overdue`, `closed`) and days late are computed in `Loan::status()` and in the overdue
query. There is no column for them.

## Article VIII — Frozen values do not follow the catalog

`loan_item` stores `book_title`, `isbn` and `category_name` as they were at the moment of the loan. The report
and the overdue listing always read from `loan_item`, never from live `book`/`category`.

## Article IX — No secret enters the repository or a log

The real `.env` is in `.gitignore`; only `.env.example` is committed. The password hash (`password_hash`) is in
`User`'s `$hidden` and never logged. Verified by a pattern grep in CI and a contract test confirming no response
ever includes `password` or `password_hash`.

## Article X — The engine wins

If `data-model.md` and the database disagree, the document is wrong and gets corrected.

## Article XI — Language

Code, columns and routes in English (`loan_item`, `due_at`, `/api/v1/loans`). By the deviation noted above,
documentation and the API's user-facing error messages are also in English in this fork.

## Article XII — Nothing is assumed done without evidence

A task in [`tasks.md`](tasks.md) is done when its evidence exists and can be run — not when "it's already coded".
