# CPS Reusable Workflows

Centralized repository for GitHub Actions reusable workflows used across CPS-Innovation repositories.

## Available Workflows

| Workflow | Description | Documentation |
|----------|-------------|---------------|
| [dotnet-build.yml](.github/workflows/dotnet-build.yml) | Build, test, and package .NET applications | [docs/dotnet-build.md](docs/dotnet-build.md) |
| [terraform-plan.yml](.github/workflows/terraform-plan.yml) | Terraform plan with security scanning | [docs/terraform-plan.md](docs/terraform-plan.md) |
| [terraform-apply.yml](.github/workflows/terraform-apply.yml) | Terraform apply with approval gates | [docs/terraform-apply.md](docs/terraform-apply.md) |
| [docker-build.yml](.github/workflows/docker-build.yml) | Build and push Docker images to ACR | [docs/docker-build.md](docs/docker-build.md) |

## Quick Start

### 1. Reference a workflow

```yaml
name: Build

on:
  push:
    branches: [main]

jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'src/MyApp.sln'
```

### 2. Use self-hosted runners (recommended)

```yaml
jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      runner: 'cps-cent-runner-nonprod'
      solution-file: 'src/MyApp.sln'
```

### 3. Pass secrets (when required)

```yaml
jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'src/MyApp.sln'
      use-github-packages: true
    secrets: inherit  # Passes GITHUB_TOKEN automatically
```

## Versioning

Always reference workflows by **tag** (not `main`) for stability:

| Reference | Use Case |
|-----------|----------|
| `@v1` | Production - stable, receives patches |
| `@v1.2.0` | Pin to specific version |
| `@main` | Development only - may have breaking changes |

```yaml
# Recommended - major version tag
uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1

# Pin to exact version
uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1.2.0

# NOT recommended for production
uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@main
```

## Runner Selection

| Runner | When to Use |
|--------|-------------|
| `cps-cent-runner-nonprod` | Default for non-prod environments |
| `cps-cent-runner-prod` | Production deployments |
| `ubuntu-latest` | Public repos or when self-hosted unavailable |

Self-hosted runners have pre-installed tools (.NET, Terraform, Azure CLI) - faster builds, no firewall issues.

## Best Practices for Calling Teams

### 1. Always set timeouts
```yaml
with:
  timeout: 15  # Fail fast, don't wait 6 hours
```

### 2. Use environment-specific runners
```yaml
jobs:
  deploy-nonprod:
    uses: ./.github/workflows/deploy.yml
    with:
      runner: 'cps-cent-runner-nonprod'
      environment: 'nonprod'

  deploy-prod:
    needs: deploy-nonprod
    uses: ./.github/workflows/deploy.yml
    with:
      runner: 'cps-cent-runner-prod'
      environment: 'prod'
```

### 3. Pin workflow versions in production
```yaml
# Create a .github/dependabot.yml to get update PRs
version: 2
updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

## Contributing

1. Create a feature branch
2. Update workflow and documentation
3. Test in a non-prod repo first
4. Create PR with example usage
5. After merge, create a release tag

## Support

- Issues: [GitHub Issues](https://github.com/CPS-Innovation/CPS-Reusable-Workflows/issues)
- Teams: #devops-support channel
