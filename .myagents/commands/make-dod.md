# Make DoD

Create `dod.md`.

## Input

User request describing the desired change.

## Output

Working `dod.md`

## Steps

- [ ] Confirm the relevant facts from the user request and current repository context
- [ ] Derive `Required changes based on facts`
- [ ] Define `Verification`
- [ ] Put unresolved blocking items in `Open Questions`
- [ ] Put safe-to-postpone items in `Deferred`
- [ ] If `Open Questions` is non-empty, ask the user in one batch and update the items
- [ ] Write the final items to `dod.md`
- [ ] Treat `dod.md` as the editable working copy until the workflow reaches finalization

## `dod.md` Format

```md
# Definition of done

## Facts

- [ ] <Confirmed fact from the repository, API, requirements, or runtime behavior>
- [ ] <Why this fact matters to the requested change>

## Required changes based on facts

- [ ] <Concrete change that becomes necessary once the facts above are confirmed>
- [ ] <Scope or implementation constraint derived from those facts>

## Verification

- [ ] <How to confirm the change is correct>
- [ ] <Command, test, manual check, or review point with a verifiable result>

## Open Questions

- <Critical ambiguity that blocks confirmation, change design, or verification>

## Deferred

- <Unresolved item that is safe to postpone>
```

## Rules

- Keep items specific and verifiable
- Do not proceed to implementation while `Open Questions` is non-empty
- Store the first complete draft at the repository root as `dod.md`
