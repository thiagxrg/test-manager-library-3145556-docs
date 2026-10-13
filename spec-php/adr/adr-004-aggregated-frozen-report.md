# ADR-004 · The report is computed by the engine and grouped by frozen values

- **Status:** accepted
- **Context:** the most-borrowed report (HU-10) joins `loan` and `loan_item` over a date range. If it read the
  title or category from the live catalog, renaming a book or a category would change a report for an
  already-closed period.
- **Decision:** (1) the report is **never persisted**: it is computed with a single aggregated query (query
  builder, not Eloquent, to avoid loading unnecessary models) inside `MostBorrowedReportService`. (2) It
  groups by `book_id, book_title, category_name` **exactly as frozen in `loan_item`**. (3) `loan_item.category_name`
  has **no FK**, on purpose. (4) If a book changed category within the range, the report shows **two rows**:
  it does not pick a winner.
- **Rejected alternatives:** joining the live catalog (the report stops being stable); taking "the most recent
  value in the range" (a new loan would change what was already read); materializing a report table (a second
  source of truth).
- **Consequences:** CA-10.1 says "one row per book and frozen category name", not "one row per book". The
  report never includes members or librarians (privacy, `data-model.md` §7).
- **How it's verified:** lend a book, rename it (`PUT /books/{id}`), request the report for the same range:
  the report's title is still the old one.
