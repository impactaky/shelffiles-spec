# nix-flake Specification

## Purpose
TBD - created by archiving change move-packages-nix-to-example. Update Purpose after archive.
## Requirements
### Requirement: User Package Configuration Separation

The system SHALL support user-specific package customization without git conflicts by:
- Providing `example/packages.nix` as a template with example packages
- Requiring users to copy `example/packages.nix` to root `packages.nix` before building
- Not tracking root `packages.nix` in upstream repository

#### Scenario: New user setup

- **WHEN** a user clones the repository for the first time
- **THEN** they copy `example/packages.nix` to root `packages.nix`
- **AND** customize packages as needed
- **AND** `nix build` completes successfully

#### Scenario: Upstream updates

- **WHEN** upstream updates `example/packages.nix` with new example packages
- **AND** user has their own root `packages.nix`
- **THEN** `git pull` does not cause merge conflicts
- **AND** user's root `packages.nix` remains unchanged

