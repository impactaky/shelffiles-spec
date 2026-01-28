# Testing Patterns

**Analysis Date:** 2026-01-28

## Test Framework

**Framework:**
- BATS (Bash Automated Testing System)
- Version: Not explicitly specified in codebase (standard BATS syntax used)
- Invocation: `bats test/` or individual test files

**Test Execution:**

```bash
# Docker-based test execution (recommended)
docker build -f test/Dockerfile .

# Individual test execution (requires bats installed)
bats test/test/bashrc_load.bats
bats test/test/zshrc_load.bats
bats test/test/fishrc_load.bats
bats test/test/alias_path.bats
```

**Docker Test Environment:**
- Base image: `nixos/nix:2.24.14`
- Enables flakes: `experimental-features = nix-command flakes`
- Build environment: Full Nix build with `nix build`
- Path setup: `ENV PATH="/app/result/bin:$PATH"`
- Test execution: `RUN bats test`

## Test File Organization

**Location:**
- Co-located in single test directory: `test/test/`
- Config files separate: `test/config/`

**Naming Convention:**
- Pattern: `{component}_load.bats` or `{component}_path.bats`
- Examples:
  - `bashrc_load.bats` - Tests bash configuration loading
  - `zshrc_load.bats` - Tests zsh configuration loading
  - `fishrc_load.bats` - Tests fish configuration loading
  - `alias_path.bats` - Tests PATH manipulation

**Test File Structure:**

```
test/
├── test/
│   ├── bashrc_load.bats       # Bash entrypoint tests
│   ├── zshrc_load.bats        # Zsh entrypoint tests
│   ├── fishrc_load.bats       # Fish entrypoint tests
│   └── alias_path.bats        # PATH and alias tests
├── config/
│   ├── bash/
│   │   └── .bashrc            # Test bash config
│   ├── zsh/
│   │   └── .zshrc             # Test zsh config
│   └── fish/
│       └── config.fish        # Test fish config
├── Dockerfile                 # Test runner container
└── packages.nix              # Test environment packages
```

## Test Structure

**Test Pattern:**
```bash
#!/usr/bin/env bats

setup() {
  # Clear or initialize test environment variables
  unset SHELFFILES_BASH_TEST
}

@test "descriptive test name" {
  # Arrange: Set up test

  # Act: Run command
  run ./entrypoint/bash -i -c 'echo $SHELFFILES_BASH_TEST'

  # Debug output (optional but recommended)
  echo "Status: $status"
  echo "Output: $output"

  # Assert: Check results
  [ "$status" -eq 0 ]
  [[ "$output" == *"loaded" ]]
}
```

**Key Components:**
- `setup()` - Initializes test state, unsets variables that might affect tests
- `@test "description"` - Test case declaration
- `run` - BATS command to execute a command and capture output/status
- `$status` - Exit code of last run command
- `$output` - Combined stdout/stderr of last run command
- Echo statements for debugging during test runs
- Assertions using bash conditionals and string matching

## Test Configuration Files

**Test Bash Config (`test/config/bash/.bashrc`):**
```bash
# shellcheck shell=bash
export SHELFFILES_BASH_TEST="loaded"
```

**Test Zsh Config (`test/config/zsh/.zshrc`):**
```bash
# shellcheck shell=bash
export SHELFFILES_ZSH_TEST="loaded"
```

**Test Fish Config (`test/config/fish/config.fish`):**
Fish-specific config syntax with equivalent environment variable export.

**Purpose:**
- Each test config sets a unique environment variable
- Tests verify this variable is set after shell initialization
- Confirms that shell-specific entrypoints correctly load configuration files from `config/` directories

## Test Patterns

**Configuration Loading Test:**
```bash
@test "entrypoint/bash loads config/bash/.bashrc implicitly via BASH_ENV" {
  run ./entrypoint/bash -i -c 'echo $SHELFFILES_BASH_TEST'

  echo "Status: $status"
  echo "Output: $output"

  [ "$status" -eq 0 ]
  [[ "$output" == *"loaded" ]]
}
```

**Explanation:**
- Invokes entrypoint with `-i` (interactive) and `-c` (command)
- Command echoes environment variable set by sourced config
- Passes if status is 0 and variable is present in output

**PATH Environment Test:**
```bash
@test "entrypoint/bash prepends alias directory to PATH" {
  run ./entrypoint/bash -i -c 'echo $PATH'

  echo "Status: $status"
  echo "Output: $output"

  [ "$status" -eq 0 ]
  [[ "$output" == *"$alias_dir"* ]]
}
```

**Explanation:**
- Verifies that alias directory is in PATH
- Uses setup() to derive repo root and alias directory location
- Checks for substring presence with wildcard matching

**Fish-Specific Pattern:**
```bash
@test "entrypoint/fish loads config/fish/config.fish implicitly" {
  # shellcheck disable=SC2016
  run ./entrypoint/fish -i -c 'echo $SHELFFILES_FISH_TEST'

  echo "Status: $status"
  echo "Output: $output"

  [ "$status" -eq 0 ]
  [[ "$output" == *"loaded" ]]
}
```

**Differences:**
- Uses `SC2016` disable for literal variable references (Fish syntax differs)
- Allows wildcards in output matching (`*"loaded"`) because Fish may emit warnings

## Setup and Teardown

**Setup Pattern:**
```bash
setup() {
  unset SHELFFILES_BASH_TEST
  # Or for path tests:
  repo_root="$(CDPATH='' cd -- "$BATS_TEST_DIRNAME" && pwd)"
  while [ ! -d "$repo_root/entrypoint" ] && [ "$repo_root" != "/" ]; do
    repo_root="$(dirname "$repo_root")"
  done
  alias_dir="$repo_root/alias"
}
```

**Key Variables:**
- `$BATS_TEST_DIRNAME` - Directory containing current test file
- `$status` - Exit code of `run` command
- `$output` - Output of `run` command

**Teardown:**
- Not used in current test suite
- BATS cleans up automatically; no explicit cleanup needed

## Test Data & Fixtures

**Approach:**
- Minimal test data - primarily uses environment variables as markers
- Test configuration files (`test/config/*/`) serve as fixtures
- No separate fixture files or factories

**Test Marker Pattern:**
- Uses environment variables to signal configuration loading success
- Example: `SHELFFILES_BASH_TEST="loaded"`
- Simpler than return codes; visible in shell output

## Assertions

**Assertion Patterns:**

```bash
# Status code assertion
[ "$status" -eq 0 ]

# Exact string match
[[ "$output" == "loaded" ]]

# Substring match with wildcards
[[ "$output" == *"loaded" ]]
[[ "$output" == *"$alias_dir"* ]]

# Wildcard at beginning
[[ "$output" == *"loaded" ]]
```

**Available Operators:**
- `[ "$status" -eq 0 ]` - Exit code equals
- `[ "$status" -ne 1 ]` - Exit code not equals
- `[[ "$output" == pattern ]]` - Bash pattern matching
- `[[ "$output" =~ regex ]]` - Regex matching (not used in current tests)

## Test Isolation

**Environment Isolation:**
- Each test runs in isolated subprocess via `run` command
- Variables are unset before each test in `setup()`
- No state shared between tests

**Filesystem Access:**
- Tests run from repository root (uses `./entrypoint/...` paths)
- Can read actual shell configurations and files
- No mocking of filesystem

## Docker Test Infrastructure

**Dockerfile Location:** `test/Dockerfile`

**Build Process:**
```dockerfile
FROM nixos/nix:2.24.14

# Enable flakes
RUN mkdir -p /etc/nix && \
    echo 'experimental-features = nix-command flakes' >> /etc/nix/nix.conf

WORKDIR /app

# Copy files needed for nix build
COPY . .
COPY test/packages.nix ./packages.nix

# Create environment
RUN nix build

# Copy test infrastructure
COPY entrypoint ./entrypoint/
COPY test/config ./config/
COPY test/test ./test/

ENV PATH="/app/result/bin:$PATH"

# Run tests
RUN bats test
```

**Key Steps:**
1. Base image has Nix with flakes enabled
2. Copy repository and override packages.nix with test version
3. Run full nix build to create environment
4. Copy test files and entrypoints
5. Add result/bin to PATH (contains BATS)
6. Execute: `bats test`

**Test Command:**
```bash
docker build -f test/Dockerfile .
```

**Advantages:**
- Reproducible environment across machines
- Full Nix build executed - tests real output
- No dependency on local Nix installation
- Self-contained test environment

## Coverage

**Coverage Measurement:**
- Not measured in current test suite
- No code coverage tools configured
- Tests focus on integration (shell loading behavior) not unit coverage

**Test Scope:**
- **Bash entrypoint:** Config loading via BASH_ENV
- **Zsh entrypoint:** Config loading via ZDOTDIR
- **Fish entrypoint:** Config loading via fish startup files
- **PATH manipulation:** Alias directory prepended to PATH

**Untested Areas:**
- `launch_in_bwrap.sh` containerization logic
- `launch_with_nsenter.sh` sudo mount logic
- Nix package resolution
- Complex argc flag parsing in setup.sh
- Error handling edge cases

---

*Testing analysis: 2026-01-28*
