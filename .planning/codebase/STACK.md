# Technology Stack

**Analysis Date:** 2026-01-28

## Languages

**Primary:**
- Bash - Shell scripting for setup, entrypoints, and environment management
- Nix - Declarative package management and environment configuration via `flake.nix`

**Secondary:**
- BATS (Bash Automated Testing Framework) - Test framework for shell functionality

## Runtime

**Environment:**
- Nix package manager with flakes support (experimental features enabled)
- Linux/macOS support across x86_64 and aarch64 architectures
- Fallback execution via bubblewrap (bwrap) containerization when Nix unavailable

**Package Manager:**
- Nix with flakes
- Lockfile: `flake.lock` (git-ignored)
- Multi-architecture targets: aarch64-linux, x86_64-linux, aarch64-darwin, x86_64-darwin

## Frameworks

**Core:**
- Nix buildEnv - Creates reproducible package environments
- XDG Base Directory specification - Configuration and state management
- bubblewrap (bwrap) - Containerized execution fallback

**Testing:**
- BATS (Bash Automated Testing Framework) - Shell integration tests
- Test framework configuration: Docker-based execution in `test/Dockerfile`

**Build/Dev:**
- argc - CLI argument parsing and script generation for `setup.sh`
- pre-commit - Git hook framework for code quality checks

## Key Dependencies

**Critical:**
- `nixpkgs` (nixos/nixpkgs?ref=nixos-unstable) - Primary package source
- `nix` version 2.24.14 minimum - Test Docker image uses nixos/nix:2.24.14

**Infrastructure:**
- Docker - Fallback build environment and testing infrastructure
- bubblewrap (bwrap) - Secure containerization without elevated privileges
- Git - Version control with custom filter hooks

## Configuration

**Environment:**
- XDG Base Directory variables (XDG_CONFIG_HOME, XDG_CACHE_HOME, XDG_DATA_HOME, XDG_STATE_HOME)
- User/group-specific path ID generation to prevent multi-user conflicts
- Optional user-specific overrides via `user_env.sh` (git-ignored)

**Build:**
- `flake.nix` - Nix flake configuration with multi-architecture support
- `packages.nix` - User-customizable package list (copied from `example/packages.nix`)
- `.pre-commit-config.yaml` - Pre-commit hook definitions

**Environment Variables:**
- `SHELFFILES` - Repository root path
- `XDG_CONFIG_HOME`, `XDG_CACHE_HOME`, `XDG_DATA_HOME`, `XDG_STATE_HOME` - XDG compliance
- `HISTFILE` - Shell history location (per-shell configured)
- `PATH` - Includes alias directory and Nix build outputs

## Platform Requirements

**Development:**
- Nix package manager with flakes enabled
- Shell interpreter (bash, zsh, or fish) for entrypoints
- bubblewrap (optional, fallback for Nix unavailability)
- Argc CLI tool (for regenerating `setup.sh`)

**Production:**
- Linux/macOS with Nix package manager
- Docker (optional, for builds without root Nix access)
- nix-portable (optional, as no-root alternative to standard Nix installation)

**CI/CD:**
- GitHub Actions with DeterminateSystems/nix-installer-action
- Docker for reproducible test execution
- Pre-commit framework for linting

---

*Stack analysis: 2026-01-28*
