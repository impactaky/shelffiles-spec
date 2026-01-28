# Architecture

**Analysis Date:** 2026-01-28

## Pattern Overview

**Overall:** Layered environment management system with XDG-compliant configuration isolation

**Key Characteristics:**
- Multi-layer abstraction separating package management, environment setup, and shell integration
- Declarative package configuration through Nix flakes
- POSIX shell scripting for universal shell compatibility
- Containerized fallback execution via bubblewrap when Nix unavailable
- Path-based user/group isolation to prevent multi-user conflicts

## Layers

**Nix Flakes Layer:**
- Purpose: Declarative package environment definition and reproducible builds across architectures
- Location: `flake.nix`, `packages.nix`
- Contains: Package declarations, build expressions, output definitions
- Depends on: Nixpkgs (github:nixos/nixpkgs)
- Used by: XDG Environment layer for buildEnv generation

**XDG Environment Management Layer:**
- Purpose: Set up XDG base directory variables and isolate configuration per user/group
- Location: `entrypoint/env.sh`
- Contains: XDG variable exports, PATH manipulation, directory initialization
- Depends on: SHELFFILES variable, user/group ID detection
- Used by: All shell entrypoint scripts

**Shell Integration Layer:**
- Purpose: Shell-specific initialization with configuration file discovery
- Location: `entrypoint/bash`, `entrypoint/zsh`, `entrypoint/fish`
- Contains: Shell-specific setup (ZDOTDIR for zsh, HISTFILE for bash)
- Depends on: XDG Environment layer
- Used by: End user shell startup

**Containerization Fallback Layer:**
- Purpose: Provide isolated execution environment when Nix packages unavailable
- Location: `entrypoint/launch_in_bwrap.sh`, `entrypoint/launch_with_nsenter.sh`
- Contains: Bubblewrap mount configuration, namespace setup
- Depends on: bubblewrap binary, /nix directory structure
- Used by: Shell entrypoints when `result/` symlink missing

**Utilities & Testing Layer:**
- Purpose: Docker-based builds and BATS test suite
- Location: `utils/build_nix_in_docker.sh`, `test/Dockerfile`, `test/test/`
- Contains: Docker build orchestration, shell integration tests
- Depends on: Docker, Nix, BATS framework
- Used by: CI/CD pipelines, local development validation

## Data Flow

**Package Environment Setup Flow:**

1. User invokes `./entrypoint/zsh` (or bash/fish)
2. Entrypoint sources `env.sh`
3. `env.sh` exports XDG variables pointing to repository subdirectories
4. `env.sh` prepends `result/bin` and `result_docker/bin` to PATH
5. Shell-specific entrypoint sets shell configuration variables
6. Shell sources configuration from `config/` directories
7. User shell ready with Nix packages available in PATH

**Nix Build Flow:**

1. User runs `nix build` or `./setup.sh`
2. `flake.nix` reads `packages.nix` from repository root (user-customizable)
3. Nix builds `buildEnv` with specified packages for all architectures
4. Build result symlinked to `result/`
5. Entrypoint scripts detect `result/` exists and skip bubblewrap

**Containerized Fallback Flow:**

1. Entrypoint detects missing `result/` and `result_docker/`
2. Invokes `launch_in_bwrap.sh`
3. Bubblewrap creates isolated namespace
4. `/nix` directory from Nix store mounted into container
5. Shell executed within container
6. User operates in isolated environment with Nix packages accessible

**State Management:**

- Configuration: Tracked in `config/` (shared across users)
- Per-user state: Stored in `cache/`, `share/`, `state/` with `${PATH_ID}` suffix (prevents conflicts)
- PATH_ID formula: `"${SHELFFILES}_${USER_ID}_${GROUP_ID}"` converted to safe string
- User environment overrides: Optional `user_env.sh` (git-ignored, not shared)

## Key Abstractions

**SHELFFILES Variable:**
- Purpose: Root path reference for entire system
- Set in: `env.sh`
- Used by: All subsequent path calculations, XDG directory setup
- Allows: Repository to function from any installation path

**XDG Environment Variables:**
- Purpose: Standard-based configuration and state directory redirection
- Exports: XDG_CONFIG_HOME, XDG_CACHE_HOME, XDG_DATA_HOME, XDG_STATE_HOME
- Standard: Follows XDG Base Directory specification
- Applications: Auto-discover configs in `config/` without modification

**PATH_ID (User/Group Isolation Key):**
- Purpose: Prevent cache/state file conflicts in multi-user environments
- Formula: SHA-like encoding of SHELFFILES path + user ID + group ID
- Applied to: `cache/`, `share/`, `state/` subdirectory names
- Example: User alice and bob can both use same installation with isolated caches

**Nix buildEnv:**
- Purpose: Deterministic package aggregation
- Files: `flake.nix` (reads `packages.nix`)
- Output: Symlink tree in `result/`
- Reproducible: Same packages.nix yields identical environment

## Entry Points

**./entrypoint/bash:**
- Location: `entrypoint/bash`
- Triggers: User invokes script directly or via symlink
- Responsibilities: Source env.sh, set HISTFILE, launch bash with config

**./entrypoint/zsh:**
- Location: `entrypoint/zsh`
- Triggers: User invokes script directly
- Responsibilities: Source env.sh, set ZDOTDIR, launch zsh with config, VSCode shell integration workaround

**./entrypoint/fish:**
- Location: `entrypoint/fish`
- Triggers: User invokes script directly
- Responsibilities: Source env.sh, launch fish with config

**./setup.sh:**
- Location: `setup.sh`
- Triggers: `./setup.sh [--no-root] [--sudo-mount] [nix_args...]`
- Responsibilities: Argc-based CLI parsing, invoke `nix build` with flags, optional Docker fallback

**nix build:**
- Entry: Implicit via flake.nix outputs
- Reads: `flake.nix`, `packages.nix`
- Produces: `result/` symlink, `result-2/` for multi-generational builds

## Error Handling

**Strategy:** Fail-fast with explicit error messages

**Patterns:**

- Script exit on error: `set -eu` in bash/sh scripts
- SHELFFILES check: Explicit error if variable unset in `env.sh`
- Directory checks: `[ ! -d "$NIX_HOST_PATH" ]` with stderr output before exit
- Build failure fallback: Entrypoint detects missing `result/` and transparently uses bubblewrap
- Test failures: BATS framework captures stdout/stderr for diagnostic output

## Cross-Cutting Concerns

**Logging:**
- Strategy: stderr for errors, stdout for informational messages
- Pattern: Echo to appropriate stream before exit or success
- Example: `echo "Error: ..." >&2` in env.sh and launch scripts

**Validation:**
- Packages.nix: Nix syntax validation via flake.nix import
- Shell scripts: Shellcheck linting via pre-commit hooks (args: "-x -P entrypoint")
- BATS: Tests verify each shell correctly loads configuration

**Authentication:**
- Strategy: Not directly handled; shell auth delegated to applications
- Configuration: Git auth configured in `example/config/git/config`
- User environment: Can add auth-related vars in `user_env.sh` (git-ignored)

**Containerization Safety:**
- Bubblewrap isolation: Read-only system bindings except /tmp
- /nix mount: Explicitly bound from host Nix store
- Special dirs: proc, dev, sys, run handled per platform

---

*Architecture analysis: 2026-01-28*
