# Test data

Good test data makes a test's intent obvious and catches the bugs that live at
the edges. Bad test data hides intent, passes by coincidence, or ties tests
together.

## Builders with realistic defaults

Use a builder or factory per significant data type. Defaults should be a
valid, realistic record, so a test only states what is special about it:

```ts
const card = aCard().expired().build();           // intent is obvious
const customer = aCustomer().withNoSavedCards().build();
```

Not:

```ts
const card = { number: "4111111111111111", expiry: "01/20", cvc: "123",
               holder: "x", billingPostcode: "", ... };  // which field matters?
```

Rules:
- Defaults pass validation and look like production data (real formats,
  plausible lengths, correct types).
- Overrides are named for meaning (`expired()`, `withBalance(0)`), not just
  field setters, where it helps readability.
- Keep builders in test utilities, never in production code.
- Use the same builders for mock responses and for integration tests, so the
  shapes stay in sync.

## Choosing values on purpose

For each input that drives behaviour, consider:

| Category | Examples |
|---|---|
| Typical | A normal valid value |
| Boundaries | min, max, min-1, max+1, exactly-at-limit, zero |
| Empty / missing | `""`, whitespace, `null`/`undefined`, empty list, missing field |
| Format | wrong type, wrong format, too long, unicode, leading/trailing spaces |
| Business edge cases | expired, duplicate, already processed, other user's record |

Let the risk ranking in `test-plan.md` decide how far to go: high-risk inputs
get boundaries and invalid cases; low-risk ones get the typical case.

Avoid "magic" values that only work by coincidence (e.g. an amount that
happens to equal a threshold). If a value matters, name it:
`const AMOUNT_AT_DAILY_LIMIT = 1000`.

## Data at the contract

Data crossing a mocked boundary must match the real contract exactly:
field names, casing, types, nullability, date and money formats, enum values.
A mock returning `{ status: "ok" }` when the real service returns
`{ state: "OK" }` produces a test that passes and a bug that ships.

## Independence

- Each test creates the data it needs. No shared mutable fixtures.
- Tests pass in any order and in parallel.
- Use fixed, injected clocks and ID generators rather than real time or
  randomness. If you use random/generated data, log the seed so failures can
  be reproduced.
- In integration tests, create data through the system's own API or
  repository where practical, and clean it up (or isolate it per test).
