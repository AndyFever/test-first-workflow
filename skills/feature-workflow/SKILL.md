---
name: feature-workflow
description: Use when starting work on a ticket, feature, user story, or bug fix that will change production code. Runs the full test-first workflow in order - plan the work, plan acceptance tests, build with isolated-unit TDD, review coverage - with human checkpoints between phases. Start here instead of jumping straight into code or TDD.
---

# Feature Workflow

Planning delivery into the begininning of the development is key for effective development. This
set of skills enables the enineer to use agents to plan and execute the feature workflow in collaboration with each
other with the agreed goals in mind.

This skill allows the engineer to run a combination of workflows that fits with their development goals 
and the risk associated with the feature.

## Phases

| # | Skill to load | Output (in `docs/work/<ticket-id>/`) | Checkpoint |
|---|---|---|---|
| 1 | `work-planning` | `plan.md` | Engineer agrees the component breakdown |
| 2 | `acceptance-test-planning` | `test-plan.md` | Engineer approves risks and test levels |
| 3 | `isolated-tdd` (per component) | Code + tests | Acceptance tests for the component pass |
| 4 | `coverage-review` | `coverage-review.md` | Engineer reviews gaps before calling it done |

When installed as a plugin, the skills are namespaced, e.g.
`test-first-workflow:work-planning`. Load each skill when its phase starts,
not before; follow its instructions in full.

## How to run it

1. **Find the ticket.** Get the ticket number and title, description and acceptance
   criteria. If the engineer gave only a summary, ask for the acceptance criteria
   or agree them in phase 1. Use a short slug if there is no ticket ID.
2. **Check for an understanding of the data.** Analyse the ticket and any type files for the data model. 
   Make sure we have a good understanding of what any mocks or fixtures would look like.
3. **Run phases 1 and 2**, stopping at each checkpoint. Do not write
   production code or tests during planning.
4. **Phase 3: double loop.** For each component in the agreed order:
   - *Outer loop:* write the acceptance tests from `test-plan.md` that this
     component makes reachable. Run them and watch them fail for the right
     reason.
   - *Inner loop:* load `isolated-tdd` and build the component unit by unit
     until those acceptance tests pass.
   - Tick the component off in `plan.md`.
5. **Phase 4.** Load `coverage-review`. Fix any gaps it finds with the same
   test-first loop, then re-run the review.
6. **Finish** with a short summary: what was built, the test counts per level,
   the full-suite result, and anything left open.

## Checkpoints

A checkpoint means: show the engineer the output file's key points, ask the
specific questions it raises, and wait. A checkpoint is passed only when the
engineer says so in the conversation. Record the approval (date and any changes)
at the top of the output file.

If the human wants to skip a phase, that is their call. Record it and why in
`plan.md` so the coverage review knows.

## Rules

- No production code before phase 3.
- Planning outputs are living documents. If phase 3 shows the plan was wrong,
  update `plan.md` / `test-plan.md` and tell the engineer, rather than quietly
  drifting from it.
- Keep the outputs short. They are working notes, not reports.
