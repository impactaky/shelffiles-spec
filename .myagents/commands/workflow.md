# Workflow

This is the workflow to develop.
**IMPORTANT** You need to follow **Steps** strictly.
Proceed through the steps autonomously unless blocked, destructive approval is required, or user input is necessary to resolve ambiguity.

## Input

User request.
It might contain DoD.

## Output

DoD check report.
Explain how each DoD is satisfied by the implementation.
DoD report should be output in Japanese.

## Steps

- [ ] `.myagents/prompts/opsx-new.md <input>`
- [ ] `.myagents/prompts/opsx-ff.md`
- [ ] Loop these steps until verify and review pass. Up to 5 times.
  - [ ] `.myagents/prompts/opsx-apply.md` in `programmer` agent
  - [ ] `.myagents/prompts/opsx-verify.md` and review in `reviewer` agent
- [ ] `.myagents/prompts/opsx-sync.md`
- [ ] Output the DoD check report in Japanese.
- [ ] Wait user review. If you got feedback, rerun implementation loop
- [ ] If there is a essential feedback, reflectiing feedback to `openspec/specs/dod-points/spec.md` to improve next run
- [ ] `.myagents/prompts/opsx-archive.md`

