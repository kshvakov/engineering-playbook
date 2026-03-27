---
description: GitLab CI/CD configuration template using integros/ops shared templates
---

# GitLab CI

## TL;DR

- Use shared templates from the `integros/ops` repository.
- Include: `ai_review.yml`, `build.yml`, `deploy_stage.yml`, `deploy_production.yml`.
- Stages: `review` → `build` → `stage` → `production` (production is usually manual).

## Minimal skeleton

```yaml
---
include:
  - project: 'integros/ops'
    ref: 'master'
    file: 'gitlab-ci/ai_review.yml'
  - project: 'integros/ops'
    ref: 'master'
    file: 'gitlab-ci/build.yml'
  - project: 'integros/ops'
    ref: 'master'
    file: 'gitlab-ci/deploy_stage.yml'
  - project: 'integros/ops'
    ref: 'master'
    file: 'gitlab-ci/deploy_production.yml'

image: registry.kinescope.dev/integros/ops:latest
stages: [review, build, stage, production]
```
