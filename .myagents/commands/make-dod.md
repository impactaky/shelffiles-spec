# Make DoD

Create or update `dod.md` for the current change.

## Input

User request describing the desired change.

## Output

`openspec/changes/<name>/dod.md`

## Steps

- [ ] Draft `dod.md` from the user request and available change artifacts
- [ ] If `Must Resolve` is non-empty, ask the user and update `dod.md`

## `dod.md` Format

```md
# Definition of done

## DoD list

### 1. <Group name>

- [ ] <Specific, verifiable completion criterion>

## Must Resolve

- <Critical gray area that must be resolved before the DoD is finalized>

## Deferred

- <Unresolved item that is safe to postpone>
```

## Rules

- Keep `Content` specific and verifiable
- Use `Must Resolve` for gray areas that affect scope, implementation direction,
  verification, or user-visible impact
- Use `Deferred` for unresolved items that are safe to postpone
- Do not ask one question at a time while drafting; batch critical review points
- Do not proceed to implementation while `Must Resolve` is non-empty
