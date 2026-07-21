# Terraform Plan Workflow

Reusable workflow for Terraform plan with Azure OIDC authentication.

## Usage

```yaml
jobs:
  plan:
    uses: CPS-Innovation/CPS-Centralised-Reusable-Workflows/.github/workflows/terraform-plan.yml@v1.2
    with:
      environment: 'nonprod'
      var-file: 'environments/nonprod.tfvars'
      state-resource-group: 'rg-tfstate-nonprod'
      state-storage-account: 'sttfstatenonprod'
      state-key: 'myapp-nonprod.tfstate'
    secrets:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      AZURE_SUBSCRIPTION_ID_STATE: ${{ secrets.AZURE_SUBSCRIPTION_ID_STATE }} # State subscription optional
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `runner` | No | `cps-cent-runner-nonprod` | Runner to execute the job |
| `working-directory` | No | `.` | Terraform working directory |
| `var-file` | **Yes** | - | Path to tfvars file |
| `environment` | **Yes** | - | Environment name (for artifact naming) |
| `state-resource-group` | **Yes** | - | Resource group containing state storage |
| `state-storage-account` | **Yes** | - | Storage account name for tfstate |
| `state-container` | No | `tfstate` | Storage container name |
| `state-key` | **Yes** | - | State file name (e.g., `nonprod.tfstate`) |
| `timeout` | No | `30` | Job timeout in minutes |

## Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `AZURE_CLIENT_ID` | Yes | Azure AD app client ID (for OIDC) |
| `AZURE_TENANT_ID` | Yes | Azure AD tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Yes | Target Azure subscription ID |
| `AZURE_SUBSCRIPTION_ID_STATE` | No | Subscription ID for state storage (defaults to `AZURE_SUBSCRIPTION_ID`) |

## Centralised State Storage

To use a different subscription for Terraform state (avoids chicken-and-egg bootstrap issues):

```yaml
jobs:
  plan:
    uses: CPS-Innovation/CPS-Centralised-Reusable-Workflows/.github/workflows/terraform-plan.yml@v1.2
    with:
      var-file: 'environments/dev.tfvars'
      environment: 'dev'
      state-resource-group: 'rg-tfstate-mgmt'        # In management subscription
      state-storage-account: 'sttfstatemgmt'         # In management subscription
      state-key: 'myapp-dev.tfstate'
    secrets:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
      AZURE_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID_DEV }}        # Target subscription
      AZURE_SUBSCRIPTION_ID_STATE: ${{ secrets.AZURE_SUBSCRIPTION_ID_STATE }} # State subscription
```

## Outputs

| Output | Description |
|--------|-------------|
| `has_changes` | `true` if terraform plan detected changes, `false` otherwise |

Use this output to conditionally run apply:

```yaml
apply:
  needs: plan
  if: needs.plan.outputs.has_changes == 'true'
  uses: .../terraform-apply.yml@v1.2
```

## Prerequisites

1. **Azure OIDC Authentication** - Configure federated credentials in Azure AD:
   - Subject: `repo:your-org/your-repo:ref:refs/heads/main`
   - Issuer: `https://token.actions.githubusercontent.com`

2. **GitHub Secrets** - Add to your repository:
   - `AZURE_CLIENT_ID`
   - `AZURE_TENANT_ID`
   - `AZURE_SUBSCRIPTION_ID`
   - `AZURE_SUBSCRIPTION_ID_STATE` (optional)

3. **RBAC Permissions** - Grant the managed identity:
   - `Contributor` on target resource group/subscription
   - `Storage Blob Data Contributor` on state storage account
