# Threat model

Lightweight [STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) review.
Revisit it when you add an external integration, a data store, or a new trust boundary.

## Assets

- Uploaded videos and voice recordings (biometric data)
- Consent recordings, the proof that a voice may be cloned
- Generated videos
- GPU capacity and the AI bill
- Build and release pipeline (images, signing identity)

## Threats and mitigations

| STRIDE | Threat | Mitigation | Status |
|---|---|---|---|
| Spoofing | Forged release images | Keyless cosign signatures tied to this repo's workflow | ✅ |
| Tampering | Compromised GitHub Action or dependency | Actions pinned to SHAs, Dependabot, Trivy scans, SBOM | ✅ |
| Repudiation | Unknown who changed what | Protected `main`, required reviews, signed release provenance | ✅ |
| Information disclosure | Secrets committed to git | gitleaks in pre-commit and CI; `.env` ignored | ✅ |
| Elevation of privilege | Container breakout | Non-root, read-only, no capabilities, minimal base image | ✅ |
| Denial of service | Request floods exhaust the service | Rate limits at the edge (AWS WAF on the ALB) and per user in the API | ⏳ planned |
| Denial of service | Abuse runs up the AI or GPU bill ("denial of wallet") | Per-user daily limits, a global daily budget, alerts at 80% | ⏳ planned |
| Tampering | Prompt injection through uploaded content | Model output is schema-validated and never triggers an action on its own | ⏳ planned |
| Spoofing | Someone clones another person's voice | Rule R1: consent phrase with a one-time code, transcript match and speaker match against the video | ⏳ planned |
| Repudiation | A dubbed video is passed off as real | Visible "AI-dubbed" label and file metadata; the consent recording is kept with the output | ⏳ planned |
| Elevation of privilege | The API's service account runs arbitrary pods | RBAC: it can only create Workflows from the approved WorkflowTemplate | ⏳ planned |
| Tampering | A malicious model file runs code in a step | Weights in safetensors format from pinned revisions, mirrored to a private S3 bucket | ⏳ planned |

## Open risks

List accepted risks with an owner and a review date.
