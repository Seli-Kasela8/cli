# GitHub Workflows Documentation

This document describes the CI/CD workflows used in the IPinfo CLI repository.

## Table of Contents

- [Overview](#overview)
- [Workflow Triggers](#workflow-triggers)
- [Deployment Workflows](#deployment-workflows)
- [Required Secrets & Variables](#required-secrets--variables)
- [Release Process](#release-process)
- [Troubleshooting](#troubleshooting)

## Overview

The repository uses GitHub Actions to automate building, testing, and releasing the IPinfo CLI across multiple platforms and package managers. All workflows are configured in the `.github/workflows/` directory.

### Supported Distribution Channels

- **GitHub Releases** - Direct binary downloads (Linux, macOS, Windows)
- **Docker Hub** - Container images
- **Gemfury (APT)** - Debian/Ubuntu package repository
- **Chocolatey** - Windows package manager
- **WinGet** - Windows Package Manager

## Workflow Triggers

Workflows are triggered by git tags pushed to the repository. The tagging convention is:

```
<cli-name>-<version>
```

**Example tags:**
- `ipinfo-1.2.3`
- `ipinfo-2.0.0-rc1`

### Trigger Patterns

| Trigger Type | Used By | Pattern |
|--------------|---------|---------|
| Tag push (all) | GitHub Release, Docker | `*-*` |
| Tag push (specific) | Gemfury/APT | `ipinfo-*` |
| Release event | Chocolatey, WinGet | Release creation in GitHub UI |

## Deployment Workflows

### 1. Release to GitHub

**File:** `.github/workflows/cd_github.yaml`

**Trigger:** Push any tag matching `*-*` (e.g., `ipinfo-1.2.3`)

**What it does:**
1. Extracts CLI name and version from the tag
2. Sets up Go environment (v1.20)
3. Builds binaries for all platforms (Linux, macOS, Windows)
4. Generates changelog from git history
5. Creates GitHub Release with built artifacts and release notes

**Artifacts Published:**
- `.tar.gz` - Linux/macOS archives
- `.zip` - Windows archives
- `.deb` - Debian package
- Installation scripts (`macos.sh`, `windows.ps1`, `deb.sh`)

**Required:**
- Go 1.20 or compatible
- `./scripts/build-archive-all.sh` script
- `./scripts/changelog.sh` script
- GitHub App token with release permissions

---

### 2. Publish Docker Image

**File:** `.github/workflows/cd_docker.yaml`

**Trigger:** Push any tag matching `*-*` (e.g., `ipinfo-1.2.3`)

**What it does:**
1. Extracts CLI name and version from the tag
2. Sets up Go environment
3. Authenticates to Docker Hub
4. Builds and pushes Docker images with version tag
5. Also pushes image tagged as `latest`

**Images Published:**
- `<registry>/<name>:<version>` (e.g., `ipinfo/ipinfo:1.2.3`)
- `<registry>/<name>:latest`

**Required:**
- Docker Hub credentials (`DOCKER_USERNAME`, `DOCKER_PASSWORD`)
- `./scripts/docker.sh` build script

---

### 3. Release to Gemfury (APT Repository)

**File:** `.github/workflows/cd_gemfury.yaml`

**Trigger:** Push tag matching `ipinfo-*` (e.g., `ipinfo-1.2.3`)

**What it does:**
1. Extracts CLI name and version from the tag
2. Validates tag format
3. Builds all platform binaries
4. Installs Gemfury CLI tool
5. Uploads Linux .deb packages to Gemfury

**Packages Published:**
- `.deb` files for various Linux architectures

**Required:**
- Gemfury API token (`IPINFO_GEMFURY_PUSH_TOKEN`)
- `./scripts/build-archive-all.sh` script

**Installation from APT:**
```bash
echo "deb [trusted=yes] https://apt.fury.io/cli/ * *" | sudo tee /etc/apt/sources.list.d/fury-cli.list
sudo apt-get update
sudo apt-get install ipinfo
```

---

### 4. Publish to Chocolatey

**File:** `.github/workflows/cd_chocolatey.yaml`

**Trigger:** Release event created in GitHub UI (e.g., Draft Release marked as Released)

**What it does:**
1. Validates release name matches `ipinfo-x.x.x` pattern
2. Only proceeds if validation passes
3. Installs Chocolatey AU (Automatic Updater) module
4. Updates package metadata and version
5. Tests the package installation
6. Pushes package to Chocolatey repository

**Requirements:**
- GitHub Release name must follow pattern: `ipinfo-x.x.x`
- Chocolatey API key (`CHOCO_API_KEY`)
- `./chocolatey-packages/ipinfo/update.ps1` script
- Runs on Windows (`windows-latest`)

**Installation from Chocolatey:**
```bash
choco install ipinfo
```

---

### 5. Publish to WinGet

**File:** `.github/workflows/cd_winget.yaml`

**Trigger:** Release event created in GitHub UI

**What it does:**
1. Validates release name matches `ipinfo-x.x.x` pattern
2. Only proceeds if validation passes
3. Generates GitHub App token with repository permissions
4. Uses vedantmgoyal2009/winget-releaser action
5. Detects Windows installer files and submits to WinGet registry

**Requirements:**
- GitHub Release name must follow pattern: `ipinfo-x.x.x`
- GitHub App credentials (`G_APP_ID`, `G_APP_PRIVATE_KEY`)
- Windows installer files (`_windows_*.zip`) in GitHub Release
- Runs on Windows (`windows-latest`)

**Installation from WinGet:**
```bash
winget install ipinfo.ipinfo
```

---

## Required Secrets & Variables

### GitHub Secrets

These must be configured in **Settings > Secrets and variables > Actions**:

| Secret Name | Used By | Purpose |
|-------------|---------|---------|
| `DOCKER_USERNAME` | cd_docker.yaml | Docker Hub authentication |
| `DOCKER_PASSWORD` | cd_docker.yaml | Docker Hub authentication |
| `CHOCO_API_KEY` | cd_chocolatey.yaml | Chocolatey repository access |
| `IPINFO_GEMFURY_PUSH_TOKEN` | cd_gemfury.yaml | Gemfury repository access |
| `G_APP_PRIVATE_KEY` | cd_github.yaml, cd_winget.yaml | GitHub App authentication |

### GitHub Variables

These must be configured in **Settings > Secrets and variables > Actions > Variables**:

| Variable Name | Used By | Purpose |
|---------------|---------|---------|
| `G_APP_ID` | cd_github.yaml, cd_winget.yaml | GitHub App ID |

### How to Set Up

1. Go to **Settings > Secrets and variables > Actions**
2. Click **New repository secret** (for secrets) or **New repository variable** (for variables)
3. Enter the name and value
4. Click **Add secret/variable**

---

## Release Process

### Step 1: Prepare Release

1. Update version in code (if needed)
2. Update CHANGELOG with release notes
3. Commit and push changes to main branch
4. Create a git tag:

```bash
git tag -a ipinfo-1.2.3 -m "Release version 1.2.3"
git push origin ipinfo-1.2.3
```

### Step 2: Automatic Deployments (Tag-based)

When you push a tag matching `ipinfo-*`:
- ✅ GitHub Release workflow triggers
- ✅ Docker workflow triggers
- ✅ Gemfury/APT workflow triggers

Artifacts are automatically built and published to:
- GitHub Releases (with changelog)
- Docker Hub
- Gemfury APT repository

### Step 3: Manual Deployments (Release-based)

For Chocolatey and WinGet, use GitHub's web UI:

1. Go to **Releases** section
2. Click **Create a new release**
3. Select the tag you created (e.g., `ipinfo-1.2.3`)
4. Set **Release title** to: `ipinfo-1.2.3` (must match pattern `ipinfo-x.x.x`)
5. Fill in release description
6. Ensure **Set as latest release** is checked
7. Click **Publish release**

This triggers:
- ✅ Chocolatey workflow
- ✅ WinGet workflow

---

## Workflow Execution Details

### Build Scripts

All workflows rely on shell scripts in `./scripts/`:

| Script | Purpose | Parameters |
|--------|---------|------------|
| `build-archive-all.sh` | Build binaries for all platforms | `<cli-name> <version> [push-to-registry]` |
| `changelog.sh` | Generate changelog from git history | `<cli-name> <version>` |
| `docker.sh` | Build and push Docker images | `<cli-name> <version> [-r]` |

### Platform Support

#### Supported OS/Architectures

- **Linux**: x86_64, arm64, 386, armv7
- **macOS**: x86_64, arm64
- **Windows**: x86_64, 386

#### Go Version

- **Current**: Go 1.20
- **Recommendation**: Update regularly for security patches

---

## Troubleshooting

### Workflow Fails at Build Step

**Symptoms:** Build fails in GitHub, Docker, or Gemfury workflow

**Check:**
1. Verify Go version is installed: `go version`
2. Check build script exists: `./scripts/build-archive-all.sh`
3. Review build logs for compiler errors
4. Ensure all dependencies are vendored

**Action:**
- Fix code compilation errors locally first
- Test build locally before pushing tag

### Tag Not Recognized

**Symptoms:** Workflow doesn't trigger when tag is pushed

**Check:**
1. Tag format: Must be `ipinfo-x.x.x` (semantic versioning)
2. Tag was pushed to main branch: `git push origin <tag-name>`
3. Workflow trigger pattern matches tag name

**Correct format:**
```bash
git tag ipinfo-1.2.3
git push origin ipinfo-1.2.3
```

### Release Event Not Triggering Chocolatey/WinGet

**Symptoms:** Push tag works, but Chocolatey/WinGet workflows don't run

**Check:**
1. Release name must match pattern: `ipinfo-x.x.x`
2. Release must be published (not draft)
3. Release was created from the correct tag

**Action:**
- Create release via GitHub UI
- Verify release title matches `ipinfo-x.x.x` exactly
- Click "Publish release" (not draft)

### Docker Authentication Failed

**Symptoms:** Error message: "Unauthorized: authentication required"

**Check:**
1. `DOCKER_USERNAME` secret is set
2. `DOCKER_PASSWORD` secret is set (not access token)
3. Docker Hub credentials are valid
4. Account has permission to push to repository

**Action:**
- Regenerate Docker Hub credentials
- Update secrets in GitHub Settings

### Missing Artifacts in Release

**Symptoms:** GitHub Release created but missing .deb, .zip, .tar.gz files

**Check:**
1. Build script completed successfully
2. Artifacts exist in `./build/` directory
3. File paths in workflow match actual locations

**Action:**
- Review workflow logs
- Check if build step had errors
- Verify artifact paths in workflow

### WinGet/Chocolatey Publish Fails

**Symptoms:** Release event triggers but package not published

**Check:**
1. GitHub App token is valid (check `G_APP_ID`, `G_APP_PRIVATE_KEY`)
2. Chocolatey API key is valid (`CHOCO_API_KEY`)
3. Gemfury token is valid (`IPINFO_GEMFURY_PUSH_TOKEN`)

**Action:**
- Regenerate and update secrets
- Check package name matches registry expectations
- Review workflow logs for specific error messages

---

## Best Practices

### For Release Management

1. ✅ Always test locally before pushing tags
2. ✅ Use semantic versioning: `ipinfo-MAJOR.MINOR.PATCH`
3. ✅ Include changelog in release notes
4. ✅ Review generated artifacts before publishing
5. ✅ Test released binaries on each platform

### For Security

1. ✅ Rotate API keys regularly
2. ✅ Use GitHub App tokens instead of personal access tokens
3. ✅ Restrict workflow permissions to minimum required
4. ✅ Never commit secrets to repository
5. ✅ Audit workflow logs for unauthorized access

### For Maintenance

1. ✅ Keep Go version updated (check quarterly)
2. ✅ Monitor build times and optimize as needed
3. ✅ Regularly test each distribution channel
4. ✅ Document any custom build scripts
5. ✅ Review and update this documentation as workflows change

---

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Semantic Versioning](https://semver.org/)
- [Go Build System](https://golang.org/doc/cmd)
- [Docker Hub](https://hub.docker.com/)
- [Chocolatey](https://chocolatey.org/)
- [WinGet](https://learn.microsoft.com/en-us/windows/package-manager/)
- [Gemfury](https://gemfury.com/)

---

**Last Updated:** 2026-05-01

For questions or issues, please open an issue in the repository or contact the maintainers.
