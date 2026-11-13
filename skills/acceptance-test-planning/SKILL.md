---
name: acceptance-test-planning
description: Use when planning acceptance tests for a ticket, after the work is planned and before code is written. Decides whether acceptance tests are needed, ranks acceptance criteria by risk, and places each test at the lowest level that can prove it (integration or end-to-end), following shift-left principles. Produces docs/work/<ticket-id>/test-plan.md.
---

# Acceptance Test Planning

Decide which behaviours need acceptance tests, how much testing each deserves,
and at which level. Read `docs/work/<ticket-id>/plan.md` first; if it doesn't
exist, get the acceptance criteria from the ticket and the enginneer.

Scope: functional behaviour only. Performance, load, security and
accessibility testing are out of scope for this skill.

## 1. Are acceptance tests needed?

Acceptance tests are usually needed when the ticket changes behaviour a user,
client or another system can observe. They may not be needed for:

- Pure internal refactoring with no behaviour change (existing tests cover it)
- Changes fully proven by unit tests inside one component with no boundary

Decide, record the reason, and confirm with the enginneer if the answer is "no".

## 2. Rank risk

For each acceptance criterion (including agreed implied ones), score:

- **Likelihood** it breaks (1-3): new code, complex logic, many collaborators,
  unclear contracts, data transformations, past bugs in this area → higher.
- **Impact** if it breaks (1-3): money, data loss or corruption, security,
  many users, hard to detect, hard to roll back → higher.

**Risk = likelihood × impact** (1-9). Use it to decide depth:

| Risk | Depth |
|---|---|
| 6-9 | Happy path + key failure paths + boundary data at the right level; consider one e2e journey |
| 3-4 | Happy path + main failure path at integration level |
| 1-2 | Covered by unit tests; acceptance test only if cheap |

## 3. Choose the level (shift left)

Place each test at the **lowest level that can prove the behaviour**. Lower
levels are faster, more stable and point straight at the cause.

| Level | Use it to prove | Real vs. replaced |
|---|---|---|
| **Integration** | The behaviour through a real boundary: API endpoint, database, queue, service-to-service call. Also confirms the contracts that unit tests mock. | Real component(s) under test and their own infrastructure (e.g. a containerised DB); external third parties stubbed at the network edge |
| **Contract** | Two services agree on a request/response shape, without running both together | Consumer and provider tested separately against a shared contract |
| **End-to-end** | A critical user journey works across the whole system | Everything real, as close to production as practical |

Rules of thumb:

- If an integration test can prove it, don't write an e2e test for it.
- Keep e2e for the few highest-risk journeys. Each one should cover a whole
  flow a user cares about, not a single field rule.
- Validation rules, calculations and branching logic belong in unit tests;
  acceptance tests check they are wired in, not every case.
- Every collaborator contract listed in `plan.md` needs at least one
  integration or contract test, so unit-test mocks can't drift from reality.

## 4. Test data

For each test, say what data it needs and why: realistic valid records,
boundary values, invalid inputs. Note who creates it (test sets up its own
data via builders/factories/API) and how it is cleaned up. Avoid depending on
shared, pre-existing environment data.

## 5. Review with the enginneer

Show the risk ranking and the test table. Ask specifically about:
any criterion ranked high risk, any e2e test (is it worth its cost?), and any
"not needed" decisions. Revise until approved.

## Output: `docs/work/<ticket-id>/test-plan.md`

```markdown
# Test plan: <ticket-id>
Approved: <date> by enginneer (<notes>)
Acceptance tests needed: yes | no — <reason>

## Risk ranking
| AC | Summary | L | I | Risk | Why |
|---|---|---|---|---|---|

## Acceptance tests
| ID | AC | Level | Test name (behaviour) | Data | Component(s) |
|---|---|---|---|---|---|
| AT1 | AC1 | integration | saves card and returns token for valid card | valid Visa test card, new customer | CardService |

## Contract coverage
| Contract (from plan.md) | Confirmed by |
|---|---|
| PaymentGateway.tokenise | AT1 |

## Deliberately not tested at acceptance level
- <what> — <why, and where it is covered instead>
```
