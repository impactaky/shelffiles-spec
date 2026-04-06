# Workflow

You need to follow the steps strictly.
Proceed through the steps autonomously unless blocked, destructive approval is required, or user input is necessary to resolve ambiguity.
If `.myagents/dod.md` exists, treat user comments as feedback on the current workflow until the task is done.

## `dod.md` Format

```md
## User request

- [ ] <What the user asked for, restated concretely and minimally>

## Relevant context

- [ ] <Confirmed fact from the repository, API, requirements, or runtime behavior>

## Required changes based on facts

- [ ] <Concrete change required by the facts above>

## Constraints

- [ ] <Implementation or scope constraint derived from the facts above>

## Verification for user request

- [ ] <Test, command, manual check, or review point that confirms the user request is satisfied>

## Deferred

- <Unresolved item that is safe to postpone>
```

## Steps

### 1. Prepare DoD

- [ ] Confirm the relevant facts from the user request and current repository context
- [ ] If critical ambiguity remains, ask the user in one batch before implementation
- [ ] Create `.myagents/dod.md` with User request, Relevant context, Required changes, Constraints, Verification for user request, and Deferred

### 2. Implement And Internal Review

Repeat until implementation and review pass, up to 5 times.

- [ ] You may add scaffolding tests during implementation, but remove them before finishing. Keep only tests for the user request
- [ ] Implement the change to satisfy `.myagents/dod.md` in `programmer` agent
- [ ] Review the implementation against `.myagents/dod.md` and `project-rules.md` if present in `reviewer` agent

### 3. User Review

- [ ] Output the user review report in Japanese
- [ ] If the user requests follow-up changes, rerun Step 2
- [ ] If follow-up fixes a reusable review issue, update `project-rules.md` before rerunning Step 2

#### User Review Format

```md
## Review report

### Request
- <Request>

### Before
- <Before state>
- <Why it was not satisfied>

### Resolution
- [path/to/file.ext:line] <Change>. <Why it matters>
- [path/to/file.ext:line] <Change>. <Why it matters>
```

- `Resolution` must cover all meaningful changes.
- Each bullet must include a file path and line.
- Each bullet must say what changed and why.

### 4. Archive

- [ ] Save the LGTM report to `.myagents/artifacts/review/<task-summary>-<YYYYMMDD-HHMMSS>.md`
