# test-first-workflow

A set of Claude Code skills for delivering a ticket test-first. Testing is never
an isolated step: it starts in planning and finishes with a coverage check
against the ticket.

```
 ┌──────────────┐   ┌──────────────────────┐   ┌──────────────┐   ┌─────────────────┐
 │ 1. Plan the  │──▶│ 2. Plan acceptance   │──▶│ 3. Build with│──▶│ 4. Review       │
 │    work      │   │    tests (risk-based)│   │  isolated TDD│   │    coverage     │
 └──────┬───────┘   └──────────┬───────────┘   └──────────────┘   └─────────────────┘
        │ human agrees         │ human approves
        ▼ breakdown            ▼ test plan
```

## Skills

| Skill | What it does | Output |
|---|---|---|
| `feature-workflow` | Runs the phases in order, with human checkpoints between them. Start here. | — |
| `work-planning` | Checks TDD applies, analyses the ticket, and agrees a component breakdown with you. | `plan.md` |
| `acceptance-test-planning` | Decides whether acceptance tests are needed, ranks risk, and places each test at the lowest level that can prove it (integration or e2e). | `test-plan.md` |
| `isolated-tdd` | Red-green-refactor on small, isolated units. Collaborators are mocked behind explicit contracts, with deliberate test data. | Code + tests |
| `coverage-review` | Maps every acceptance criterion and mocked contract to a real test. Excludes non-functional testing. | `coverage-review.md` |

Each phase writes its output to `docs/work/<ticket-id>/` in the project, so the
next phase (and you) can read it. Every skill also works on its own, e.g. run
`coverage-review` on an existing branch.

## Testing approach

Unit tests follow the **mockist (London-school)** style: a unit is a small part
of the application, tested in isolation. Collaborators are mocked, and every
mock stands for an explicit contract (inputs, outputs, errors, data shape).

Mockist testing has two known weaknesses, and the workflow covers both:

1. **Mocks can drift from the real component.** Every mocked contract must be
   confirmed by an integration or contract test. `coverage-review` checks this.
2. **Tests can lock in the implementation.** `isolated-tdd` mocks roles, not
   values; wraps third-party code in adapters; and asserts on outcomes, checking
   interactions only when the interaction *is* the behaviour.

## Install

From a private GitHub repo (needs git access to the repo):

```
/plugin marketplace add <your-github-user>/test-first-workflow
/plugin install test-first-workflow@test-first-workflow
```

Or for local development, from a clone:

```
/plugin marketplace add /path/to/test-first-workflow
/plugin install test-first-workflow@test-first-workflow
```

## Usage

Ask Claude to start work on a ticket, for example:

> Start the feature workflow for PAY-142: customers can save a card for later.

Or run a single phase: "Review test coverage for this branch against PAY-142".

## Licence

MIT. See [LICENSE](LICENSE).
