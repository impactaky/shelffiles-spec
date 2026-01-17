## ADDED Requirements
### Requirement: Global Alias Directory
The system SHALL provide a `shelffiles/alias` directory for user-defined executable aliases and ensure it is included in PATH when shelffiles environment setup is applied.

#### Scenario: Alias available in entrypoint shells
- **WHEN** a user adds an executable to `shelffiles/alias`
- **AND** launches a shell via `shelffiles/entrypoint/*`
- **THEN** the alias is available on PATH

#### Scenario: Alias available in user scripts
- **WHEN** a script sources `shelffiles/entrypoint/env.sh`
- **THEN** the script resolves executables in `shelffiles/alias` via PATH
