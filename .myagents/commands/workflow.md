# Workflow

This is the workflow to develop.
**IMPORTANT** You need to follow **Steps** strictly.
Proceed through the steps autonomously unless blocked, destructive approval is required, or user input is necessary to resolve ambiguity.

## Input

User request.
It may include DoD content.

## Output

DoD check report.
Explain how each DoD is satisfied by the implementation.
Output the DoD report in Japanese.

## Steps

- [ ] `.myagents/commands/make-dod.md`
- [ ] Loop until implementation and review pass. Up to 5 times.
  - [ ] Implement the change to satisfy `dod.md`
  - [ ] Review the implementation against `dod.md` and `openspec/specs/dod-points/spec.md`
- [ ] Output the DoD check report in Japanese.
- [ ] Wait for user review. If you get feedback, rerun the implementation loop.
- [ ] If the feedback is essential, reflect it in `openspec/specs/dod-points/spec.md` to improve future runs.
