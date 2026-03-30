# Workflow

This is the workflow to develop.
You need to follow the steps strictly.
Proceed through the steps autonomously unless blocked, destructive approval is required, or user input is necessary to resolve ambiguity.

## Steps

### 1. Prepare DoD

- [ ] Confirm the relevant facts from the user request and current repository context
- [ ] Create `dod.md` with Facts, Required changes, Constraints, Verification, Open Questions, and Deferred
- [ ] If `Open Questions` is non-empty, ask the user in one batch and update `dod.md`
- [ ] Do not proceed to implementation while `Open Questions` is non-empty

### 2. Implement And Review

- [ ] Loop until implementation and review pass. Up to 5 times.
  - [ ] Implement the change to satisfy `dod.md` in `programmer` agent
  - [ ] Review the implementation against `dod.md` and `project-rules.md` if present in `reviewer` agent

### 3. Report And Iterate

- [ ] Output the DoD check report in Japanese.
- [ ] Wait for user review. If you get feedback, rerun the implementation loop.

### 4. Finalize

- [ ] Update `dod.md` to match the final result
- [ ] Ensure `Open Questions` is empty
- [ ] Save a copy to `.myagents/artifacts/dod/<task-summary>-<YYYYMMDD-HHMMSS>.md`
- [ ] If the feedback is essential, reflect it in `project-rules.md` to improve future runs.

## `dod.md` Format

```md
## Facts

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
