<!-- Business-equivalent to spec-python/spec.md: the business spec does not depend on the language. Translated to English per this fork's explicit language decision (see constitution.md Article XI). -->

# Specification — Manager Library

**What is being built and why.** A system for a small library to manage its book catalog, its members and
its loans, and to see which books are borrowed the most.

Governed by [`constitution.md`](constitution.md). The data model lives only in [`data-model.md`](data-model.md)
and the HTTP contract only in [`api-contract.md`](api-contract.md).

---

## 1. Actors

| Actor | What they do | How they authenticate |
|---|---|---|
| **Administrator** (`admin`) | Maintains the book catalog and members; onboards librarians; sees every report | Username and password → token |
| **Librarian** (`librarian`) | Registers loans and returns; browses catalog, members and overdue items | Username and password → token |
| **Member** | A person who borrows books. **Never logs into the system**: they are data, not a user | — |

The `admin` role is provisioned by the deployment from environment variables. **Nobody grants it at runtime.**

## 2. Scope

**In scope:** book catalog with optional cover, five fixed categories, members, loans with multiple items,
per-item returns, overdue books, most-borrowed report, role-based login.

**Out of scope, and not reopened:** fines and payments, reservations, renewals, email or SMS, bulk import,
inter-branch loans, book attributes beyond §3 (no publisher, no year, no language).

## 3. User stories and acceptance criteria

### HU-01 · Log in
As a user I want to authenticate so I can operate according to my role.
- **CA-01.1** With valid credentials, a token is returned, valid for 8 hours.
- **CA-01.2** With invalid credentials, responds 401 with a message that does **not** say whether the username or the password failed.
- **CA-01.3** Every endpoint except login and `/health` requires a token. No token: 401. Insufficient role: 403.

### HU-02 · Onboard librarians (admin)
- **CA-02.1** The admin creates a user with a username, a password and role `librarian`.
- **CA-02.2** The username is stored lowercase and trimmed; it is unique. Repeated: 409.
- **CA-02.3** Requesting the `admin` role through the API responds 422. The password is never returned nor logged.

### HU-03 · Manage books (admin)
- **CA-03.1** Create a book with title, author, ISBN, category and total copies. Available copies start equal to the total.
- **CA-03.2** The ISBN has 13 digits and is unique among active books. Repeated: 409.
- **CA-03.3** Total copies ≥ 1. Changing the total adjusts available copies by the same difference and **never** leaves available copies < 0 (422).
- **CA-03.4** Uploading a cover (JPG/PNG, max 2 MB) stores an **opaque key**; with no cover the field is `null`, never an empty string.
- **CA-03.5** Deleting a book is a **soft delete**: it stops appearing in the catalog but stays in past loans. A book with open loans cannot be deactivated (409).

### HU-04 · Search books
- **CA-04.1** Partial-text search on title or author, filter by category, only active books, ordered by title, paginated (`page`, `size`, max 50).
- **CA-04.2** The response includes the total result count.
- **CA-04.3** Fetching a book by a nonexistent or deactivated id: 404.

### HU-05 · Manage members
- **CA-05.1** The admin and the librarian create members with a name and a document number. The document is unique: repeated gives 409.
- **CA-05.2** Members are listed and searched by name or document, paginated.
- **CA-05.3** Members are never deleted.

### HU-06 · Register a loan
- **CA-06.1** A loan belongs to **one** member, is registered by the authenticated user, and carries 1 to 3 items.
- **CA-06.2** The same book cannot repeat in a loan (422).
- **CA-06.3** Each item withdraws **1** available copy in the same operation. With no copies left: 409 and **nothing is registered**.
- **CA-06.4** A member cannot have more than **3 unreturned books** in total (422 with the current count).
- **CA-06.5** A member with any **overdue** book cannot request another (409).
- **CA-06.6** The due date is `loaned_at + 14 days`. The item freezes the title, ISBN and category name **at that instant**.
- **CA-06.7** A registered loan is never edited or deleted.

### HU-07 · Register a return
- **CA-07.1** **One item** is returned at a time. Sets `returned_at` and adds 1 to available copies.
- **CA-07.2** Returning an already-returned item: 409. **Never** increments twice.
- **CA-07.3** Available copies never exceed the total (the database guarantees it).
- **CA-07.4** When every item is returned, the loan shows as **closed**.

### HU-08 · View a loan and list loans
- **CA-08.1** Fetching a loan returns its items and its computed status (`open`, `overdue`, `closed`).
- **CA-08.2** Listed by date range, status and member, ordered by date descending, paginated.
- **CA-08.3** The status is **computed**, never stored.

### HU-09 · Overdue books
- **CA-09.1** Lists unreturned items whose `due_at` is before now: book, member, days late.
- **CA-09.2** Ordered by most days late first. Only `admin` and `librarian`.

### HU-10 · Most-borrowed report
- **CA-10.1** Given a date range, one row per **book and frozen category name**, with the number of times borrowed, ordered highest to lowest.
- **CA-10.2** End before start: 422. A range with no loans: empty list, not an error.
- **CA-10.3** The report is **not** broken down by librarian or by member.
- **CA-10.4** A report for a closed period **does not change** even if the book or its category is later renamed.
- **CA-10.5** Only `admin`.

## 4. Business rules

| # | Rule |
|---|---|
| RN-1 | Available copies always between 0 and the total. |
| RN-2 | Loan period: 14 days. Maximum 3 unreturned books per member. |
| RN-3 | A member with overdue books cannot borrow more until they are returned. |
| RN-4 | What is frozen on an item (title, ISBN, category) does not follow the catalog. |
| RN-5 | Loans and returns are never deleted. Books only ever get soft-deleted. |
| RN-6 | Dates are UTC instants. User-facing messages and documentation in English in this fork; code and columns in English. |

## 5. Non-functional requirements

| # | Requirement | How it's measured |
|---|---|---|
| RNF-1 | Book search responds in under 300 ms with 5,000 books | Load test with the `tool` seeder |
| RNF-2 | Two librarians lending the **last copy** at once: one wins, the other gets 409 | Concurrent test |
| RNF-3 | The password never travels or is stored in clear text; never appears in logs | Review of logs and responses |
| RNF-4 | Every HTTP error uses the same body (see contract) | Contract test |
| RNF-5 | The system boots with a single command and the database starts empty | `infra` README |
