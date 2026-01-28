# Codebase Structure

**Analysis Date:** 2026-01-28

## Directory Layout

```
shelffiles/
├── flake.nix                 # Nix flake definition for multi-arch builds
├── setup.sh                  # Argc-based setup CLI tool
├── .pre-commit-config.yaml   # Pre-commit hooks for shellcheck, hadolint, nixfmt
├── README.md                 # Project documentation
│
├── entrypoint/               # Shell-specific entry points
│   ├── bash                  # Bash launcher (sets HISTFILE, sources .bashrc)
│   ├── zsh                   # Zsh launcher (sets ZDOTDIR, sources .zshrc)
│   ├── fish                  # Fish launcher (sources config.fish)
│   ├── env.sh                # XDG environment setup (shared by all shells)
│   ├── launch_in_bwrap.sh    # Bubblewrap containerized execution fallback
│   └── launch_with_nsenter.sh # Nsenter/mount-based execution (optional)
│
├── config/                   # XDG configuration directory (user-customizable)
│   ├── nix/                  # Nix-related configuration
│   ├── bash/                 # Bash shell config (created by user)
│   ├── zsh/                  # Zsh shell config (created by user)
│   ├── git/                  # Git configuration (created by user)
│   └── [app-name]/           # Application-specific configs (user-created)
│
├── cache/                    # XDG_CACHE_HOME per user (git-ignored)
│   └── {PATH_ID}/            # User/group-specific cache isolation
│
├── share/                    # XDG_DATA_HOME per user (git-ignored)
│   └── {PATH_ID}/            # User/group-specific data isolation
│
├── state/                    # XDG_STATE_HOME per user (git-ignored)
│   └── {PATH_ID}/            # User/group-specific state isolation (bash history here)
│
├── alias/                    # Executable shortcuts directory
│   ├── README.md             # Documents alias directory purpose
│   └── [user-created].sh     # User-created shell aliases (git-ignored)
│
├── example/                  # Template files for customization
│   ├── packages.nix          # Template: copy to root packages.nix and customize
│   ├── user_env.sh           # Template: user-specific env overrides (git-ignored)
│   └── config/
│       └── git/
│           ├── config        # Example Git configuration with filter setup
│           └── attributes    # Example: devcontainer.json filter
│
├── shelffiles/               # Package-specific scripts and metadata
│   └── packages/             # Shell scripts documenting package-specific setup
│       ├── bash.sh           # Bash package metadata
│       ├── zsh.sh            # Zsh package metadata
│       ├── fish.sh           # Fish package metadata
│       ├── nodejs.sh         # Node.js package metadata
│       ├── deno.sh           # Deno package metadata
│       ├── starship.sh       # Starship prompt package metadata
│       ├── claude-code.sh    # Claude Code package metadata
│       └── .shellcheckrc     # Shellcheck configuration
│
├── utils/                    # Utility scripts
│   └── build_nix_in_docker.sh # Fallback Docker-based Nix build
│
├── test/                     # Testing infrastructure
│   ├── Dockerfile            # Docker test environment definition
│   ├── packages.nix          # Nix packages for testing (bats, shells)
│   ├── config/               # Test configuration files
│   │   ├── bash/
│   │   │   └── .bashrc       # Test bash config (sets SHELFFILES_BASH_TEST)
│   │   ├── zsh/
│   │   │   └── .zshrc        # Test zsh config (sets SHELFFILES_ZSH_TEST)
│   │   └── fish/
│   │       └── config.fish   # Test fish config (sets SHELFFILES_FISH_TEST)
│   └── test/                 # BATS test suite
│       ├── bashrc_load.bats  # Bash entrypoint integration test
│       ├── zshrc_load.bats   # Zsh entrypoint integration test
│       ├── fishrc_load.bats  # Fish entrypoint integration test
│       └── alias_path.bats   # Alias directory PATH test
│
├── result/                   # Nix build output symlink (git-ignored)
│   └── bin/                  # Executable binaries from Nix packages
│
├── result_docker/            # Docker-based build output (git-ignored)
│   └── bin/                  # Fallback executable binaries
│
├── nix/                      # Nix store copy for containerized execution (git-ignored)
│   └── store/                # Nix package binaries for bubblewrap isolation
│
└── [user-created]/           # User-created directories for customization
    └── packages.nix          # User package definitions (copy from example/)
```

## Directory Purposes

**entrypoint/:**
- Purpose: Shell-specific launcher scripts and environment initialization
- Contains: Bash/Zsh/Fish wrappers, shared env setup, containerization fallback
- Key files: `env.sh` (shared), bash/zsh/fish (shell-specific), launch_in_bwrap.sh (fallback)

**config/:**
- Purpose: XDG Base Directory compliant configuration storage
- Contains: User application configurations (bash, zsh, git, nvim, etc.)
- Key pattern: `config/{app-name}/` for each application
- Special: `config/nix/` for Nix-related settings

**cache/, share/, state/:**
- Purpose: XDG-compliant per-user data directories
- Pattern: `{type}/{PATH_ID}/` to isolate multiple users
- Git-ignored: Not committed to repository
- Auto-created: By `env.sh` on first run

**alias/:**
- Purpose: User-writable directory for shell alias scripts
- Added to PATH: Via `env.sh` (before Nix packages)
- Git-ignored: User-created aliases not tracked
- Use case: Local shortcuts and wrapper scripts

**example/:**
- Purpose: Template files for user customization
- Contains: `packages.nix` template, `user_env.sh` template, example git config
- Pattern: User copies files to root or config/ and customizes
- Git strategy: Example files tracked, user copies git-ignored

**shelffiles/packages/:**
- Purpose: Package-specific documentation and metadata
- Contains: Shell scripts documenting setup for each package
- Pattern: One `.sh` file per package describing its purpose and setup
- Not executable: Informational metadata only

**utils/:**
- Purpose: Helper scripts for build and setup
- Contains: Docker-based Nix build fallback
- Called by: `setup.sh --no-root` or manual invocation

**test/:**
- Purpose: BATS test suite and Docker test environment
- Contains: Dockerfile for reproducible testing, BATS test files, test configs
- Test configs: Set environment variables (SHELFFILES_*_TEST) to verify loading
- Run: `docker build -f test/Dockerfile .` or `bats test/test/*.bats`

**result/ and result_docker/:**
- Purpose: Nix build outputs (symlinks to Nix store)
- Generated by: `nix build` or `./utils/build_nix_in_docker.sh`
- Contents: Symlink tree with /bin/ containing all installed binaries
- Git-ignored: Build artifacts not tracked

**nix/:**
- Purpose: Nix store copy for containerized execution
- Generated by: `./utils/build_nix_in_docker.sh`
- Used by: `launch_in_bwrap.sh` to access packages in bubblewrap
- Git-ignored: Not committed (too large)

## Key File Locations

**Entry Points:**
- `entrypoint/bash`: Bash shell launcher
- `entrypoint/zsh`: Zsh shell launcher
- `entrypoint/fish`: Fish shell launcher
- `setup.sh`: Setup and build CLI tool

**Configuration:**
- `flake.nix`: Nix flake definition (don't edit)
- `packages.nix`: User package list (create by copying example/packages.nix)
- `config/`: Application configurations (user-created)
- `.pre-commit-config.yaml`: Pre-commit hook definitions

**Core Logic:**
- `entrypoint/env.sh`: XDG variable setup and PATH manipulation
- `entrypoint/launch_in_bwrap.sh`: Bubblewrap containerization logic

**Testing:**
- `test/Dockerfile`: Docker test environment
- `test/test/bashrc_load.bats`: Bash integration test
- `test/test/zshrc_load.bats`: Zsh integration test
- `test/test/fishrc_load.bats`: Fish integration test
- `test/test/alias_path.bats`: Alias PATH test

## Naming Conventions

**Files:**
- Entrypoint scripts: No extension (e.g., `entrypoint/bash`, not `entrypoint/bash.sh`)
- Source files: `.sh` extension (e.g., `entrypoint/env.sh`, `entrypoint/launch_in_bwrap.sh`)
- Nix files: `.nix` extension
- Test files: `.bats` extension
- Config examples: No extension (follow app standards, e.g., `git/config` not `git/config.txt`)

**Directories:**
- Lowercase with hyphens: `pre-commit-config.yaml`, `devcontainer.json`
- XDG standard: `config/`, `cache/`, `share/`, `state/`
- Shell names: Match shell binary names (`bash/`, `zsh/`, `fish/`)
- App configs: Match application names (`git/`, `nvim/`, `starship/`)

**Variables:**
- Shell variables: UPPERCASE_WITH_UNDERSCORES (e.g., SHELFFILES, XDG_CONFIG_HOME, PATH_ID)
- Function names: lowercase_with_underscores in bash/sh

## Where to Add New Code

**New Shell Configuration:**
- Primary code: `config/{shell-name}/` (e.g., `config/bash/.bashrc`)
- Sourced automatically: Via entrypoint shell launcher
- Example: User creates `config/nvim/init.lua` for Neovim

**New Package Installation:**
- Primary code: `packages.nix` (user-created from example template)
- Edit: Add package name to array in packages.nix
- Rebuild: Run `nix build`
- Example: `pkgs: with pkgs; [ git nodejs neovim ]`

**New Entrypoint Script:**
- Shell-specific launcher: `entrypoint/{shell-name}` (if needed)
- Must source: `entrypoint/env.sh`
- Must check: For `result/` and `result_docker/` before containerization
- Fallback: Invoke `launch_in_bwrap.sh` if builds missing

**New Application Configuration:**
- Template: Copy from `example/config/git/` structure
- Location: `config/{app-name}/`
- XDG-compliant: App auto-discovers in XDG_CONFIG_HOME
- User-specific: Copy to `config/` and customize

**User-Specific Environment:**
- Location: `user_env.sh` (git-ignored, not shared)
- Purpose: User-specific env variables and overrides
- Template: See `example/user_env.sh`
- Export: `export VAR_NAME="value"` at top level

**Utilities and Helpers:**
- Shared shell helpers: `utils/` directory
- Package metadata: `shelffiles/packages/{name}.sh`
- Pre-commit hooks: Update `.pre-commit-config.yaml`

**Tests:**
- Unit/integration tests: `test/test/*.bats` files
- Test configuration: `test/config/` (app-specific test configs)
- Test packages: Update `test/packages.nix` if new deps needed
- Run locally: `bats test/test/*.bats` (requires BATS)

## Special Directories

**config/nix:**
- Purpose: Nix-specific configuration
- Contains: Nix settings, flake configuration references
- Committed: Yes
- Auto-generated: No

**cache/, share/, state/ (per-user subdirs):**
- Purpose: Application-specific state, cache, and data storage
- Generated: Automatically by `env.sh` on first run
- Committed: No (git-ignored)
- Pattern: `{type}/{PATH_ID}/` where PATH_ID is user/group specific
- Cleanup: Safe to delete (regenerated on next shell launch)

**alias/:**
- Purpose: User-writable executable directory
- Generated: Directory created by `env.sh`
- Committed: No (git-ignored)
- Added to PATH: Before Nix packages (takes precedence)

**result/ and result_docker/:**
- Purpose: Nix build output symlinks
- Generated: By `nix build` or Docker build
- Committed: No (git-ignored)
- Content: Symlink tree to Nix store binaries
- Life cycle: Removed by `nix flake update` or manual cleanup

**nix/:**
- Purpose: Nix store copy for bubblewrap (fallback build)
- Generated: By `./utils/build_nix_in_docker.sh`
- Committed: No (git-ignored, too large)
- Used by: `launch_in_bwrap.sh` when Nix unavailable
- Cleanup: Safe to delete (regenerated via Docker build)

---

*Structure analysis: 2026-01-28*
