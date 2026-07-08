# Terraform Plan Workflow

Reusable workflow for Terraform plan with security scanning.

## Usage

```yaml
jobs:
  plan:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/terraform-plan.yml@v1
    with:
      environment: 'nonprod'
      working-directory: 'terraform/'
      var-file: 'environments/nonprod.tfvars'
    secrets:
      ARM_CLIENT_ID: ${{ secrets.ARM_CLIENT_ID }}
      ARM_CLIENT_SECRET: ${{ secrets.ARM_CLIENT_SECRET }}
      ARM_SUBSCRIPTION_ID: ${{ secrets.ARM_SUBSCRIPTION_ID }}
      ARM_TENANT_ID: ${{ secrets.ARM_TENANT_ID }}
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `runner` | No | `cps-cent-runner-nonprod` | Runner to execute the job |
| `working-directory` | No | `.` | Terraform working directory |
| `terraform-version` | No | `1.10.0` | Terraform version (GitHub-hosted only) |
| `environment` | **Yes** | - | Environment name |
| `var-file` | No | - | Path to tfvars file |
| `timeout` | No | `30` | Job timeout in minutes |
| `run-tflint` | No | `true` | Run TFLint linting |
| `run-tfsec` | No | `true` | Run tfsec security scan |

## Required Secrets

| Secret | Description |
|--------|-------------|
| `ARM_CLIENT_ID` | Azure Service Principal client ID |
| `ARM_CLIENT_SECRET` | Azure Service Principal secret |
| `ARM_SUBSCRIPTION_ID` | Azure subscription ID |
| `ARM_TENANT_ID` | Azure tenant ID |

## Security Scanning

The workflow includes:
- **TFLint**: Terraform linting and best practices
- **Tfsec**: Security vulnerability scanning (HIGH severity threshold)

Disable scanning if needed:
```yaml
with:
  run-tflint: false
  run-tfsec: false
```
