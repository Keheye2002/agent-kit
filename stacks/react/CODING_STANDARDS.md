# Coding Standards

How this app is written where no linter decides. Authors apply every rule
while writing; a review applies every rule to the diff. Each rule carries a
tag:

- **Hard**: a breach is a violation. Cite this file and the section.
- **Judgement**: report it as "possible …" with the reason, and let the author
  decide.

## Owned elsewhere

These documents own their topic. Apply them as written.

| Topic | Owner |
| --- | --- |
| Web tests | `docs/standards/web-testing.md` |

Formatting (oxfmt) and every rule enabled in `.oxlintrc.json` belong to the
tools. `pnpm format`, `pnpm lint`, and `pnpm test` run them.

## Placement and extraction

The leading word is **colocate**: code sits beside its only caller until a
second owner needs it.

**Hard.**

- A helper starts unexported, in the file of its first caller. It moves
  outward one step at a time (file, then a shared module) when a caller in
  the next scope needs it. A symbol moves together with its tests.
- A module is named for what it provides. Shared code joins a module with
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
  first. An options object fits when most callers set several options.

**Smell baseline override.** These rules replace the generic Duplicated Code
smell: flag duplication only per the rules above.

## Callers

The leading word is **caller**: every surface a change adds has a caller in
that change.

**Hard.**

- An exported symbol, prop, option, or route with no caller outside its own
  test is deleted. It returns with its first caller.
- A branch that the code's own invariants make unreachable is deleted with
  the test that forced it.

**Judgement.** A list, table, or registry with one entry, or a union with one
member, is a singleton dressed as a catalog. Write the single case directly
and introduce the catalog with the second entry.

**Smell baseline override.** These rules decide Speculative Generality.

## Failures

The leading word is **loud**: a miss, a parse failure, or missing
configuration reaches someone as an error. App code and mock code alike.

**Hard.**

- A lookup that finds nothing fails or returns an explicit not-found. It does
  not fall back to the first item, a default row, or an empty object
  (`?? items[0]`, `{ name: "Untitled" }`).
- A parse or validation failure is reported, not converted to "unset".
  `.catch(undefined)` or `catch { return null }` erases the difference
  between missing and corrupt.
- An unknown enum or mode value is an error at the point that reads it. A
  `default:` branch does not stand in for a real member.
- A failure the user can act on keeps its own reason. A network or
  configuration error is not shown as a user outcome (cancelled, permission
  denied, invalid input).

**Judgement.** A `??`, `||`, `catch`, or `default` that supplies a value is a
place to ask which invariant it hides. When the answer is "a value that
should always be present", remove the fallback and let the miss surface.

## Effects

The leading word is **external system**, as React's
[You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
uses it.

**Judgement.**

- `useEffect` synchronizes a component with an external system: a media
  element, a window listener, an observer.
- A value computable from props, state, or query data is computed during
  render. Server data stays in the TanStack Query cache.
- Component state that points at server data holds the ID, not a copy of
  the row. The row is derived from the latest query result on each render,
  so a mutation's refetch updates the open dialog, and a row that vanished
  closes it. A selection that changes a server query is URL search state.
- Work a user event causes (open, click, submit) runs in that event's
  handler.

## Routing

TanStack Router owns the URL. TanStack Query owns server data.

**Hard.**

- File routes live under `src/routes/`. The Vite plugin is
  `@tanstack/router-plugin/vite`, registered before the React plugin, with
  `autoCodeSplitting: true`.
- `src/routeTree.gen.ts` is generated and committed. Hand edits to that file
  are discarded. Lint and format ignore it.
- A route file exports `Route` only. A helper that exists for that route
  lives in a sibling file or directory whose name starts with `-`, so the
  generator skips it.
- Path segments are lowercase English `kebab-case`. A collection segment is
  a plural noun. A parameter name says which thing it is (`sceneId`), and is
  camelCase in TypeScript.
- Web URLs stay independent of the API version.
- Each resource has one canonical URL. The path carries a stable id. A
  display name stays out of the path.

**Judgement.** A loader reads what the route needs in order to render. A
mutation that a click causes runs in that click's handler, then invalidates
the query.

## Names and comments

**Hard.**

- When the project has a glossary, identifiers, file names, error messages,
  and log messages use its terms.
- An exported identifier reads well after its module name.

**Judgement.**

- A comment states what the code cannot show: a constraint, a gotcha, or the
  reason for an odd choice. A comment that restates the next line is noise.
- Ticket identifiers belong in the commit, so a comment stays true after the
  review closes.

## Records

The leading word is **same change**: a change that alters a behavior also
edits every record that states it.

**Hard.**

- Records are `README.md`, ADRs, and the glossary, where the project has
  them. A diff that changes a documented route, command, or behavior edits
  the record in the same change.
- A superseded sentence is rewritten or deleted. A "superseded by" banner
  above a body that still states the old rule leaves two rules in force.
- A record describes the tree as it is. A file layout or route table that
  does not exist on disk is a second source of truth.

**Judgement.** A new document that restates rules another record already
owns wants deleting in favor of a pointer to the owner.
