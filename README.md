# Reusable GitHub Actions Workflows

This repository contains reusable workflow templates for CI/CD pipelines across projects.

## Available Workflows

### 🐳 docker-build.yml
Builds and scans Docker images with security analysis.

**Features:**
- Builds Docker image and pushes to GitHub Container Registry (GHCR)
- Generates SBOM (Software Bill of Materials) using Anchore
- Scans images for vulnerabilities with Grype
- Supports custom build arguments
- Layer caching for faster builds

Publishes `ghcr.io/<owner>/<repo>:<sha>`. A nested Dockerfile such as `web/Dockerfile` publishes `<repo>/web` instead.

**Inputs:**
- `build-args` - Docker build arguments (default: `GITHUB_SHA=${{ github.sha }}`)
- `runs-on` - Runner OS (default: `ubuntu-latest`)
- `continue-on-security-error` - Continue on security scan failures (default: `false`)
- `dockerfile` - Dockerfile path (default: `Dockerfile`)
- `context` - Build context (default: `.`)
- `image` - Override the GHCR image name

**Example:**
```yaml
jobs:
  build:
    uses: spire-labs/workflows/.github/workflows/docker-build.yml@main
```

---

### 🦀 rust-check.yml
Comprehensive Rust code quality and security checks.

**Features:**
- Formatting check with `cargo fmt`
- Linting with `cargo clippy`
- Dependency auditing with `cargo audit` and `cargo deny`
- TOML formatting with `taplo`
- Workspace testing

**Inputs:**
- `env-vars` - Environment variables (multiline KEY=VALUE pairs)
- `test-exclude` - Packages to exclude from tests
- `runs-on` - Runner OS (default: `ubuntu-latest`)
- `continue-on-security-error` - Continue on security failures (default: `false`)

**Example:**
```yaml
jobs:
  check:
    uses: spire-labs/workflows/.github/workflows/rust-check.yml@main
    with:
      test-exclude: "heavy-integration-tests"
```

---

### 🔐 sast.yml
Static Application Security Testing using Semgrep.

**Features:**
- Runs Semgrep security analysis
- Generates vulnerability reports
- Uploads results as artifacts
- Summarizes findings in GitHub Step Summary

**Inputs:**
- `runs-on` - Runner OS (default: `ubuntu-latest`)
- `continue-on-security-error` - Continue on findings (default: `false`)

**Example:**
```yaml
jobs:
  security:
    uses: spire-labs/workflows/.github/workflows/sast.yml@main
```

---

### ⚙️ solidity-check.yml
Solidity smart contract validation with Foundry.

**Features:**
- Format checking with `forge fmt`
- Linting with `forge lint`
- Unit and integration testing with `forge test`
- Supports custom contract directories

**Inputs:**
- `contract-directory` - Root folder for contracts (default: `contracts`)
- `run-integration-tests` - Run integration tests (default: `true`)
- `env-vars` - Environment variables (multiline KEY=VALUE pairs)

**Example:**
```yaml
jobs:
  contracts:
    uses: spire-labs/workflows/.github/workflows/solidity-check.yml@main
    with:
      contract-directory: "src/contracts"
```

---

### 🚀 ecr-push.yml
Tags and pushes Docker images to Amazon ECR.

**Features:**
- Pulls images from GHCR
- Tags and pushes to AWS ECR by git SHA
- Optionally also tags sandbox/prod (on by default)
- Uses OIDC for AWS authentication

Copies the matching GHCR image to ECR (`<repo>` or `<repo>/<dir>` from `dockerfile`). Tags `:<sha>` and, by default, `:<environment>`.

**Inputs:**
- `account-id` - AWS account ID (default: Spire account)
- `aws-region` - AWS region (default: `us-east-1`)
- `environment` - GitHub environment for OIDC (default: `prod` on `prod`, otherwise `sandbox`)
- `dockerfile` - Same path as docker-build, used to derive the image name
- `image` - Override GHCR and ECR image name
- `tag-environment` - Also write `:<environment>` (default: `true`)

**Example:**
```yaml
jobs:
  publish:
    uses: spire-labs/workflows/.github/workflows/ecr-push.yml@main
    secrets: inherit
```

---

### 🏷️ tag-release.yml
Creates timestamped release tags with changelogs.

**Features:**
- Generates environment-specific tags
- Creates annotated tags with changelogs
- Maintains both timestamped and latest tags
- Includes compare URLs for changes

**Inputs:**
- `target_env` - Target environment for tagging (required)

**Example:**
```yaml
jobs:
  tag:
    uses: spire-labs/workflows/.github/workflows/tag-release.yml@main
    with:
      target_env: "production"
```

---

### ✅ deployment-check.yml
Verifies deployment by checking commit SHA.

**Features:**
- Polls deployment endpoint for commit verification
- Retries up to 6 times with 5-second intervals
- Supports manual and automated triggers

**Inputs:**
- `url` - URL to check for deployed commit SHA (required)
- `commit-sha` - Expected commit SHA (required)

**Example:**
```yaml
jobs:
  verify:
    uses: spire-labs/workflows/.github/workflows/deployment-check.yml@main
    with:
      url: "https://api.example.com/health"
      commit-sha: ${{ github.sha }}
```

---

## Usage

Reference these workflows in your repository:

```yaml
name: CI Pipeline

on: [push, pull_request]

jobs:
  rust-checks:
    uses: spire-labs/workflows/.github/workflows/rust-check.yml@main

  docker-build:
    uses: spire-labs/workflows/.github/workflows/docker-build.yml@main
    needs: rust-checks
```

## Permissions

Workflows may require specific permissions:
- `contents: read/write` - For repository access and tagging
- `packages: write` - For pushing to GHCR
- `id-token: write` - For AWS OIDC authentication

## Security

All workflows include security scanning:
- **SAST** via Semgrep
- **Dependency auditing** for Rust projects
- **Container scanning** with Grype
- **SBOM generation** for Docker images
