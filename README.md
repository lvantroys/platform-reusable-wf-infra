# platform-reusable-wf-infra

Central, platform-owned **reusable GitHub Actions workflows** for Terraform in a regulated AWS environment.

This repo is **high-trust**: changes here can affect how infrastructure is planned/applied across the organization.

## What’s in this repo

Reusable workflows (called via `workflow_call`):

- `.github/workflows/terraform-plan.yml`
- `.github/workflows/terraform-apply.yml`

These workflows:
- assume AWS roles via **GitHub OIDC** (no long-lived AWS keys)
- write an ephemeral `backend.hcl` in the target stack folder
- run `terraform init/validate/plan` or `terraform init/plan/apply`
- upload plan artifacts for review

## Calling pattern (from other repos)

Callers live in:
- `platform-iac-core`
- `platform-iac-env`
- `app-<name>-iac` repos

Example caller job:

```yaml
jobs:
  plan:
    uses: lvantroys/platform-reusable-wf-infra/.github/workflows/terraform-plan.yml@v1
    permissions:
      id-token: write
      contents: read
    with:
      stack_path: live/dev/10-vpc
      environment: dev
      aws_region: us-east-1
      terraform_version: 1.6.6
      role_arn: arn:aws:iam::123456789012:role/gha-platform-iac-env-dev-plan
      tfstate_bucket: acme-demo1-tfstate-us-east-1
      tfstate_kms_key_arn: arn:aws:kms:us-east-1:123456789012:key/xxxx
      state_key: platform-iac-env/dev/10-vpc/terraform.tfstate
