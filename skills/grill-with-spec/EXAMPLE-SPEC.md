# Example spec (calibration only)

The exemplar for depth, tone, and structure is [SYMPHONY-SPEC.md](SYMPHONY-SPEC.md), a verbatim,
bundled copy of openai/symphony's `SPEC.md` (Apache-2.0; license and NOTICE in
[SYMPHONY-SPEC.LICENSE](SYMPHONY-SPEC.LICENSE)). It ships with the skill so it can be read locally;
never fetch it.

Skim its headings once, then write the new spec from [SPEC-FORMAT.md](SPEC-FORMAT.md).

## What to take from it

- The shape: how a section moves from prose to numbered rules to cheat sheets, how entities are
  laid out field by field, how every normative statement reappears in the test matrix.
- The tone: `MUST`/`SHOULD`/`MAY` prose, `Implementation-defined` paired with a duty to document,
  `Note:` and `Important nuance:` blocks for the reasoning a reader would otherwise get wrong.
- The scale: about 2300 lines at a 100-column wrap for a long-running orchestrator with one
  external protocol, one adapter family, and one appendix. The "Exemplar" column in
  SPEC-FORMAT.md's Length section is measured from it.

## What not to take from it

- Its content. It specifies a different system. Never copy its rules, entities, or defaults into a
  spec for another system.
- Its agent. It is language-agnostic in form but agent-specific in subject: it integrates one
  particular coding-agent protocol. A spec produced by this skill stays agent-agnostic unless that
  agent is an actual external dependency of the system under spec, and language-agnostic unless the
  user named a target stack.
- Its length. That is what its system needed, not a length to aim for. Estimate a new spec from
  its own minimum (SPEC-FORMAT.md, "Length").

## Heading map

Names will change per system; the order of concerns should not.

```text
# <System> Specification
Status / Purpose
Normative Language
1. Problem Statement
2. Goals and Non-Goals
3. System Overview          (components, layers, external dependencies)
4. Core Domain Model        (entities, identifiers, normalization)
5. Primary Input Contract   (file/schema the user authors)
6. Configuration            (resolution, reload, preflight, cheat sheet)
7. State Machine
8. Core Loop                (poll, eligibility, concurrency, retry, reconcile)
9. Resource Management      (layout, hooks, safety invariants)
10. External Protocol       (source of truth is the protocol, not this spec)
11. Adapter Contract
12. Derived Artifact Construction
13. Observability
14. Failure Model
15. Security
16. Reference Algorithms    (language-agnostic pseudocode, or target-language sketches)
17. Test and Validation Matrix
18. Definition of Done
Appendix: OPTIONAL extensions
```
