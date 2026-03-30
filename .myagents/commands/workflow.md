# Workflow

You need to follow the steps strictly.
Proceed through the steps autonomously unless blocked, destructive approval is required, or user input is necessary to resolve ambiguity.
If `.myagents/dod.md` exists, treat user comments as feedback on the current workflow until it is archived.

## Steps

### 1. Prepare DoD

- [ ] Confirm the relevant facts from the user request and current repository context
- [ ] Create `.myagents/dod.md` with Relevant context, Required changes, Constraints, Verification, Open Questions, and Deferred

If `Open Questions` is non-empty, ask the user in one batch and update `.myagents/dod.md`.
Do not proceed to implementation while `Open Questions` is non-empty.

### 2. Implement And Review

Repeat until implementation and review pass, up to 5 times.

- [ ] Implement the change to satisfy `.myagents/dod.md` in `programmer` agent
- [ ] Review the implementation against `.myagents/dod.md` and `project-rules.md` if present in `reviewer` agent

### 3. Report And Iterate

- [ ] Output the DoD check report in Japanese.
- [ ] Wait for user review. If you get feedback, rerun the implementation loop.

### 4. Finalize

- [ ] Update `.myagents/dod.md` to match the final result
- [ ] Ensure `Open Questions` is empty
- [ ] Save a copy to `.myagents/artifacts/dod/<task-summary>-<YYYYMMDD-HHMMSS>.md`
- [ ] If the workflow exposed a reusable project-specific rule, reflect it in `project-rules.md`.

## `dod.md` Format

```md
## Relevant context

- [ ] <Confirmed fact from the repository, API, requirements, or runtime behavior>

## Required changes based on facts

- [ ] <Concrete change required by the facts above>

## Constraints

- [ ] <Implementation or scope constraint derived from the facts above>

## Verification

- [ ] <Command, test, manual check, or review point that confirms the change is correct>

## Open Questions

- <Critical ambiguity that blocks confirmation, change design, or verification>

## Deferred

- <Unresolved item that is safe to postpone>
```
