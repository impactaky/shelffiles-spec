# Coding Conventions

**Analysis Date:** 2026-01-28

## Language & Shell Compatibility

**Primary Language:** Shell (Bash/POSIX sh, Zsh, Fish)

**Compatibility Approach:**
- Core scripts use POSIX sh syntax where possible (e.g., `env.sh` declares `# shellcheck shell=sh`)
- Bash-specific scripts explicitly declare their shell target (e.g., `setup.sh` is bash)
- Non-core shell integrations (Fish, Zsh) have separate config files (`config/fish/config.fish`, `config/zsh/.zshrc`)

## Naming Patterns

**Environment Variables:**
- ALL_CAPS with underscores for exported variables
- Example: `SHELFFILES`, `XDG_CONFIG_HOME`, `XDG_CACHE_HOME`, `PATH_ID`, `USER_ID`, `GROUP_ID`
- Test indicators use `SHELFFILES_*_TEST` pattern: `SHELFFILES_BASH_TEST`, `SHELFFILES_ZSH_TEST`, `SHELFFILES_FISH_TEST`

**Functions:**
- snake_case with underscores for internal functions
- Single-letter or short function names not used; descriptive names preferred
- Example: `mkdir -p`, `cd --`, `export` statements as primary execution pattern

**Files:**
- `.sh` extension for shell scripts
- `.nix` extension for Nix configuration
- Lowercase with hyphens for multi-word filenames
- Examples: `env.sh`, `launch_in_bwrap.sh`, `flake.nix`, `packages.nix`, `shellrc_load.bats`

**Directories:**
- Lowercase with no hyphens: `entrypoint`, `config`, `alias`, `shelffiles`, `cache`, `share`, `state`
- XDG base directory layout: `config/`, `cache/`, `share/`, `state/`

## Code Style

**Formatting:**
- Indentation: 2 spaces (observed in most scripts)
- No tabs used
- Blank lines between logical sections

**Variable Declaration & Usage:**
```bash
# Proper quoting for paths with potential spaces
SCRIPT_DIR="$(CDPATH='' cd -- "$(dirname -- "$0")" && pwd)"
SHELFFILES="$(CDPATH='' cd -- "$SCRIPT_DIR/.." && pwd)"
export SHELFFILES="$SHELFFILES"

# XDG variables always quoted
export XDG_CONFIG_HOME="$SHELFFILES/config"
export XDG_CACHE_HOME="$SHELFFILES/cache/${PATH_ID}"

# Conditional variable assignment with default
if [ -z "$SHELFFILES" ]; then
  echo "Error: SHELFFILES variable is not set..."
  return 1 2>/dev/null
fi
```

**Error Handling:**
- Set `set -eu` at script start for strict error handling (see `launch_in_bwrap.sh`, `setup.sh`)
- Exit codes checked explicitly in conditionals: `[ "$status" -eq 0 ]`
- Error messages sent to stderr: `echo "Error: ..." >&2`
- Early return/exit on missing dependencies or invalid state

**Example from `launch_in_bwrap.sh` (lines 1-15):**
```bash
#!/bin/bash

set -eu  # Enable strict mode

# Source environment and config
SCRIPT_DIR="$(CDPATH='' cd -- "$(dirname -- "$0")" && pwd)"
# shellcheck disable=SC1091
. "$SCRIPT_DIR/env.sh"

NIX_HOST_PATH="${SHELFFILES}/nix"

if [ ! -d "$NIX_HOST_PATH" ]; then
  echo "Error: Directory not found: ${NIX_HOST_PATH}" >&2
  exit 1
fi
```

## Linting & Static Analysis

**ShellCheck:**
- Enabled in pre-commit hooks (see `.pre-commit-config.yaml` line 9-12)
- Configuration: `shellcheck -x -P entrypoint`
- Directives used to suppress specific checks:
  - `# shellcheck disable=SC1091` for dynamically sourced files
  - `# shellcheck disable=SC1090` for user-provided env files
  - `# shellcheck disable=SC2016` for literal variable references in tests
  - `# shellcheck shell=sh` or `# shellcheck shell=bash` to specify target shell

**Pre-commit Hooks:**
- `shellcheck-py` for shell script linting
- `hadolint` for Dockerfile linting
- `nixfmt` for Nix formatting (local hook)
- `trailing-whitespace` check enabled
- `check-yaml` for YAML validation
- `check-added-large-files` to prevent large file commits
- `check-merge-conflict` to catch merge conflict markers

**Nix Linting:**
- Run: `nixfmt -c` on all `.nix` files

## Import Organization

**Shell Sourcing Pattern:**
1. Script initialization (shebang, set options)
2. Directory/path setup and validation
3. Environment variable sourcing and configuration files (`. "$SHELFFILES/config/shelffiles.conf"`)
4. User environment file sourcing (`. "$USER_ENV_FILE"` with proper escaping)
5. Function definitions and execution

**Example from `env.sh` (lines 1-11):**
```bash
# shellcheck shell=sh

SCRIPT_DIR="$(CDPATH='' cd -- "$(dirname -- "$0")" && pwd)"
SHELFFILES="$(CDPATH='' cd -- "$SCRIPT_DIR/.." && pwd)"
export SHELFFILES="$SHELFFILES"

# Load shelffiles configuration
if [ -f "$SHELFFILES/config/shelffiles.conf" ]; then
    # shellcheck disable=SC1091
    . "$SHELFFILES/config/shelffiles.conf"
fi
```

## Comments

**Documentation:**
- Comments explain "why", not "what" (code is clear enough)
- Line comments use `#` with single space: `# Comment here`
- Comments placed above code they describe, not at end of line

**Examples from codebase:**
```bash
# Create a unique ID based on the path, user ID and group ID to avoid conflicts
USER_ID=$(id -u)
GROUP_ID=$(id -g)
PATH_ID=$(echo "${SHELFFILES}_${USER_ID}_${GROUP_ID}" | tr '/:' '__')

# Create necessary directories
mkdir -p "$XDG_CACHE_HOME"

# Exclude special directories/mount points
case "$entry_name" in
  proc|dev|tmp|sys|run|nix|lost+found) # Exclude special directories/mount points
    ;;
```

**ShellCheck Directives:**
- Placed immediately before problematic line
- Specific rule codes always provided
- Format: `# shellcheck disable=SC<code>` or `# shellcheck shell=<shell>`

## Function Design

**Functions Not Widely Used:**
- Scripts primarily contain sequential procedural code with `set -eu` for safety
- When logic repeats, variables or subshells handle parameterization
- Argc-generated setup.sh shows generated function structure with complex parsing

**Parameter Passing:**
- Environment variables used for state across script invocations
- Positional arguments captured by argc framework in `setup.sh`
- Subshells used for isolated execution contexts (e.g., in bwrap)

## Module Design

**Package Organization:**
- Small individual shell scripts per package (`shelffiles/packages/bash.sh`, `shelffiles/packages/nodejs.sh`)
- Each package script exports minimal environment setup
- Central `env.sh` coordinates and sources all pieces

**Example package file (`shelffiles/packages/bash.sh`):**
```bash
mkdir -p "$XDG_STATE_HOME"/bash
export HISTFILE="$XDG_STATE_HOME"/bash/history
```

**Entrypoint Design:**
- Shell-specific launchers (`entrypoint/bash`, `entrypoint/zsh`, `entrypoint/fish`)
- All source `entrypoint/env.sh` for consistent environment
- Fallback to containerized execution via `launch_in_bwrap.sh` when needed

## Argc CLI Framework Integration

**setup.sh Structure:**
- Lines 1-201: Argc-generated boilerplate (DO NOT MANUALLY EDIT)
- Regenerate with: `argc --argc-build setup.sh > setup.sh`
- Lines 202+: Implementation logic
- Argc enforces flag parsing and usage generation
- Flag pattern: `@flag --flag-name     Description here`
- Arg pattern: `@arg name~      Description (tilde = variadic)`

## Path Handling Best Practices

**CDPATH='' idiom:**
- Used universally to avoid `cd` following CDPATH directories
- Pattern: `CDPATH='' cd -- "$(dirname -- "$0")" && pwd`
- Prevents directory traversal surprises

**Path Concatenation:**
- Use `"$VAR/subpath"` not `$VAR/subpath` (prevents word splitting on spaces)
- Use `--` after `cd` to prevent filename arguments being interpreted as options

---

*Convention analysis: 2026-01-28*
