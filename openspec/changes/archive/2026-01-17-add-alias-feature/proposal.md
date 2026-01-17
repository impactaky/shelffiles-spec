## Why
Shell aliasing does not work consistently in user scripts and AI agent contexts because there is no shared alias directory on PATH.

## What Changes
- Add a global alias directory at `shelffiles/alias`.
- Ensure `shelffiles/alias` is added to PATH for entrypoint shells and user scripts.

## Impact
- Affected specs: configure-aliases
- Affected code: `shelffiles/entrypoint/env.sh`, repository layout
