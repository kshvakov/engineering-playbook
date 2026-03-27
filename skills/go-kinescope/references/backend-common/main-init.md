---
description: Main.go initialization pattern with logging, Prometheus metrics, and CLI setup
---

# main.go init pattern

## TL;DR

- Register `application_info` (gauge) in `init()` via `promauto.NewGauge(...)` and `.Add(1)`.
- Logrus:
  - `SetReportCaller(term.IsTerminal(...))` by default (expensive; prefer only for local dev).
  - `TextFormatter` for terminal, `JSONFormatter` for non-terminal.
  - `CallerPrettyfier` shortens paths by trimming `gitlab.kinescope.dev/` and `github.com/kinescope/`.
  - `TimestampFormat`: `2006-01-02 15:04:05`.
- CLI: `urfave/cli`; a global `--debug` flag (and `DEBUG` env) enables debug level and `ReportCaller=true`.
- Use `log.Fatal()` only in `main()` (fatal startup error).

## Notes

- For gRPC services, you may need to import the gzip encoding package (if gzip is used in the project).
