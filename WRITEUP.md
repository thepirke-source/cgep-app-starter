# Evidence Chain of Custody — Lab 4.4

The Lab 4.3 pipeline produces evidence on every PR. This lab makes that evidence
*provable*: signed, hashed, timestamped, and stored where it cannot be deleted.
Chain of custody is four properties — each maps to a concrete artifact an auditor
verifies without trusting us.

## The four properties and what proves each

| Property | What it answers | Artifact that proves it | How to verify |
|---|---|---|---|
| **Authenticity** | Who produced this, from where? | Cosign keyless signature (`*.sig.bundle`) — Sigstore Fulcio cert binds the bundle to the GitHub OIDC subject (`repo:.../grc-gate.yml@ref`). | `cosign verify-blob --bundle <b>.sig.bundle --certificate-oidc-issuer https://token.actions.githubusercontent.com <bundle>` |
| **Integrity** | Has it changed since signing? | `*.sha256` recorded at sign time. | Recompute `shasum -a 256 <bundle>`, compare. |
| **Timeliness** | When was it signed? | Sigstore **Rekor** transparency-log entry (timestamp inside the sig bundle). | `cosign verify-blob` confirms the Rekor entry exists. |
| **Preservation** | Can it be deleted before its retention window? | S3 **Object Lock** retention on the vault object (Lab 2.5). | `aws s3api get-object-retention` → `RetainUntilDate` in the future. |

Completeness/preservation is the chain link Object Lock owns: an insider with AWS
admin still cannot delete a locked object before `RetainUntilDate`, and cannot forge
the Rekor entry because it lives outside the AWS account.

## Pipeline flow

```
PR → plan/policy/scan (4.3) → bundle evidence/ → cosign sign-blob (keyless, OIDC)
   → aws s3 cp bundle + .sha256 + .sig.bundle + receipt.json → s3://VAULT/runs/<run_id>/
```

## Verify one run (auditor)

```bash
scripts/verify-evidence.sh <run_id> --vault <bucket> --profile <aws-profile>
# integrity (SHA-256) + authenticity/timestamp (cosign+Rekor) + preservation (retention)
# → "CHAIN INTACT for run <run_id>"
```

## NIST 800-53 mapping

- **AU-9** — protection of audit information (Object Lock + signature).
- **AU-10** — non-repudiation (Fulcio cert chain ties bundle to the signing identity).
- **SI-7** — software/information integrity (SHA-256 + signature verification).
