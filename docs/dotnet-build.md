# .NET Build Workflow

Reusable workflow for building, testing, and packaging .NET applications.

## Usage

### Basic Usage

```yaml
name: Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'src/MyApplication.sln'
```

### Full Example with All Options

```yaml
jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      runner: 'cps-cent-runner-nonprod'
      dotnet-version: '8.0.x'
      solution-file: 'src/MyApplication.sln'
      configuration: 'Release'
      run-tests: true
      use-github-packages: true
      timeout: 15
      upload-artifact: true
      artifact-name: 'my-app'
    secrets: inherit
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `runner` | No | `cps-cent-runner-nonprod` | Runner to execute the job |
| `dotnet-version` | No | `8.0.x` | .NET SDK version (GitHub-hosted only) |
| `solution-file` | **Yes** | - | Path to .sln or .csproj file |
| `configuration` | No | `Release` | Build configuration |
| `run-tests` | No | `true` | Execute unit tests |
| `use-github-packages` | No | `false` | Add GitHub Packages NuGet source |
| `timeout` | No | `30` | Job timeout in minutes |
| `upload-artifact` | No | `true` | Upload build output as artifact |
| `artifact-name` | No | `build-output` | Name prefix for artifacts |

## Runner Selection

### Self-Hosted (Recommended)

```yaml
with:
  runner: 'cps-cent-runner-nonprod'  # Non-production
  runner: 'cps-cent-runner-prod'     # Production
```

**Benefits:**
- .NET SDK pre-installed (faster builds)
- No firewall issues with NuGet/Azure
- Consistent environment

### GitHub-Hosted

```yaml
with:
  runner: 'ubuntu-latest'
  dotnet-version: '8.0.x'  # Specify version
```

**Use when:**
- Self-hosted runners unavailable
- Testing across multiple .NET versions
- Public repositories

## GitHub Packages Integration

To consume private NuGet packages from GitHub Packages:

```yaml
jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'src/MyApp.sln'
      use-github-packages: true
    secrets: inherit  # Required for GITHUB_TOKEN
```

The workflow automatically configures:
- NuGet source: `https://nuget.pkg.github.com/CPS-Innovation/index.json`
- Authentication using `GITHUB_TOKEN`

## Artifacts

### Build Output

When `upload-artifact: true`, the workflow uploads:
- All files from `**/bin/{Configuration}/`
- Excludes `**/obj/` directories

Download via GitHub UI or API:
```bash
gh run download <run-id> -n build-output-123
```

### Test Results

When `run-tests: true`, test results are uploaded as:
- Format: TRX (Visual Studio Test Results)
- Artifact name: `test-results-{run_number}`

## Examples

### Console Application

```yaml
jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'ConsoleApp/ConsoleApp.csproj'
      run-tests: false
```

### Web API with Tests

```yaml
jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'WebApi.sln'
      run-tests: true
      timeout: 20
```

### Library with GitHub Packages

```yaml
jobs:
  build:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'MyLibrary/MyLibrary.csproj'
      use-github-packages: true
      configuration: 'Release'
    secrets: inherit
```

### Multi-Environment Build

```yaml
jobs:
  build-debug:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'src/App.sln'
      configuration: 'Debug'
      artifact-name: 'debug-build'

  build-release:
    uses: CPS-Innovation/CPS-Reusable-Workflows/.github/workflows/dotnet-build.yml@v1
    with:
      solution-file: 'src/App.sln'
      configuration: 'Release'
      artifact-name: 'release-build'
```

## Troubleshooting

### "dotnet: command not found"

**Cause:** Self-hosted runner image doesn't have .NET installed.

**Solution:** Ensure runner uses the latest image with .NET SDK, or switch to GitHub-hosted:
```yaml
with:
  runner: 'ubuntu-latest'
```

### NuGet restore timeout

**Cause:** Firewall blocking NuGet feeds.

**Solution:** Use self-hosted runners (they have firewall exceptions):
```yaml
with:
  runner: 'cps-cent-runner-nonprod'
```

### GitHub Packages authentication failed

**Cause:** Missing `secrets: inherit`.

**Solution:**
```yaml
jobs:
  build:
    uses: ...
    with:
      use-github-packages: true
    secrets: inherit  # Add this line
```
