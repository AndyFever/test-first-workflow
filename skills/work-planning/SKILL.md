---
name: work-planning
description: Use when planning the work for a ticket, feature, or bug fix before any code is written. Confirms TDD applies, analyses the ticket's features and acceptance criteria, and works with the human to break the work into small, testable components. Produces docs/work/<ticket-id>/plan.md.
---

# Work Planning

Turn a ticket into an agreed list of small components that can each be built
test-first. The human decides the breakdown; your job is to analyse, propose
and ask good questions.

## 1. Confirm the approach

Assume TDD. Then check whether this work is one of the exceptions:

- Throwaway prototype or spike
- Generated code
- Configuration only

If it might be, ask the human. If they agree it is an exception, record that
in `plan.md` and stop the test-first workflow for that part only.

## 2. Analyse the ticket

Read the ticket and any linked material. Look at the code it touches. Extract:

- **Features:** what the user or system can do after this change.
- **Acceptance criteria:** as written. Number them `AC1`, `AC2`, ...
- **Implied criteria:** behaviour the ticket assumes but doesn't state
  (errors, empty states, permissions, existing behaviour that must not change).
  Number them `AC-i1`, `AC-i2`, ... and mark them *proposed*.
- **Data:** the main data the change creates, reads or changes, and its shape.
- **Questions:** anything ambiguous. Do not guess an answer to a question that
  changes behaviour; ask.

## 3. Propose components

Break the work into components that are each:

- **Small:** one responsibility, buildable in a short TDD session.
- **Isolated:** its collaborators can be named, and mocked in unit tests.
- **Contract-clear:** you can say what it receives, returns and fails with.

For each component, list its collaborators and the contract with each one
(method/endpoint, input, output, errors). These contracts become the mocks in
unit tests and the targets for integration tests.

Suggest a build order: usually the component nearest the acceptance criteria's
entry point first, mocking what it depends on, then work inward.

## 4. Work it through with the human

Present the analysis and the proposed breakdown, then ask:

- Are the acceptance criteria and implied criteria right?
- The open questions.
- Does the breakdown and build order make sense? Too big, too small, missing?

Revise until the human agrees. Do not move on without that agreement.

## Output: `docs/work/<ticket-id>/plan.md`

```markdown
# <ticket-id>: <title>
Approved: <date> by human (<any notes>)

## Approach
TDD: yes | exceptions: <what and why>

## Acceptance criteria
- AC1: ...
- AC-i1 (implied, agreed): ...

## Data
<entities/payloads touched, key fields and constraints>

## Components (build order)
### 1. <ComponentName> — <one-line responsibility>
- Collaborators and contracts:
  - <Collaborator>.<operation>(<input>) → <output> | errors: <...>
- Covers: AC1, AC-i1
- [ ] Done

## Open questions / assumptions / out of scope
```
