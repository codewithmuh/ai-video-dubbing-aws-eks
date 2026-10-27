# Fukikae (吹き替え): AI Video Dubbing on AWS EKS

[![ci](https://github.com/codewithmuh/ai-video-dubbing-aws-eks/actions/workflows/ci.yml/badge.svg)](https://github.com/codewithmuh/ai-video-dubbing-aws-eks/actions/workflows/ci.yml)
[![release](https://github.com/codewithmuh/ai-video-dubbing-aws-eks/actions/workflows/release.yml/badge.svg)](https://github.com/codewithmuh/ai-video-dubbing-aws-eks/actions/workflows/release.yml)
[![License: learning only, no commercial use](https://img.shields.io/badge/License-learning%20only%2C%20no%20commercial%20use-orange.svg)](LICENSE)

**Hear yourself speak a language you don't know.** Fukikae turns a video of you speaking one language into a
video of you speaking another, in your own voice, with your lips matching the new words. It is a seven-step
pipeline of open-source models and an LLM, run on GPUs in Amazon EKS with Argo Workflows.

> **Only your own voice.** A voice is cloned only after its owner records a consent phrase, and every output is
> labelled AI-dubbed.

**Status: phase 0, building in public.** The repository, CI and the API's plumbing are done; the consent check and choosing the models is next.
See the [roadmap](docs/roadmap.md).

## Part 4 of AI on EKS

Five production AI apps on Amazon EKS, one industry each. Every part adds one new layer of Kubernetes on top of
the parts before it.

| Part | Project | Industry | New on Kubernetes |
|---|---|---|---|
| 1 | [Heya](https://github.com/codewithmuh/ai-virtual-staging-aws-eks): AI virtual staging | Real estate | EKS basics: ALB Ingress with HTTPS, HPA, Pod Identity, then Argo CD |
| 2 | [Karute](https://github.com/codewithmuh/ai-medical-scribe-aws-eks): AI medical scribe | Healthcare | Security: NetworkPolicies, VPC endpoints, External Secrets, WebSockets |
| 3 | [Kisekae](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks): AI virtual try-on | Fashion | GPUs: Karpenter, KEDA on SQS, Spot, scale to zero |
| **4** | **Fukikae: AI dubbing in your own voice (this repo)** | **Media** | **GPU pipelines: Argo Workflows, GPU sharing, RBAC** |
| 5 | [Mimamori](https://github.com/codewithmuh/ai-cctv-video-search-aws-eks): AI CCTV video search | Security & retail | Operators (CRD), batched model serving, GPU metrics, SLOs |

## The demo

1. Record the consent phrase, then upload a 60-second video of yourself talking.
2. Pick a language, for example Japanese.
3. Watch the pipeline run step by step: transcribe, translate, clone the voice, lip-sync.
4. Get the video back in that language, in your voice, with your lips in sync, plus subtitles.

Then the platform demo: kill the GPU node halfway through. Argo Workflows retries the failed step, and the
finished steps are not run again.

## What it proves

| Question an interviewer asks | How Fukikae answers it |
|---|---|
| "How do you run a multi-step ML pipeline on Kubernetes?" | Argo Workflows: one container per step, retries per step, artifacts in S3 |
| "CPU and GPU steps in one pipeline?" | ffmpeg steps on CPU nodes, model steps on GPU nodes; Karpenter picks the instance for each step |
| "How do several models share one GPU?" | NVIDIA time-slicing, measured against one step per GPU |
| "How do you let an API start workflows safely?" | A service account that can create Workflows from one approved template and nothing else |
| "What does a dubbed minute cost?" | GPU minutes and dollars per minute of video, per language, published |

The full design, rules and the numbers it will publish are in [docs/scope.md](docs/scope.md).

## The stack

| Layer | Choice |
|---|---|
| AI | Whisper (speech to text), Claude or OpenAI (translation that fits the original timing), open-source voice-cloning and lip-sync models (chosen in ADRs) |
| App | Next.js web, FastAPI API, one container per pipeline step, ffmpeg |
| Kubernetes | Argo Workflows, Karpenter (CPU and GPU NodePools), NVIDIA time-slicing, RBAC, Argo CD |
| AWS | GPU instances (g5 or g6), S3 for artifacts, ECR, Pod Identity, all in Terraform |

## Before you start

- GPU quota, as in [part 3](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks#before-you-start). Ask early:
  approval can take days.
- An Anthropic or OpenAI API key, and a Hugging Face account for the model weights.
- Phase 0 runs the CPU steps on any machine; the GPU steps need one GPU with 24 GB (a single g5 or g6 instance is
  enough).
- Every phase publishes its measured cost per hour and ends with a teardown that leaves nothing running.

## Quick start

Requirements: [uv](https://docs.astral.sh/uv/), [pre-commit](https://pre-commit.com/) and Docker, or just open
the **Dev Container**.

```bash
make setup    # install dependencies and git hooks
make check    # lint, type-check and test (same as CI)
make up       # the API on :8000

curl localhost:8000/healthz          # {"status":"ok"}
curl localhost:8000/readyz           # {"status":"ready"}

make down
```

Run `make` with no arguments to list every command.

## Repository layout

```
services/api/      FastAPI service: src/api (config, logs, probes, metrics), tests/, Dockerfile. The
                   consent and dubbing endpoints go here as they are built.
docs/              scope, roadmap, architecture, threat model, ADRs, runbooks
compose.yaml       local run with production-like hardening
Makefile           every command you need
```

The web app, Terraform and Kubernetes manifests arrive with their phases in the [roadmap](docs/roadmap.md).

## How changes flow

1. Open a pull request with a [Conventional Commit](https://www.conventionalcommits.org/) title.
2. CI must pass: lint, types, tests, secret and vulnerability scans, image build and smoke test.
3. release-please keeps a Release PR open. Merging it tags the version and publishes a signed image to GHCR with an
   SBOM.

## Production-grade checklist

Every item must pass before v1.0.

- [ ] Code quality: linters, formatters, type checks and pre-commit pass
- [ ] Tests: unit + integration, ≥ 80% coverage, a smoke test and a load test with published numbers
- [ ] CI: every PR runs lint, tests, build and scans; `main` is protected
- [ ] Security: no secrets in git, least privilege, scanned and signed images, SBOM, threat model
- [ ] Infrastructure as code: remote state, `make up` / `make down`, cost estimate, clean teardown
- [ ] Releases: semantic versions, changelog, images in a registry
- [ ] Observability: logs, metrics, traces, dashboards as code, alerts linked to runbooks
- [ ] Reliability: probes, graceful shutdown, timeouts/retries/idempotency, tested backups, written SLOs
- [ ] Docs: README with quick start, architecture, ADRs, runbooks, contributing, license
- [ ] Developer experience: one-command setup, Dev Container or Compose, seed data
- [ ] Demo: screenshots, a short demo video, a write-up

## License

**Learning only. Commercial use is not allowed.** You may clone, run, modify and deploy this project to learn, use
it to teach at a school or in free workshops, and show it in your portfolio with credit. You may not sell it, run it
as a service, use it in company or client work, or use it in paid or monetised courses and videos without written
permission. The full terms are in [LICENSE](LICENSE).

For a commercial license, or if you are unsure whether your use is allowed: **contact@codewithmuh.com**

© 2026 Muhammad Rashid ([CodeWithMuh](https://youtube.com/@codewithmuh)), Senior Platform Engineer ·
[codewithmuh.com](https://codewithmuh.com) · [GitHub](https://github.com/codewithmuh) ·
[LinkedIn](https://www.linkedin.com/in/muhammad-rashid-daha/)
