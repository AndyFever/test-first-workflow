---
name: isolated-tdd
description: Use when writing or changing production code - new behaviour, bug fixes, refactoring. Red-green-refactor on small units tested in isolation, with collaborators mocked behind explicit contracts and deliberate, realistic test data. Inside feature-workflow, use it in the build phase after planning is approved.
---

# Isolated-Unit TDD

Build code one small behaviour at a time, test first. Each unit test covers a
small part of the application **in isolation**: the unit's collaborators are
mocked, and every mock stands for a clear contract. That keeps failures
pointed at one place and makes the data flowing between components explicit.

If you are inside `feature-workflow`, read `plan.md` (components, contracts)
and `test-plan.md` (acceptance tests, risk) first. They tell you what to build,
what to mock, and where the risky data is.

## The cycle

### 1. Red: write one failing test

- Pick the next smallest behaviour of the unit.
- Before writing it, say in one line **what change to the production code
  would make this test fail**. If you can't, the test isn't testing anything.
- Write the test: one behaviour, a name that states it
  (`rejects_expired_card`, not `test_validate_2`), mocked collaborators set
  up from their contracts, test data built on purpose (see below).
- **Run it and watch it fail.** It must fail on the assertion, for the reason
  you predicted. An error (import, typo, bad setup) is not a red; fix it and
  re-run. If it passes straight away, the behaviour already exists or the test
  is wrong. Find out which before going on.

### 2. Green: make it pass simply

- Write the least production code that makes the test pass.
- No extra options, branches or features the test doesn't ask for.
- Run the test, then the unit's whole test file. All green.

### 3. Refactor: clean up while green

- Remove duplication, improve names, simplify, in both code and tests.
- Extract test-data builders and mock helpers when setup repeats.
- Run the tests after each change. No new behaviour in this step.

Then pick the next behaviour and go again.

## Isolation and mocks

Read [references/mocks-and-contracts.md](references/mocks-and-contracts.md)
before writing tests that use mocks. The short version:

- **Mock collaborators, not values.** Mock the components your unit talks to
  (repositories, gateways, other services, the clock). Use real value objects,
  DTOs and pure functions.
- **Mock from the contract.** The mock's inputs, outputs and errors must match
  the contract in `plan.md` (or the real interface). Include the failure
  responses, not only the happy one.
- **Don't mock what you don't own.** Wrap third-party libraries and SDKs in a
  thin adapter of your own; mock the adapter.
- **Assert on outcomes first.** Check what the unit returns, raises or
  changes. Verify a call to a collaborator only when making that call *is*
  the behaviour (sends the email, publishes the event, charges the card).
- **Every mocked contract needs a real check.** Note it in `test-plan.md`'s
  contract coverage so an integration or contract test confirms the real
  collaborator behaves the way your mock does.

## Test data

Read [references/test-data.md](references/test-data.md) when building test
data. The short version:

- Build data with builders/factories that produce **valid, realistic**
  defaults. Each test overrides only the fields that matter to it, so the
  reader can see what the test is about.
- Pick **boundary and invalid values on purpose**, guided by the risk ranking.
- Data passed to and returned from mocks must have the **same shape as the
  real contract**: field names, types, nullability, formats.
- Each test sets up its own data. No shared mutable fixtures, no reliance on
  test order.

## Verify before calling it done

- Every new or changed behaviour has a test you watched fail first.
- The unit's tests pass, **and the project's full test suite passes**. Run the
  project's normal test command, not just your file. Report any failure by
  test name, including ones you didn't cause.
- Test output is clean: no unexpected errors, warnings or noisy logs.
- Any new or changed collaborator contract is recorded for integration/contract
  coverage.

## Bugs

A bug fix starts with a failing test that reproduces the bug, at the lowest
level that can show it. Then fix it with the same cycle.

## When it gets hard

| Symptom | Likely cause | Try |
|---|---|---|
| Many mocks per test | Unit has too many collaborators | Split the unit, or introduce a role that groups them |
| Huge data setup | Missing builders, or unit needs too much data | Add a builder; pass the unit only what it uses |
| Test breaks on harmless refactors | Asserting on interactions instead of outcomes | Assert on results; verify calls only when they are the behaviour |
| Can't say what the mock should return | Contract unclear | Stop and agree the contract (update `plan.md`) |
| Tempted to write the code first | — | Write the test you wish you had. Ask the engineer if the behaviour is unclear |

Exceptions to test-first (throwaway spikes, generated code, configuration)
are agreed with the engineer, not assumed.
