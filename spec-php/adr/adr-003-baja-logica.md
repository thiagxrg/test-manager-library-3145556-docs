# ADR-003 · Books are deactivated, never deleted

- **Status:** accepted
- **Context:** a loan item (`loan_item`) points to a book. If the book is physically deleted, history and the
  report break.
- **Decision:** the `Book` entity carries `deletedAt`. Deactivating it sets this field; the catalog filters
  by `deletedAt IS NULL` in a single place (a repository-level scope, not Laravel's `SoftDeletes` trait,
  because that trait assumes `created_at`/`updated_at` columns this schema does not have — Article VIII of
  the constitution). A book with open loans **cannot** be deactivated (409 `BOOK_HAS_OPEN_LOANS`). ISBN
  uniqueness is a **partial** unique index over active books (`WHERE deleted_at IS NULL`), so a deactivated
  ISBN can be registered again.
- **Rejected alternatives:** physical delete with cascade (destroys history); Laravel's `SoftDeletes` as-is
  (brings audit columns the data model forbids); an `active = false` flag (doesn't say when).
- **Consequences:** `loan_item.book_id` carries a `RESTRICT` FK as a last line of defense: a manual `DELETE`
  fails loudly instead of corrupting history. The cover binary is the only data that is actually deleted
  physically.
- **How it's verified:** fetching a deactivated book returns 404; a `DELETE` via `psql` on a book with loan
  items fails on the FK.
