# Question Bank

Probing questions per spec section. Adapt wording to the system; skip anything the facts already
answer; always attach a recommended answer. A question in a later group belongs to a later round
until the groups it depends on are settled.

## Identity and boundary (Sections 1, 2)

- In one sentence, what does the system do, and what kind of thing is it (daemon, CLI, library,
  service)? Who runs it and who consumes its output?
- What are the three or four operational problems it exists to solve? What is painful today?
- What is it explicitly **not** responsible for, and who or what owns that instead? (Writes to an
  external system? Business logic? UI? Persistence?)
- What does "success" mean for one unit of work? Is it a terminal state, or a handoff to a human
  or another system?
- What is the trust boundary: trusted operators only, untrusted inputs, both? Should the spec
  mandate one safety posture or let implementations document theirs?
- Which adjacent features are tempting but out of scope (multi-tenancy, dashboards, generic
  workflow engine, persistence)?
- Is there a target language, framework, or runtime, or must the spec stay implementable in any
  language? (Ask only when the user has not said; recommend binding when the spec is for an
  existing repository with an obvious stack.) If bound: which frameworks and libraries are
  already decided, and which are open for the interview to settle?

## Target stack (only when `Target:` is set)

- Which framework mechanisms does the system commit to (routing, middleware, dependency
  injection, ORM, job scheduler, test runner), and which does it deliberately avoid?
- What is the package or module layout, and which module owns each component from Section 3?
- Which standard-library or framework types carry the domain entities (structs, classes,
  schemas, ORM models), and which fields are typed differently from their wire form?
- What are the build, run, and test commands the definition of done will point at?
- Which of the framework's conventions (config loading, env handling, logging) replace a rule the
  spec would otherwise have to write, and which rules still need stating because the framework
  leaves them open?
- Is there a minimum language or framework version, and what feature requires it?

## Components and layers (Section 3)

- What are the main components, and which single component owns mutable state?
- Which components are OPTIONAL? Can the system be correct without them?
- What layers would a port to another language need to preserve? What is policy (owned by users)
  vs. mechanism (owned by the implementation)?
- What must exist on the host: external APIs, executables, filesystem, credentials? Which of those
  MUST NOT be visible to child processes?

## Domain model (Section 4)

- What is the unit of work, and what is its stable identity? Is that identity the external
  system's ID, or something the adapter chooses? Can the core treat it as opaque?
- Which fields are REQUIRED for the core to act at all? Which are nullable, and what does null
  mean?
- Is there a human-readable identifier distinct from the ID? Where is it used (logs, paths,
  routes) and what uniqueness does that use require?
- How do external states map into the core's view: raw strings compared case-insensitively, or
  an internal enum?
- Which eligibility rules can only the adapter know (assignment, board membership, blockers)?
  Should they collapse into one explicit boolean rather than be reconstructed by the core?
- What per-run and per-session metadata must be tracked for observability (IDs, timestamps,
  counters)?
- How is any name derived from user data sanitized for use in paths or routes? What guarantees
  collision resistance?

## Primary input contract (Section 5)

- What does the user author to drive the system (file, schema, request)? Where is it discovered,
  and what is the precedence when several sources exist?
- What is the file format, and what exactly happens for each malformed variant (missing, bad
  syntax, wrong root type)?
- What are the top-level keys, and what happens to unknown ones?
- For each field: type, default, validation, and whether a change applies live or needs restart?
- Is there a template or DSL inside it? Strict or lenient on unknown variables and filters? What
  variables are available?
- Which errors block all new work, and which fail only the affected unit?

## Configuration (Section 6)

- In what order are values resolved (locate, parse, defaults, environment indirection, coerce)?
  Do environment variables ever override values that are present in the file?
- How are secrets referenced? What does an empty resolved secret mean?
- Which values are paths and get `~` or `$VAR` expansion, and which are commands or URIs that
  must be left alone? Relative to what are relative paths resolved?
- Must changes be detected and re-applied without restart? Which settings, and what about
  in-flight work? What happens on an invalid reload?
- What is validated at startup vs. before every cycle? What is the effect of each failure?

## State machine (Section 7)

- What are the internal claim states of one unit, distinct from external states?
- What are the lifecycle phases of one attempt, and which terminal reasons need to be
  distinguished because retries or logs differ?
- What triggers transitions (tick, exit, timer, external refresh, event, timeout), and what does
  each mutate?
- Does a normal exit mean "done", or does the system re-check and possibly continue? On the same
  session or a new one? Up to what limit?
- What single authority mutates state, and what pre-checks are REQUIRED before starting work?
- What survives a restart, and what is explicitly not restored?

## Core loop (Section 8)

- What is the exact tick sequence? Does reconciliation run before dispatch?
- What is the full eligibility predicate? Which parts are the adapter's and which the core's?
- How are candidates ordered, and what are the tie-breakers? How do null values sort?
- What are the global and per-bucket concurrency limits, as formulas?
- What is the retry formula, base delay, and cap? Are continuation retries different from failure
  retries?
- On a retry timer, what is re-checked, and what happens if the unit is gone, terminal, still
  eligible but no slots, or eligible?
- What does reconciliation do for each observed state (terminal, active, active but not routable,
  neither, missing, refresh failed)? Which of those clean up resources?
- Is stall detection needed? Based on what timestamps, and how is it disabled?
- What is swept at startup?

## Resources and safety (Section 9)

- Where do per-unit resources live, how are they named, and are they reused or recreated?
- What is the creation algorithm, and what distinguishes "created now" from "reused"?
- What lifecycle hooks exist, when do they run, what is their timeout, and is each failure fatal
  or logged-and-ignored?
- What are the safety invariants, and what concrete check enforces each before any child
  process launches?
- Is resource population (checkout, bootstrap) built in or left to hooks? What happens on failure
  for a new vs. reused resource?

## External protocol (Section 10)

- Which external protocol or executable is the source of truth? Which version? Will the spec defer
  to it on conflict and refuse to restate its schema?
- What is the launch contract: command, shell invocation, cwd, transport, buffering limits?
- What MUST the client do at startup (initialize, create or resume session, supply cwd, supply
  policy, advertise tools, include titles or metadata)?
- What signals end a turn or unit, and which count as success vs. failure? What about silence and
  subprocess exit?
- How are continuation turns sent: same live session, and with what content?
- What events flow upstream, with what envelope fields?
- Approvals, user-input requests, unsupported tool calls: what is the documented policy, and what
  invariant holds regardless ("never stall indefinitely")?
- What timeouts exist and what does each measure (silence vs. total)? What normalized error
  categories are RECOMMENDED?

## Adapter contract (Section 11)

- What is the minimum set of read operations the core needs? Can any write operation be removed
  from the core and pushed into agent-facing tools?
- For each operation: behavior on empty input, on omitted records, on malformed records, on
  partial paging failure?
- What must every adapter own (auth, scope, pagination, rate limits, normalization,
  dispatchability) and what must the core never inspect?
- What does an adapter have to publish as a documented profile?
- What are the RECOMMENDED error categories, and what does the core do at each failure site?
- If adapters expose tools to an agent: how are tool specs bound to a session snapshot across
  reloads, where do they execute, and how are secrets kept out of the child environment?

## Derived artifacts (Section 12)

- What is rendered from what inputs? Strict or lenient? What extra variables (attempt, retry
  kind) exist?
- What happens on render failure: fail the attempt, or fall back?

## Observability (Section 13)

- Which context fields are REQUIRED on every log line about a unit or session?
- Where can logs go, and what MUST an operator be able to see without a debugger? What happens if
  a sink fails?
- Is there a synchronous snapshot for dashboards? What fields and error modes?
- How are metrics accumulated without double counting (absolute totals vs. deltas)? What is live
  vs. cumulative?
- Is an HTTP or API surface an OPTIONAL extension? Which endpoints, bind host, port config, CLI
  override, error envelope, and is a hot-rebind required?

## Failure model (Section 14)

- What are the failure classes, with examples, and the recovery for each?
- Which failures skip work but keep the service alive? Which fail startup?
- After a restart, how does the system get back to useful operation without a database?
- What can an operator change at runtime, and what requires a restart?

## Security (Section 15)

- What trust assumptions must an implementation state?
- Which filesystem or data rules are mandatory, and which are RECOMMENDED hardening?
- How are secrets referenced, validated, kept out of logs, and kept away from children?
- Are user-authored scripts fully trusted? What are the implications and required limits?
- Which inputs are externally controllable (tracker text, repository contents, tool arguments), and
  what hardening measures should implementations consider?

## Tests and definition of done (Sections 17, 18)

- Which behaviors are `Core Conformance` and must be deterministic? Which are `Extension
  Conformance`? Which need real credentials and must report as skipped when skipped?
- Is there a normative statement in the body with no matching test bullet? Add one or cut the
  statement.
- Which capabilities are REQUIRED to ship, which are RECOMMENDED, and which are `TODO`?
- What must be validated on the target host before production?
