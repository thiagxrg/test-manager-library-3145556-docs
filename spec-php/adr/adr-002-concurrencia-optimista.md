# ADR-002 · Concurrency over copies: row locking + CHECK as the last line of defense

- **Status:** accepted
- **Context:** two librarians can lend the **last copy** of the same book at the same time (RNF-2). If both
  read `available_copies = 1` and both write, the system lends a book it doesn't have.
- **Decision:** inside `LendBooksService`'s transaction, the book row is locked with `lockForUpdate()`
  (Eloquent) before withdrawing the copy; the second request waits for the lock and re-reads the already
  updated `available_copies`, so it fails with **409 `BOOK_NOT_AVAILABLE`** if no copies are left. The
  **last line of defense**, same as the reference spec, is the `CHECK` `ck_book_available_range`
  (`0 <= available_copies <= total_copies`): if anything writes outside this service, the database rejects
  the data before it can corrupt state.
- **Rejected alternatives:** optimistic concurrency with a version token (as in the Python spec): closer to
  the original, but `lockForUpdate` is the idiomatic Eloquent pattern for this case and gives the same
  observable guarantee (one request 201, the other 409) with less new code.
- **Consequences:** the library is not serialized globally: the lock is per book row, not per table. Returning
  the same item twice does not increment twice because `returnedAt` is already set (CA-07.2).
- **How it's verified:** a test with two simultaneous HTTP requests against a book with a single copy
  available: exactly one gets 201, the other gets 409.
