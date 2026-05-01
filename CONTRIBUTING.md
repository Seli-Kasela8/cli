# Contributing to IPinfo CLI

Thank you for your interest in contributing to the IPinfo CLI! This document provides guidelines and instructions for contributing to the project.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Setup](#development-setup)
- [Making Changes](#making-changes)
- [Testing](#testing)
- [Submitting Changes](#submitting-changes)
- [Pull Request Process](#pull-request-process)
- [Release Process](#release-process)
- [Code Style Guide](#code-style-guide)
- [Documentation](#documentation)

## Code of Conduct

We are committed to providing a welcoming and inspiring community for all. Please read and follow our Code of Conduct:

- Be respectful and inclusive
- Welcome different viewpoints and experiences
- Focus on constructive criticism
- Respect privacy and confidentiality
- Report violations to the maintainers

## Getting Started

### Prerequisites

- **Go**: 1.20 or later ([Download](https://golang.org/dl/))
- **Git**: For version control
- **Make**: For running build commands (optional)
- **Docker**: For testing Docker builds (optional)

### Fork and Clone

1. Fork the repository on GitHub
2. Clone your fork:
   ```bash
   git clone https://github.com/YOUR-USERNAME/cli.git
   cd cli
   ```
3. Add upstream remote:
   ```bash
   git remote add upstream https://github.com/Seli-Kasela8/cli.git
   ```

## Development Setup

### 1. Install Dependencies

```bash
# Download dependencies
go mod download

# Verify dependencies
go mod verify
```

### 2. Build the Project

```bash
# Build for your current platform
go build -o ipinfo ./cmd/ipinfo

# Or use the build script for all platforms
./scripts/build-archive-all.sh ipinfo 0.0.0-dev
```

### 3. Run the CLI

```bash
# Using the built binary
./ipinfo --help

# Or use go run directly
go run ./cmd/ipinfo --help
```

### 4. Set Up Your Environment

For API-dependent features, you may need to set environment variables:

```bash
# IPinfo API token (optional for basic queries)
export IPINFO_TOKEN=your_token_here

# Proxy settings (if needed)
export HTTP_PROXY=http://proxy.example.com:8080
```

## Making Changes

### 1. Create a Branch

Create a feature branch from `master`:

```bash
git fetch upstream
git checkout -b feature/my-feature upstream/master
```

**Branch naming conventions:**
- `feature/description` - New features
- `bugfix/description` - Bug fixes
- `docs/description` - Documentation updates
- `chore/description` - Maintenance, deps, tooling
- `test/description` - Tests and test infrastructure

### 2. Make Your Changes

- Write clear, descriptive commit messages
- Keep commits focused and atomic
- Reference related issues in commit messages

```bash
# Example commit message
git commit -m "feat: add support for IPv6 geolocation lookup

- Add IPv6 validation in address parser
- Update geolocation endpoint to handle IPv6
- Fixes #123"
```

### 3. Keep Your Branch Updated

```bash
git fetch upstream
git rebase upstream/master
```

## Testing

### Running Tests

```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run tests with detailed output
go test -v ./...

# Run specific test
go test -run TestName ./...
```

### Running Linters

```bash
# Install golangci-lint (if not already installed)
# See https://golangci-lint.run/usage/install/

# Run linter
golangci-lint run

# Fix common issues
golangci-lint run --fix
```

### Manual Testing

Test your changes manually before submitting:

```bash
# Build and test the binary
go build -o ipinfo ./cmd/ipinfo

# Test basic functionality
./ipinfo 8.8.8.8
./ipinfo google.com
./ipinfo --help

# Test with flags
./ipinfo -f json 8.8.8.8
./ipinfo -f csv 8.8.8.8
./ipinfo -f yaml 8.8.8.8
```

### Testing Different Platforms

If possible, test on multiple platforms:

```bash
# macOS
GOOS=darwin GOARCH=amd64 go build -o ipinfo-darwin

# Linux
GOOS=linux GOARCH=amd64 go build -o ipinfo-linux

# Windows
GOOS=windows GOARCH=amd64 go build -o ipinfo.exe
```

## Submitting Changes

### Before You Submit

1. ✅ All tests pass: `go test ./...`
2. ✅ Code is properly formatted: `go fmt ./...`
3. ✅ No linting issues: `golangci-lint run`
4. ✅ Changes are documented
5. ✅ Commit messages are clear
6. ✅ Your branch is up-to-date with upstream

### Push Your Changes

```bash
git push origin feature/my-feature
```

## Pull Request Process

### Creating a Pull Request

1. Go to your fork on GitHub
2. Click "Compare & pull request" next to your branch
3. Fill in the PR template with:
   - **Title**: Clear, descriptive title
   - **Description**: What changes, why, and how
   - **Related Issues**: Link any related issues
   - **Testing**: Describe testing performed
   - **Checklist**: Mark completed items

### PR Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Related Issues
Fixes #(issue number)

## How Has This Been Tested?
Describe testing performed

## Screenshots/Output (if applicable)
Include any relevant screenshots or CLI output

## Checklist
- [ ] My code follows the style guidelines
- [ ] I have performed a self-review of my code
- [ ] I have commented my code where needed
- [ ] I have updated documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix/feature works
- [ ] New and existing tests pass locally
```

### PR Review Process

1. **Automated Checks**: GitHub Actions runs tests and linters
2. **Code Review**: Maintainers review your code
3. **Address Feedback**: Update your PR based on feedback
4. **Approval**: After approval, your PR will be merged

### Addressing Review Comments

```bash
# Make changes based on feedback
# Commit with descriptive message
git commit -m "refactor: address review feedback on PR #XX"

# Push updated branch
git push origin feature/my-feature
# No need to force push - new commits are added to the PR
```

## Release Process

For maintainers releasing new versions, see [`.github/WORKFLOWS.md`](.github/WORKFLOWS.md).

### Version Numbering

We follow [Semantic Versioning](https://semver.org/):

- **MAJOR**: Breaking API changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

**Example:** `ipinfo-1.2.3`

### Release Steps (for maintainers)

1. Update version in code
2. Update CHANGELOG with release notes
3. Commit: `git commit -m "chore: release ipinfo-1.2.3"`
4. Create annotated tag: `git tag -a ipinfo-1.2.3 -m "Release version 1.2.3"`
5. Push tag: `git push origin ipinfo-1.2.3`
6. Create GitHub Release via web UI
7. Workflows automatically build and publish to all platforms

## Code Style Guide

### Go Code Style

We follow [Effective Go](https://golang.org/doc/effective_go) guidelines:

```go
// Use clear variable names
var userIP string
var timeout time.Duration

// Keep functions small and focused
// Maximum line length: 100 characters
// Indent with tabs

// Use meaningful comments
// Comments should explain "why", not "what"
// Good:
// Use exponential backoff to avoid overwhelming the API
// Bad:
// Retry the request
```

### Formatting

```bash
# Format code automatically
go fmt ./...

# Check formatting without modifying
gofmt -l ./...
```

### Naming Conventions

- **Packages**: lowercase, single word if possible (`ipinfo`, `geolocation`)
- **Types**: PascalCase (`IPData`, `LocationInfo`)
- **Variables**: camelCase (`userInput`, `responseData`)
- **Constants**: PascalCase or SCREAMING_SNAKE_CASE (`DefaultTimeout`, `MAX_RETRIES`)
- **Functions**: PascalCase if exported, camelCase if unexported

### Error Handling

```go
// Always handle errors
if err != nil {
    return fmt.Errorf("failed to fetch data: %w", err)
}

// Don't ignore errors silently
// Bad:
_ = os.Remove(file)

// Good:
if err := os.Remove(file); err != nil {
    log.Printf("warning: failed to clean up: %v", err)
}
```

### Comments

```go
// Package ipinfo provides functionality for IP geolocation
package ipinfo

// LookupIP retrieves geolocation data for an IP address.
// It returns an IPData struct or an error if the lookup fails.
func LookupIP(ip string) (*IPData, error) {
    // implementation
}
```

## Documentation

### Updating Documentation

- **README.md**: Usage and quick start
- **CONTRIBUTING.md**: Contributing guidelines (this file)
- **WORKFLOWS.md**: CI/CD workflow documentation
- **Inline Comments**: Explain complex logic
- **Examples**: Include usage examples in README

### Documentation Checklist

When adding a new feature:

- [ ] Update README.md with new usage examples
- [ ] Add function/type comments in code
- [ ] Update CHANGELOG.md
- [ ] Add tests with doc examples
- [ ] Update any relevant architecture docs

### Writing Documentation

- Use clear, concise language
- Provide examples
- Explain the "why", not just the "what"
- Keep code examples up-to-date
- Use proper formatting and grammar

## Getting Help

### Questions or Need Clarification?

- Open a [Discussion](https://github.com/Seli-Kasela8/cli/discussions)
- Check existing [Issues](https://github.com/Seli-Kasela8/cli/issues)
- Review [README.md](README.md) and [WORKFLOWS.md](.github/WORKFLOWS.md)

### Reporting Bugs

Please use our [Bug Report](https://github.com/Seli-Kasela8/cli/issues/new?template=bug_report.md) template and include:

- Go version: `go version`
- OS and architecture: `uname -a`
- CLI version: `ipinfo --version`
- Steps to reproduce
- Expected vs actual behavior
- Error messages and stack traces

## Useful Resources

- [Go Documentation](https://golang.org/doc/)
- [Effective Go](https://golang.org/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [IPinfo API Documentation](https://ipinfo.io/docs)
- [GitHub Flow Guide](https://guides.github.com/introduction/flow/)

## Recognition

Contributors are recognized in:
- GitHub Contributors page
- Release notes for related features
- README.md (for significant contributions)

Thank you for contributing! 🎉

---

**Last Updated:** 2026-05-01
