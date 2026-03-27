## Concurrency (goroutines) — safe patterns

### Core rules
- Every goroutine must have an **owner** and a **termination condition**.
- For fan-out/fan-in, use `errgroup` + shared `context.Context`.
- Queues/buffers must be **bounded**, otherwise OOM is a matter of time.

### Practice
- Use `errgroup.WithContext(ctx)` for parallel work.
  On first error, cancel the rest.
- Worker pools:
  - cap concurrency
  - bound the queue (buffer)
  - design backpressure intentionally
- Do not use `time.Sleep` as synchronization.
  Prefer channels, context, explicit barriers.

### Common landmines
- Goroutines not tied to ctx → leaks on cancel/timeouts.
- Shared mutable state without synchronization.
- Loop variable capture in goroutines.
- Blocking ops without timeouts (IO, locks).
