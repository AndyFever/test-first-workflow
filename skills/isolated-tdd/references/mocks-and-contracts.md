# Mocks and contracts

Unit tests isolate the unit by replacing its collaborators. Done well, this
gives fast, precise tests and makes every boundary explicit. Done badly, the
tests pass while production breaks, or they break whenever code is tidied.

## Pick the right test double

| Double | What it does | Use when |
|---|---|---|
| **Stub** | Returns canned answers | The unit needs data from a collaborator (a query) |
| **Mock** | Records calls so you can verify them | The call itself is the behaviour (a command: send, publish, charge) |
| **Fake** | A working, simplified implementation (in-memory repository) | Many tests need realistic collaborator behaviour; a stub per test would be noisy |
| **Spy** | Real object that also records calls | Rarely; prefer the above |

Queries get stubs and outcome assertions. Commands get mocks and interaction
assertions. Most tests need only stubs.

## What to replace and what to keep real

**Replace (collaborators / roles):**
- Repositories, data-access objects
- Clients for other services, gateways, message publishers
- Your adapters around third-party SDKs
- Clock, random/ID generators, file system, environment

**Keep real:**
- Value objects, DTOs, domain entities with no I/O
- Pure functions and calculations
- The unit's own private helpers (they're part of the unit)

If you're about to mock something that just holds data, use a real instance
built by a test-data builder instead.

## Write the contract down

For each collaborator, the contract is:

```
<Collaborator>.<operation>(<input shape>)
  → <success output shape>
  ! <error/failure cases and how they are signalled>
```

Example:

```
PaymentGateway.tokenise({ cardNumber: string(16), expiry: "MM/YY", cvc: string(3) })
  → { token: string, last4: string(4), brand: "visa" | "mastercard" }
  ! CardDeclined { reason }   ! GatewayUnavailable
```

Set mocks up from this, including the failure cases. A unit test suite where
collaborators only ever succeed is missing half the behaviour.

## Keep mocks honest

Mocks can drift: the real collaborator changes, the mock doesn't, and unit
tests keep passing. Defences, strongest first:

1. **Contract or integration test per mocked contract.** Recorded in
   `test-plan.md` → *Contract coverage*. `coverage-review` checks it.
2. **Typed mocks.** Generate mocks from the real interface/type so the
   compiler rejects wrong method names and shapes.
3. **Shared response builders.** Build mock responses with the same builders
   the integration tests use, or from recorded real responses/schemas.

## Don't lock in the implementation

- Verify interactions only for commands that are the behaviour. Don't verify
  that a query was called; if the stubbed data affected the result, the
  outcome assertion already proves it.
- Don't assert on call order or exact call counts unless they matter to the
  behaviour.
- Match arguments on the fields that matter, not entire objects, unless the
  whole object is the point.
- If refactoring without changing behaviour breaks a test, the test was
  asserting implementation. Fix the test's assertion, not by adding more mocks.

## Don't mock what you don't own

Third-party SDKs have complex, changing behaviour you can't fully reproduce in
a mock. Wrap them in a thin adapter with an interface in your own terms. Unit
tests mock the adapter; the adapter itself gets an integration test against
the real (or sandboxed) third party.
