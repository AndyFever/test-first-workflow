---
name: coverage-review
description: Use when checking whether a ticket's changes have the right test coverage, before calling the work done or opening a PR. Maps every acceptance criterion, planned test and mocked contract to real tests in the diff, flags gaps and weak tests, and runs the full suite. Functional coverage only (excludes non-functional testing). Produces docs/work/<ticket-id>/coverage-review.md.
---

# Coverage Review

Check that the tests actually prove the ticket, not just that tests exist.
This is about functional behaviour; performance, load, security and
accessibility testing are out of scope.

## Inputs

- `docs/work/<ticket-id>/plan.md` and `test-plan.md`, if they exist.
- Otherwise the ticket's acceptance criteria (ask the engineer for them).
- The diff for this work (e.g. against the base branch) and the tests it adds
  or changes.

Line-coverage numbers can be supporting evidence but are not the measure.
A line can be executed by a test that asserts nothing.

## Checks

Work through each. Read the tests; don't judge by names alone.

1. **Acceptance criteria → tests.** Every AC (and agreed implied AC) has at
   least one test that would fail if the behaviour broke. Note the level.
2. **Planned vs. written.** Every test in `test-plan.md` exists, at the
   planned level. Missing or moved tests are listed with the reason, if known.
3. **Risk depth.** High-risk ACs have failure paths and boundary data, not
   only the happy path.
4. **Mocked contracts confirmed.** Every collaborator mocked in unit tests has
   an integration or contract test exercising the real thing. Check the mock's
   data shape matches what the real test sees.
5. **Unit coverage of new code.** New or changed production behaviour (branches,
   error handling, validation) has unit tests. Look for untested branches in
   the diff.
6. **Test quality.** Flag tests that:
   - assert on a mock's behaviour instead of the unit's outcome
   - verify interactions for queries, or exact call order/counts that don't matter
   - mock values, pure functions, or third-party code directly
   - use unrealistic data, or data whose shape differs from the real contract
   - share mutable fixtures or depend on test order
   - have names that don't describe the behaviour
   - can't fail (no assertion, or assert something always true)
7. **Bug fixes.** Each fixed bug has a regression test.
8. **Full suite.** Run the project's full test command. Record the result and
   name every failing or skipped test, including ones this work didn't cause.

Where you're unsure whether a test can fail, say how to check (e.g. "break
X and confirm the test fails") rather than guessing.

## Output: `docs/work/<ticket-id>/coverage-review.md`

```markdown
# Coverage review: <ticket-id>
Date: <date> · Full suite: <passed/failed> (<counts>)

## Acceptance criteria
| AC | Covered by | Level | Status |
|---|---|---|---|
| AC1 | test_saves_card_and_returns_token | integration | ✅ |
| AC2 | — | — | ❌ no test |

## Mocked contracts
| Contract | Unit tests mock it | Confirmed by | Status |
|---|---|---|---|

## Gaps (must fix)
1. <what is missing, where, which test to add at which level>

## Weak tests (should fix)
1. <test name> — <problem> — <suggested fix>

## Notes
<skipped phases, agreed exceptions, failures not caused by this work>
```

Then summarise the gaps for the engineer. Fix gaps only with their agreement,
using the test-first cycle, and re-run the review afterwards.
