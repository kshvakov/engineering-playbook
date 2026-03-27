---
description: Service deployment specification (spec.yml) for Ansible deployment
---

# spec.yml

## TL;DR

- Use a single `spec.yml` for stage + production; differences go through ansible variables `{{ }}`.
- `MONITORING_HTTP_ADDR` is typically set by the deploy system (don’t set it manually in `environments`).
- Resource variables (`POSTGRES_DSN`, `REDIS_ADDRS`, ...) are injected automatically based on `service.resources`.
- `deploy.probe` is a deployment healthcheck (often `/metrics`), not “metrics configuration”.
