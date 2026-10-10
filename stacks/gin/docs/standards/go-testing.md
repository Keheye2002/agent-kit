# Go Testing Standard

A behaviour is asserted once, at the layer that owns it. Before adding a
test, name the owner. Before keeping a test, ask: if this file disappeared,
would any bug become uncaught? When another layer already catches it, delete
this copy. When nothing catches it, move the assertion to the owner.

## Owners

| Layer | Owns |
| --- | --- |
| The package that decides the rule | Business rules, state transitions, validation |
| The package that talks to a database | SQL translation and constraint outcomes, against a real database |
| The HTTP handler | Routing, decoding, and status mapping, with the inner service fabricated |

Put the assertion in the owner's file. A status code belongs beside the
handler that writes it. A unique-constraint outcome belongs beside the code
that maps that constraint.

## Doubles

**Hard.**

- A double implements every interface method it claims. It does not embed
  the interface to satisfy a compile-time assertion: an unset embedded
  interface lets a missing method compile and panic on a `nil` call.
- A method the test did not stub fails the test (`t.Fatalf`, or an error the
  test cannot mistake for success) rather than returning a zero value that
  sends the code down a default path.
- A counter or a recorded call exists only when an assertion reads it. One
  fake per interface: a second test that needs a new behavior extends the
  existing fake rather than wrapping it in another type.
- A harness function that only forwards one call is pass-through; call the
  original. A test that needs the router builds it with the production
  constructor, not a second registration stack.

## Shape

**Hard.**

- A handler test calls the handler with an `httptest` request. It fabricates
  the service result. It does not open a database to prove a status code.
- A rule test calls the function that owns the rule. It does not boot Gin to
  prove a validation.
- A test name states the behaviour: `TestHealthz` reports liveness, not
  `TestHandler`.

**Judgement.** Two tests that fail on the same bug are one test too many.
Keep the one at the owning layer.
