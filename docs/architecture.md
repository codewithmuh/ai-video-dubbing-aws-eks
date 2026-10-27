# Architecture

> Keep this page current: update it in the same pull request that changes the architecture.
> The target design is in [scope.md](scope.md). This page describes what is **built** today.

## Context

Fukikae dubs a short talking-head video into another language in the speaker's own voice, with lip-sync, after
the speaker records consent. It is part 4 of AI on EKS: a multi-step pipeline of CPU and GPU steps run with
Argo Workflows.

## Built so far

```mermaid
flowchart LR
    client([Client]) --> api[api<br/>FastAPI :8000]
    api -. /metrics .-> prom[(Prometheus)]
```

| Component | Responsibility | Tech | Status |
|---|---|---|---|
| `api` | HTTP API, runtime contract | Python 3.13, FastAPI, uvicorn | Plumbing done; the consent check and choosing the models next |

Planned components are in the [roadmap](roadmap.md).

## Runtime contract (every service)

| Concern | How |
|---|---|
| Liveness | `GET /healthz`: process is up; never checks dependencies |
| Readiness | `GET /readyz`: 503 until started and during shutdown |
| Metrics | `GET /metrics`: Prometheus `http_requests_total`, `http_request_duration_seconds` |
| Logs | One JSON line per event on stdout, with `request_id` |
| Shutdown | On SIGTERM: fail readiness, drain in-flight requests (≤ 20 s), exit 0 |
| Config | Environment variables only; secrets from the platform's secret manager |
| Security | Non-root user, read-only filesystem, no capabilities, minimal base image |

## Data

Planned: uploads, consent recordings, intermediate artifacts and outputs in S3, deleted after 7 days; dub
status in the API's store (chosen in an ADR); workflow history archived by Argo. See [scope.md](scope.md).

## Deployment

Planned: Terraform for AWS; Argo CD syncs the app and the WorkflowTemplate from `main`. Each pipeline step
ships as its own image. Roll back by reverting the commit.

## Reliability

SLOs are written in phase 2 of the [roadmap](roadmap.md), each with an alert and the runbook it links to.

## Security

Trust boundaries and controls: see [threat-model.md](threat-model.md).

## Decisions

See [adr/](adr/).
