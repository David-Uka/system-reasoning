# Domain Playbook: Databases

Load when the change touches migrations, transactions, or locking.

- **Migrations** — confirm they're backward-compatible with the
  currently-deployed application version during a rolling deploy (additive
  first, remove/rename in a later migration once old code is gone). A
  migration that drops/renames a column the running app still reads is an
  outage.
- **Locking** — distinguish row-level, table-level, and advisory locks.
  Confirm lock ordering is consistent across all code paths that acquire
  more than one lock, or this introduces a deadlock.
- **Transaction boundaries** — confirm the transaction actually wraps every
  operation that must be atomic, and nothing inside it makes a network call
  that can hang and hold the lock/connection open.
- **Isolation level** — confirm the assumed isolation level (e.g. read
  committed vs. serializable) matches what the database actually uses by
  default; read committed still allows non-repeatable reads and phantom
  reads within the same transaction.
- **Unique constraints as the real guarantee** — if uniqueness matters,
  confirm it's enforced by a DB constraint, not only application-level
  check-then-insert (which races under concurrency).
- **Long-running migrations** — confirm a migration that rewrites a large
  table doesn't hold a blocking lock for the duration in production, and has
  a tested rollback.
