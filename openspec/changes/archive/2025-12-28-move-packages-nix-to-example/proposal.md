## Why

Users who fork or clone shelffiles will customize `packages.nix` with their own package selections. When they pull upstream changes, conflicts occur because both upstream and the user modify the same file. Moving `packages.nix` to `example/` eliminates this conflict by keeping user-customized files separate from repository defaults.

## What Changes

- Create `example/packages.nix` as a template for users to copy
- Remove `packages.nix` from upstream git tracking (users copy from example and track in their forks)
- Update README to explain the new workflow: copy `example/packages.nix` to root before building

## Impact

- Affected specs: nix-flake
- Affected code: `flake.nix`, `packages.nix`, `README.md`, `example/packages.nix`
- Users can now track their own `packages.nix` in their forks without merge conflicts from upstream
