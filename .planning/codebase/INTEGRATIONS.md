# External Integrations

**Analysis Date:** 2026-01-28

## APIs & External Services

**Package Repositories:**
- NixOS Nixpkgs - Package source
  - URL: `github:nixos/nixpkgs?ref=nixos-unstable`
  - Integration: Direct dependency in `flake.nix`

**GitHub Integration:**
- GitHub Actions - CI/CD automation
  - Workflows: `.github/workflows/docker-build.yml`, `.github/workflows/lint.yml`, `.github/workflows/argc-verify.yml`, `.github/workflows/claude.yml`
  - No authentication required for public repository actions

## Data Storage

**Databases:**
- Not used - This is an environment configuration tool, not a data application

**File Storage:**
- Local filesystem only
  - XDG-compliant directories within repository: `config/`, `cache/`, `share/`, `state/`
  - User-specific subdirectories to prevent multi-user conflicts via `PATH_ID` mechanism

**Caching:**
- XDG_CACHE_HOME at `${SHELFFILES}/cache/${PATH_ID}` - Application-level caching per XDG spec
- GitHub Actions cache - Layer caching for Docker builds (`cache-from: type=gha`, `cache-to: type=gha,mode=max`)
- Nix store caching via flake outputs

## Authentication & Identity

**Auth Provider:**
- None required - Project is infrastructure tooling
- User/group identification via system `id` command for multi-user conflict prevention
  - Implementation: `USER_ID=$(id -u)`, `GROUP_ID=$(id -g)` in `entrypoint/env.sh`

## Monitoring & Observability

**Error Tracking:**
- None detected - Uses standard exit codes and stderr for error reporting

**Logs:**
- Shell command output and BATS test output
- Approach: Standard output redirection and test assertions via BATS framework

## CI/CD & Deployment

**Hosting:**
- GitHub - Repository hosting and CI/CD platform
- Docker Hub - Base images (nixos/nix:2.24.14 for testing)

**CI Pipeline:**
- GitHub Actions workflows:
  - `docker-build.yml` - Docker image build with layer caching (Ubuntu latest)
  - `lint.yml` - Pre-commit hooks (shellcheck v0.10.0.1, hadolint v2.12.1b3, nixfmt)
  - `argc-verify.yml` - Verification that setup.sh matches argc generation
  - `claude.yml` - Claude Code integration trigger

**Build Infrastructure:**
- Docker via nixos/nix:2.24.14 image for reproducible test builds
- nix-portable alternative (from GitHub releases) for no-root installations
- Standard Nix installation fallback via `https://nixos.org/nix/install`

## Environment Configuration

**Required env vars:**
- `SHELFFILES` - Repository root directory path
- No external API keys or secrets required

**Secrets location:**
- `user_env.sh` - Git-ignored file for user-specific secrets or configuration (optional)
- Not detected in standard configuration - User responsibility

## Webhooks & Callbacks

**Incoming:**
- None - This is not a service application

**Outgoing:**
- None detected - No external service callbacks

## External Tool Integration

**Build Tools:**
- argc (sigoden/argc) - CLI framework for argument parsing, used to generate `setup.sh`
- nixfmt - Nix code formatter for pre-commit hooks

**Linting & Quality:**
- shellcheck-py v0.10.0.1 - Shell script linting via pre-commit
- hadolint-py v2.12.1b3 - Dockerfile linting via pre-commit
- pre-commit-hooks v5.0.0 - Standard hooks (trailing whitespace, merge conflict checks, YAML validation)

**Container Runtime:**
- bubblewrap (bwrap) - User-space containerization for isolated execution when Nix unavailable
  - Launched by `entrypoint/launch_in_bwrap.sh`
  - Network sharing enabled (`--share-net`)
  - Mounts repository `/nix` directory into container

---

*Integration audit: 2026-01-28*
