<!--
Calibration reference for grill-with-spec.

The finished-spec depth and tone to match is openai/symphony SPEC.md
(https://github.com/openai/symphony/blob/main/SPEC.md), Apache License 2.0.
That document specifies a different system and a particular coding-agent
protocol. Skim its headings. Never copy its content into a spec for another
system, and never treat its agent-specific names as defaults.
-->

# Example spec (calibration only)

Read the headings of [openai/symphony `SPEC.md`](https://github.com/openai/symphony/blob/main/SPEC.md)
once, then write the new spec from [SPEC-FORMAT.md](SPEC-FORMAT.md).

That exemplar is language-agnostic in form and agent-specific in subject: it
integrates one coding-agent protocol. A spec produced by this skill MUST stay
agent-agnostic unless that agent is an actual external dependency of the system
under spec. It stays language-agnostic too unless the user named a target stack,
in which case the same headings are filled with stack-specific detail.

The exemplar is about 2300 lines at a 100-column wrap; the "Exemplar" column in
SPEC-FORMAT.md's Length section is measured from it. That is what its system
needed, not a length to aim for. Estimate a new spec from its own minimum.

Heading map to match for depth (names will change per system):

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
16. Reference Algorithms    (language-agnostic pseudocode)
17. Test and Validation Matrix
18. Definition of Done
Appendix: OPTIONAL extensions
```
