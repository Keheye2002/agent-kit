# Coding Standards

How this service is written where no linter decides. Authors apply every rule
while writing; a review applies every rule to the diff. Each rule carries a
tag:

- **Hard**: a breach is a violation. Cite this file and the section.
- **Judgement**: report it as "possible …" with the reason, and let the author
  decide.

## Owned elsewhere

These documents own their topic. Apply them as written.

| Topic | Owner |
| --- | --- |
| Go tests | `docs/standards/go-testing.md` |

Formatting (`gofmt`) and every rule enabled in `.golangci.yml` belong to the
tools. `make fmt`, `make lint`, and `make test` run them.

## Placement and extraction

The leading word is **colocate**: code sits beside its only caller until a
second owner needs it.

**Hard.**

- A helper starts unexported, in the file or package of its first caller. It
  moves outward one step at a time (file, then package, then a shared
  package) when a caller in the next scope needs it. A symbol moves together
  with its tests.
- A package is named for what it provides. Shared code joins a package with
  that kind of name.
- One business rule (a validation, a state transition, a limit) has one
  source. That source appears when a second caller needs the decision.
- A repeated value (a literal, a lookup table, a key list) is one fact. The
  second file that needs the same value names it.
- A helper appears when a second caller needs the same decision. Until then
  the expression stays at the call site. A name that only restates the
  expression does not earn the helper.
- When a reusable helper already exists, new code calls it. A hand-written
  second version of the same rule breaks the one-source rule, even when it is
  shorter.

**Judgement.**

- A function whose body is as clear as its name stays at the call site even
  after a second caller, until the name adds a decision the expression does
  not show.
- A **pass-through** function, type, file, or test helper forwards to another
  without adding a decision. Inline it.
- Look-alike code that encodes different rules stays duplicated until the
  third occurrence that changes for the same reason. Two similar hunks are
  not yet a pattern.
- An abstraction that has grown a boolean or mode parameter, or a branch only
  one caller takes, is split back into each caller. Each caller keeps the
  part it uses.
- A function whose parameter list keeps growing splits into simpler functions
  first. An options struct fits when most callers set several options.

**Smell baseline override.** These rules replace the generic Duplicated Code
smell: flag duplication only per the rules above.

## Construction

The leading word is **assembled**: a value its constructor returned has every
dependency it will ever call.

**Hard.**

- A dependency a method calls is a constructor argument, and the constructor
  rejects it when it is missing. Request paths do not check for `nil`
  dependencies or answer "unavailable" for an unassembled one.
- A capability is a method on the interface the constructor requires. Code
  does not type-assert a dependency at runtime to discover an optional
  method: a fake that lacks the method skips the call and still compiles.
- A test seam stays out of the production API. An unexported constructor
  exported to tests through `export_test.go` serves the test; a public
  constructor widened to nilable interfaces leaks the seam into the contract.

**Judgement.** A `With*` method or a nilable field that switches a capability
on after construction is a runtime mode. Prefer passing the configuration to
the constructor, where an empty value means off, and decide once.

## Callers

The leading word is **caller**: every surface a change adds has a caller in
that change.

**Hard.**

- An exported symbol, interface method, option, config field, error code, or
  route with no caller outside its own test is deleted. It returns with its
  first caller.
- A branch that the code's own invariants make unreachable is deleted with
  the test that forced it. A guard repeated past a boundary that already
  guarantees the value is the same case; see Parsing at boundaries.

**Judgement.** A list, table, or registry with one entry is a singleton
dressed as a catalog. Write the single case directly and introduce the
catalog with the second entry.

**Smell baseline override.** These rules decide Speculative Generality.

## Failures

The leading word is **loud**: a miss, a parse failure, or missing
configuration reaches someone as an error.

**Hard.**

- A lookup that finds nothing fails or returns an explicit not-found. It does
  not fall back to the first item, a default row, or a zero value.
- A parse or validation failure is reported, not converted to "unset". A
  corrupt config file is a different outcome from a missing one.
- An unknown enum or mode value is an error at the point that reads it. The
  zero value of an `iota` enum or a `default:` branch does not stand in for a
  real member.
- A failure the caller can act on keeps its own reason. An infrastructure or
  configuration error is not reported as a user outcome (not found,
  permission denied, invalid input), and different failures do not share one
  code.

**Judgement.** A `default:` branch or a zero-value return that supplies a
value is a place to ask which invariant it hides. When the answer is "a value
that should always be present", remove the fallback and let the miss surface.

## Parsing at boundaries

The leading word is **parse**: a boundary turns loose input into a domain
type once, and code past the boundary trusts the type.

**Hard.**

- HTTP handlers and other edges parse wire input. A stored string is parsed
  where it is read. Past that point code passes the domain type, and the
  legal values live in that one parse function.
- A domain concept with a closed value set or an invariant gets a named type,
  its constants, and one parse function. A status does not travel as a free
  `string` that a later layer casts to the named type; the cast accepts any
  value.
- The value a boundary validates is the value it stores. When a field is
  trimmed or normalized for the check, the normalized value goes to the
  store.

**Judgement.** A check repeated past the boundary, on a value the type
already guarantees, means the parse is missing or leaks.

**Smell baseline override.** These rules decide Primitive Obsession: flag it
only for concepts with a closed value set or an invariant.

## Reads and writes

The leading word is **scope-bound**: every lookup carries the scope its
caller is allowed to see.

**Hard.**

- A lookup that is allowed to see only part of the data passes that scope
  into the query. A lookup by a bare identifier returns a row the caller may
  not own.
- Uniqueness is decided by a unique constraint. The insert is the check, so
  concurrent writers cannot both pass it. The empty result of a conflict
  insert becomes the already-exists outcome.
- A write a client may retry is idempotent, keyed by a natural or
  client-supplied key. A repeat of the same request returns the existing
  result. A different request on the same key is a conflict.

## Errors

The leading word is **handle once**: each error is returned, or logged, or
recovered from. Exactly one.

**Hard.**

- Code below the process edge returns errors. Logging happens at a boundary
  (see Logging).
- A wrap adds what the caller lacks: `fmt.Errorf("<verb> <noun>: %w", err)`,
  starting lowercase. The verb names the operation, so the message reads as
  a path of operations.
- Code branches on `errors.Is` and `errors.As` against sentinels and error
  types. `Error()` text is for humans.
- Names: exported sentinels `ErrXxx`, unexported `errXxx`, error types
  `XxxError`.
- The package that understands a failure translates it there. Callers above
  that package see the translated sentinel or wrapped error.
- A layer maps only the sentinels it recognizes. Any other error is wrapped
  and returned, so it reaches the HTTP edge as an internal error. A storage
  failure turned into a validation error, a permission error, or `false`
  hides an outage behind a user outcome.
- `panic` is reserved for programmer misuse caught at construction. Runtime
  and input failures return errors.
- `os.Exit` is called only in `cmd/*/main.go`.

**Judgement.** Use `%w` when a caller up the stack inspects the cause. Use
`%v` at a boundary where the cause's type should stop.

## Outbound calls

**Hard.**

- Every outbound call is bounded in time: the client sets a timeout, or the
  call runs under a `ctx` deadline. The call passes the caller's `ctx`.
- A client makes one attempt. Retry policy belongs to the caller, the only
  one that knows whether the operation is idempotent.
- A retry loop retries an idempotent operation, on an error that reports it
  can be retried, with capped exponential backoff and jitter.
- An outbound call runs outside a database transaction. A transaction holds
  row locks and a pooled connection, and rolling it back cannot undo a remote
  side effect. The order is: outbound work that can be compensated, then a
  short transaction that records it, then compensation when the transaction
  fails.
- The row that locates a remote object outlives the object. A delete or a
  state change that depends on the object checks the object first, and
  removes or advances the row only after the outbound step succeeded, or
  persists the intent so a retry can finish it.

## Logging

The leading word is **boundary**: the HTTP edge, job runners, and `cmd/` log.
Code inside the service returns errors and leaves logging to that edge.

**Hard.**

- A component that logs receives its logger from its constructor.
- The message is a fixed phrase. Varying data goes into structured fields
  with `snake_case` keys.
- An error field carries the error value.
- Fields hold identifiers. Passwords, tokens, session values, keys, and
  connection strings stay out of every field and message.
- Metric attributes come from closed sets: the method, the route template,
  the status code. Identifiers, raw paths, and user input go to traces and
  logs.

**Judgement.** Levels:

| Level | Meaning |
| --- | --- |
| Error | A person needs to act. |
| Warn | The process degraded and handled it itself. |
| Info | Lifecycle and request or job completion. |
| Debug | Diagnostic detail, off in deployed environments. |

## Time, context, and goroutines

**Hard.**

- A decision on the current time (expiry, freshness, revocation) reads an
  injected clock, so a test can set the time. Recorded timestamps come from
  the store.
- `context.Context` is the first parameter of anything that does I/O. The
  function uses the context it was given.
- A goroutine has an owner that stops it and waits for it.
- `init` stays free of I/O and global state.

## HTTP

Gin is the HTTP router.

**Hard.**

- Handlers parse the request, call into the service with typed values, and
  write the status. The service does not import Gin.
- `GET /healthz` is liveness and responds `204` when the process can serve.

**Judgement.** A metric or log that records a request uses the route
template, the pattern registered on the router, rather than the raw path.

## Records

The leading word is **same change**: a change that alters a behavior also
edits every record that states it.

**Hard.**

- Records are `README.md` command and output contracts, ADRs, the glossary,
  and the API contract, where the project has them. A diff that changes a
  command, a table, a route's security, or an error code edits the record in
  the same change.
- A superseded sentence is rewritten or deleted. A "superseded by" banner
  above a body that still states the old rule leaves two rules in force.
- A record describes the tree as it is. A file layout or route table that
  does not exist on disk is a second source of truth.

**Judgement.** A new document that restates rules another record already
owns wants deleting in favor of a pointer to the owner.
