## Testing — determinism and contracts

### Rules
- Tests must be **deterministic** (no `sleep` as synchronization).
- Cover negative paths and boundaries (timeouts, ctx cancel, dependency failures).
- Use the race detector for concurrent code when possible.

### Practice
- Table-driven tests for scenario matrices.
- Inject time/randomness via interfaces (clock, rand) instead of globals.
- For integration tests:
  - set explicit timeouts
  - avoid flaky waits
  - validate the contract (status codes/messages/fields)
