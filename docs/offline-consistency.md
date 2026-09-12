[← Case study](../README.md) · [Run the evidence](verification.md)

# Offline state: a retry must refer to the same operation

**A failed acknowledgement does not mean the server failed to commit.** Treating a retry as new work can duplicate an effect that already happened.

## The Level Up problem

Focus-session state persists in Drift before synchronization. Supabase writes go through an outbox with stable operation IDs. A realtime echo can then bring the same completed session back into the local store.

The completion event has one owner. Reconciliation updates the stored session; it must not run the user-completion action again and publish a second activity contribution. The private regression test completes a session, counts the emitted contribution, mirrors a simulated server echo into Drift, and checks that the contribution was not emitted again.

That is a specific tested invariant, not a claim of exactly-once network delivery.

## A public failure you can reproduce

The [Flutter demo](https://github.com/wassim991/flutter-offline-sync-demo) uses append-only notes to isolate persistence and acknowledgement handling. It does not implement Level Up's focus timer or scoring.

| Step | Local state | Simulated server state |
| :--- | :--- | :--- |
| Save offline | One entry and one pending operation | No commit |
| Go online; lose the next acknowledgement | Operation remains pending | One commit |
| Retry | Original operation ID is reused | Still one commit |
| Receive matching acknowledgement | Entry remains; pending operation clears | One commit |
| Replay matching echoes | No extra entry | One commit |

In the app: **Save locally → go online → Lose next ack → Retry pending → Retry pending → Replay duplicate echoes.** The failure is deliberately injected, so the sequence does not depend on catching a real network outage.

## Decisions in the implementation

**Atomic local persistence.** The [store][store] inserts the entry and outbox payload in one SQLite transaction. If the outbox insert fails, the entry write rolls back. A visible local save must not quietly lose the work needed to synchronize it.

**Identity belongs to the operation.** The [worker][worker] retries the existing ID and payload. The simulated server rejects reuse of that ID with different content. Even after acknowledgement, the local store prevents reusing an entry's ID to overwrite it.

**Acknowledgements must match.** An echo with a different payload cannot clear pending state. Matching duplicates are harmless. A failed retry leaves pending work available for another attempt.

## Tests worth opening

- [Store tests][store-tests]: close and reopen a real disk database; verify the same pending ID survives; force a transaction failure; reject ID reuse and mismatched echoes.
- [Sync tests][sync-tests]: commit successfully, lose the acknowledgement, retry with the same ID, and assert that only one server entry exists.

## Tradeoffs and limits

The demo uses real Drift / SQLite locally and an explicitly simulated, in-memory backend. Echoes are replayed manually. The server loses its map on restart, so a prior “Synced” badge does not establish continued remote durability.

The example handles append-only notes, not multi-device edits, conflict resolution, authentication, background scheduling, or backoff. It demonstrates an idempotency boundary and selected local invariants; it is not a production sync engine.

**The lesson:** persist the identity of the work, distinguish commit from acknowledgement, and keep reconciliation from repeating user actions.

[store]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/lib/data/store.dart
[store-tests]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/test/store_test.dart
[worker]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/lib/sync/worker.dart
[sync-tests]: https://github.com/wassim991/flutter-offline-sync-demo/blob/30b143d39f8db679cb605e176ed87c17a4dd88bd/test/sync_test.dart
