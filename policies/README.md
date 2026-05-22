# Policy library — cloud targets

Each NIST 800-53 control has a cloud-neutral control ID and one Rego file per cloud
resource type it must match. The control ID is portable; the resource-type match is not.

| Control | GCP file | AWS file | Enforces |
|---|---|---|---|
| SC-28 | `sc28_encryption.rego` | `sc28_encryption_aws.rego` | Encryption at rest. GCP: `google_storage_bucket` has `encryption.default_kms_key_name`. AWS: `aws_s3_bucket` has a matching `aws_s3_bucket_server_side_encryption_configuration`. |
| AC-3 | `ac3_no_public.rego` | `ac3_no_public_aws.rego` | No public access. GCP: `uniform_bucket_level_access=true` + `public_access_prevention="enforced"`; firewalls don't expose 22/3389 to `0.0.0.0/0`. AWS: `aws_s3_bucket_public_access_block` blocks all four public vectors. |
| CM-6 | `cm6_required_tags.rego` | `cm6_required_tags_aws.rego` | Required config baseline. Every taggable resource carries the four labels/tags: `project`, `environment`, `managed_by`, `compliance_scope`. |

Six files, three control IDs. Every deny message includes the resource address AND the
NIST control ID so the developer fixes their own violation without a GRC ticket.

Tests live in `tests/` — one `_test.rego` per control, asserting both pass and fail behavior.
Run the suite: `opa test -v policies/` → `PASS: 8/8`.
