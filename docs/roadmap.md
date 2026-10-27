# Roadmap

Built in public, phase by phase. Every phase ends with measured results, not claims. See [scope.md](scope.md).

## Phase 0: the pipeline, locally (one GPU)

- [x] Repository from the production template: CI, scans, signed releases, FastAPI plumbing (probes, metrics,
      JSON logs, graceful shutdown)
- [ ] ADRs: voice-cloning model, lip-sync model (quality, languages, GPU memory, license), translation that fits the
      timing, Argo Workflows for the pipeline
- [ ] Consent: phrase with a one-time code; transcript match and speaker match must pass (rule R1)
- [ ] Each step as its own container: extract, transcribe, translate, clone voice, fit timing, lip-sync, join
- [ ] The whole pipeline runs end to end on one GPU
- [ ] Web: consent, upload, language picker, progress per step, result

## Phase 1: pipelines on EKS

- [ ] Terraform: VPC, EKS, Karpenter (CPU and GPU NodePools), S3, ECR, Pod Identity
- [ ] Argo Workflows: a WorkflowTemplate for the pipeline, artifacts in S3, retries per step
- [ ] The API starts workflows with a service account limited to that template
- [ ] NVIDIA time-slicing so steps share one GPU; compared with one step per GPU
- [ ] Argo CD for everything

## Phase 2: measured

- [ ] GPU minutes and cost per minute of dubbed video, per language
- [ ] Drill: kill a GPU node mid-pipeline; the workflow resumes from the failed step
- [ ] Quality: a back-translation score and a lip-sync score for each output

## Phase 3: v1.0

- [ ] All 11 areas of the production-grade checklist
- [ ] Results table in the README, demo video and write-up
