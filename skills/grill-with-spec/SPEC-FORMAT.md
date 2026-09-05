# SPEC.md Format

The spec is one Markdown file. Its shape is the one in [EXAMPLE-SPEC.md](EXAMPLE-SPEC.md),
generalized. Section numbers below are the canonical order; rename a section to fit the system,
merge two when one would be near-empty, but keep the order of concerns and keep the numbering
continuous. A REQUIRED section that truly does not apply stays in the file as a one-line
`Not applicable: <reason>` so a reader knows it was considered.

## Skeleton

```markdown
# <System> Specification

Status: Draft v1 (language-agnostic, agent-agnostic)
Target: <language>, <framework> (only when the user named a stack; then drop "language-agnostic")

Purpose: <one sentence: what the system does and for whom>.

## Normative Language
## 1. Problem Statement
## 2. Goals and Non-Goals
## 3. System Overview
## 4. Core Domain Model
## 5. <Primary Input Contract>
## 6. Configuration Specification
## 7. <Core> State Machine
## 8. <Core Loop, Scheduling, and Reconciliation>
## 9. <Resource> Management and Safety
## 10. <External Protocol> Integration
## 11. <Adapter> Integration Contract
## 12. <Derived Artifact> Construction
## 13. Logging, Status, and Observability
## 14. Failure Model and Recovery Strategy
## 15. Security and Operational Safety
## 16. Reference Algorithms (Language-Agnostic, or Reference Implementation Sketches)
## 17. Test and Validation Matrix
## 18. Implementation Checklist (Definition of Done)
## Appendix A. <Extension> (OPTIONAL)
## Open Questions
```

## Section guidance

### Header (REQUIRED)

- `Status: Draft vN (language-agnostic, agent-agnostic)`. Bump `N` whenever normative content
  changes. When the spec binds to a stack, write `Status: Draft vN (agent-agnostic)` instead.
- `Target:` present only when the user named a language, framework, or runtime. List exactly what
  the user named (`Target: TypeScript, Next.js App Router`). A minimum language or framework
  version MAY appear here and nowhere else.
- `Purpose:` one sentence.

### Normative Language (REQUIRED, boilerplate)

Use this text verbatim:

```markdown
The key words `MUST`, `MUST NOT`, `REQUIRED`, `SHOULD`, `SHOULD NOT`, `RECOMMENDED`, `MAY`, and
`OPTIONAL` in this document are to be interpreted as described in RFC 2119.

`Implementation-defined` means the behavior is part of the implementation contract, but this
specification does not prescribe one universal policy. Implementations MUST document the selected
behavior.
```

### 1. Problem Statement (REQUIRED)

- One paragraph: what the system is, as a running thing (daemon, library, CLI, service).
- "The system solves N operational problems:" as a short bullet list.
- A paragraph on trust and safety posture if the system executes anything.
- `Important boundary:` bullets stating what the system is **not** responsible for and who is.
  This is where scope creep is killed; be explicit.

### 2. Goals and Non-Goals (REQUIRED)

Two bullet lists. Non-goals name the tempting adjacent features and where that logic lives
instead ("That logic lives in the workflow prompt and agent tooling.").

### 3. System Overview (REQUIRED)

- `3.1 Main Components`: numbered list; each component gets a backticked name and two to four
  responsibility bullets. Mark OPTIONAL components.
- `3.2 Abstraction Levels`: the layers a port should preserve (policy, configuration,
  coordination, execution, integration, observability, or whatever fits).
- `3.3 External Dependencies`: what must exist on the host, including auth and what MUST NOT leak
  to child processes.

### 4. Core Domain Model (REQUIRED)

- `4.1 Entities`: one `#### 4.1.N Name` per entity. Each field is a bullet:
  `- \`field\` (type or null)` followed by indented bullets for REQUIRED/OPTIONAL, semantics,
  normalization, and what the core MUST NOT assume about it. Cover the internal runtime state as
  an entity too.
- `4.2 Stable Identifiers and Normalization Rules`: for each identifier, what it is used for, what
  it MUST NOT be used for, and the exact derivation (sanitization regexes, hash suffix entropy,
  case folding, composition format).

### 5. Primary Input Contract (REQUIRED when users author a file, schema, or API)

The thing users write that drives the system (a workflow file, config schema, manifest, request).

- Discovery and path precedence, numbered.
- File format and parsing rules, including the error each violation raises.
- Field-by-field schema grouped by top-level key: type, default, validation, whether changes
  apply at runtime or need restart. Say what happens to unknown keys.
- Template or DSL rules if any (strictness, available variables, fallback behavior).
- A named error-class list and the dispatch/gating effect of each class.

### 6. Configuration Specification (REQUIRED)

- Resolution pipeline as a numbered list (locate, parse, defaults, env indirection, coerce and
  validate). State explicitly that environment variables do not globally override values.
- Dynamic reload semantics: what MUST be re-read, what MAY require restart, that invalid reloads
  MUST NOT crash and keep last known good.
- Preflight validation: startup behavior and per-cycle behavior, with the exact checks.
- `Core Config Fields Summary (Cheat Sheet)`: one line per field with type and default. Say
  "This section is intentionally redundant so a coding agent can implement the config layer
  quickly."

### 7. State Machine (REQUIRED for long-running or multi-step systems)

- Internal states, numbered, each with one line. Distinguish them from any external state names.
- Lifecycle phases of one unit of work, numbered, including distinct terminal reasons.
- Transition triggers: each trigger as a bullet with the mutations it causes.
- Idempotency and recovery rules: single mutation authority, required pre-checks, what restart
  restores and what it does not.

### 8. Core Loop (REQUIRED)

- Startup sequence and repeating cycle as a numbered tick sequence.
- Eligibility rules: "X is eligible only if all are true:" bullet list. Define any predicate you
  reuse later by name (for example `issue_routable(issue)`).
- Ordering rules with tie-breakers.
- Concurrency formulas written as expressions.
- Retry and backoff: the exact formula, the cap, and the handling algorithm as a numbered list.
- Reconciliation: parts A/B, each as bullets with the action for every observed condition,
  including "refresh failed".
- Startup sweep or recovery pass.

### 9. Resource Management and Safety (REQUIRED)

- Layout (paths, naming, persistence policy).
- Creation and reuse algorithm, numbered.
- Hooks or extension points: trigger, timeout, and failure semantics for each.
- `Safety Invariants`: numbered `Invariant N:` statements with the concrete check that enforces
  each. Flag the most important portability constraint.

### 10. External Protocol Integration (as needed)

- Name the external protocol as source of truth; the spec MUST NOT restate its schema and MUST
  defer to it on conflict, while retaining authority over orchestration behavior.
- Launch contract (command, invocation, cwd, transport).
- Session startup responsibilities as a MUST list.
- Streaming and completion conditions (`signal -> outcome` bullets).
- Emitted upstream events: required envelope fields plus an example event list.
- Policy areas that are `Implementation-defined` (approvals, sandboxing, user input), each with
  the invariant that still holds ("MUST NOT stall indefinitely") and an example posture.
- Timeouts and a RECOMMENDED normalized error category list.

### 11. Adapter / Integration Contract (as needed)

- REQUIRED operations as numbered signatures in `text` form with contract bullets (empty input
  behavior, omission semantics, atomicity, malformed-record rules).
- Adapter responsibilities vs. things the core MUST NOT do.
- What each adapter MUST publish as a documented profile.
- Normalization rules per field.
- RECOMMENDED error categories and the core's behavior on each failure site.
- The boundary: which operations deliberately do not exist in the core and why.

### 12. Derived Artifact Construction (as needed)

Prompts, rendered configs, generated requests: inputs, rendering rules, retry/continuation
semantics, failure semantics.

### 13. Logging, Status, and Observability (REQUIRED)

- REQUIRED log context fields and message formatting conventions.
- Sinks: not prescribed, but operators MUST see startup/validation/dispatch failures.
- Snapshot interface (OPTIONAL but RECOMMENDED): the fields and error modes.
- Human-readable surfaces: OPTIONAL, driven from core state only, MUST NOT be needed for
  correctness.
- Metrics accounting rules (how to avoid double counting, what is live vs. cumulative).
- OPTIONAL HTTP or API extension: config keys, enablement, endpoints with suggested JSON shapes
  in fenced `json`, status codes, error envelope.

### 14. Failure Model and Recovery Strategy (REQUIRED)

- Numbered failure classes, each with examples.
- Recovery behavior per class.
- Restart semantics: what is and is not restored.
- Operator intervention points.

### 15. Security and Operational Safety (REQUIRED)

- Trust boundary: what implementations MUST/SHOULD state about their posture.
- Mandatory data/filesystem safety rules and RECOMMENDED hardening.
- Secret handling (indirection, no logging, no inheritance by children).
- Trusted-configuration scripts and their implications.
- Hardening guidance: acknowledge externally controlled inputs and list possible measures.

### 16. Reference Algorithms (RECOMMENDED)

One `text` fenced pseudocode block per core routine (startup, cycle, reconcile, dispatch one unit,
worker attempt, exit and retry handling). Use the entity and predicate names from Sections 4 and
8 exactly. Pseudocode is illustrative; normative text lives in the sections above.

When a `Target:` stack is set, these blocks MAY be written in the target language, fenced with its
language tag, using the framework's real types and APIs. They stay illustrative: a sketch a reader
can start from, not a listing to paste.

### 17. Test and Validation Matrix (REQUIRED)

- Define validation profiles: `Core Conformance` (REQUIRED, deterministic),
  `Extension Conformance` (REQUIRED only if the extension ships), `Real Integration Profile`
  (RECOMMENDED, environment-dependent, reported as skipped when skipped).
- One subsection per body area (parsing, resources, adapter, core loop, protocol client,
  observability, CLI/host lifecycle), each a flat bullet list of observable behaviors. Bullets for
  extensions begin with `If <extension> is implemented`.
- Every normative statement in Sections 5 through 15 SHOULD map to at least one bullet here.

### 18. Implementation Checklist (Definition of Done) (REQUIRED)

- `18.1 REQUIRED for Conformance`: one bullet per shippable capability, with the config key and
  default where relevant.
- `18.2 RECOMMENDED Extensions`: extension bullets plus `TODO:` items for known future work.
- `18.3 Operational Validation Before Production`.

### Appendices (OPTIONAL)

One appendix per extension profile: config keys, execution model, scheduling notes,
`Problems to Consider`.

### Open Questions (transient)

A bullet list mirroring every `TBD(Qn)` in the body. Delete the section when empty. A spec with
this section present is a draft by definition.

## Writing rules

- Keywords: `MUST`, `SHOULD`, `MAY`, `REQUIRED`, `OPTIONAL`, `RECOMMENDED` in capitals, in
  backticks only inside the Normative Language section. Prefer `MUST NOT` over "never".
- Every `Implementation-defined` behavior is paired with a duty to document the chosen behavior.
- Language-agnostic by default: no file paths, library or framework names, or source code in an
  implementation language. Backticked identifiers name concepts (`issue.identifier`,
  `hooks.timeout_ms`), not code.
- Stack-bound when the header has a `Target:` line. Then the spec SHOULD use the named stack
  wherever it makes the contract more precise: framework mechanisms (middleware, hooks, DI,
  routing conventions), standard-library and framework types in field definitions, package and
  module layout, file paths, build and test commands, and reference code in Section 16. Rules that
  still hold:
  - Only the language and frameworks the user named are assumed. Any further library is a design
    decision: settle it in the interview before it appears in the spec, and mark it OPTIONAL if
    the system is correct without it.
  - State behavior first, binding second: "The server MUST reject unknown keys with a 400.
    (Next.js: a Route Handler returning `NextResponse.json(..., { status: 400 })`.)" The reader
    can always see what is required versus how the stack meets it.
  - Concept identifiers stay generic (`issue.identifier`); stack identifiers name real code
    (`src/core/scheduler.ts`, `tokio::select!`). Do not blur the two.
- Agent-agnostic: do not name a coding-agent product, IDE, or plugin host unless that product is
  itself an external dependency of the system under spec. The spec is for any implementor.
- Numbers are exact and unit-suffixed in the key name (`interval_ms`, `timeout_ms`). Defaults are
  stated once in the schema and repeated once in the cheat sheet.
- Formulas are written as expressions: `delay = min(10000 * 2^(attempt - 1), max_backoff_ms)`.
- Signatures and pseudocode use `text` fences. Payload examples use `json` fences.
- One canonical term per concept. When a term is generic in the spec but provider-specific in
  reality, say so once ("The name `Issue` is generic; an adapter MAY map it from a card...").
- Use `Note:` and `Important nuance:` blocks for the reasoning a reader will otherwise get wrong.
- Cross-reference by section number ("using Section 4.2"), never by page or by "above".
- Hard-wrap prose at 100 columns for reviewable diffs.
- No dates, version numbers of external tools, or "currently" in normative text. Put legacy
  behavior in a clearly labeled deprecated block if it must be mentioned. The `Target:` header
  line is the one place a minimum stack version may be stated.

## Suggested length

Line counts assume prose hard-wrapped at 100 columns. They are calibrated from the exemplar in
[EXAMPLE-SPEC.md](EXAMPLE-SPEC.md) (about 2300 lines for a long-running orchestrator with one
external protocol, one adapter family, and one appendix). They are a depth signal, not a quota:
a section far under its range usually means a question was never asked; a section far over it
usually means an external schema is being restated or an extension belongs in an appendix.

Whole spec:

- Long-running system with an external protocol and adapters (daemon, orchestrator, service):
  1500-2500 lines.
- Library, SDK, or CLI with no long-lived state: 600-1200 lines.
- A stack-bound spec (`Target:` set) runs about 10-20% longer than the same spec written
  language-agnostic, because of file layout, framework mechanics, and typed reference code.

Per section:

| Section                                 | Lines   | Notes                                        |
| --------------------------------------- | ------- | -------------------------------------------- |
| Header and Normative Language           | 10-15   | Boilerplate; do not grow.                    |
| 1. Problem Statement                    | 20-40   | Boundary bullets are the bulk.               |
| 2. Goals and Non-Goals                  | 20-30   |                                              |
| 3. System Overview                      | 60-90   | Two to four bullets per component.           |
| 4. Core Domain Model                    | 120-180 | One block per entity; identifiers ~30.       |
| 5. Primary Input Contract               | 120-220 | Field-by-field schema dominates.             |
| 6. Configuration Specification          | 80-120  | Cheat sheet is one line per field.           |
| 7. State Machine                        | 70-100  |                                              |
| 8. Core Loop                            | 90-130  | Retry and reconciliation are the long parts. |
| 9. Resource Management and Safety       | 70-100  | Invariants ~20.                              |
| 10. External Protocol Integration       | 120-230 | Longest when policy areas are enumerated.    |
| 11. Adapter / Integration Contract      | 100-150 | Signatures plus contract bullets.            |
| 12. Derived Artifact Construction       | 30-50   |                                              |
| 13. Logging, Status, and Observability  | 100-150 | +100-130 with the OPTIONAL HTTP extension.   |
| 14. Failure Model and Recovery Strategy | 60-90   |                                              |
| 15. Security and Operational Safety     | 60-90   |                                              |
| 16. Reference Algorithms                | 150-250 | One 20-50 line block per core routine.       |
| 17. Test and Validation Matrix          | 100-160 | Scales with normative statements.            |
| 18. Implementation Checklist            | 40-60   |                                              |
| Appendix (each)                         | 40-80   |                                              |
| Open Questions                          | 0       | Present only in drafts.                      |

Sections marked `Not applicable` count as one line. For a library or CLI, Sections 7-11 shrink or
collapse first; Sections 4, 5, 17, and 18 keep their ranges.
