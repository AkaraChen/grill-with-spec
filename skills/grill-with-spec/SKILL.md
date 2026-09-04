---
name: grill-with-spec
description: Relentless design interview that produces a single SPEC.md, an RFC 2119-style, language-agnostic and agent-agnostic system specification written and sharpened inline as decisions settle. Use when the user wants to grill a design into a spec, write or revise a SPEC.md, or asks to "spec this out" before implementation.
disable-model-invocation: true
---

# Grill with Spec

Two disciplines run together in one session:

1. **Grilling**: interview the user relentlessly, as a design tree worked in rounds, until nothing
   is silently assumed.
2. **Spec writing**: capture every settled decision in one `SPEC.md` the moment it settles, in the
   normative style defined in [SPEC-FORMAT.md](SPEC-FORMAT.md).

The deliverable is `SPEC.md`, not the conversation. A reader who has never seen the chat, and who
is using any coding agent or none at all, must be able to implement the system from the file
alone.

## Before the first round

1. Locate the target. Default is `SPEC.md` at the repo root; honor a path the user names. There is
   exactly one spec file; do not split it into a docs tree.
2. If it exists, read it fully. Settled sections are settled. The frontier starts at `TBD(...)`
   markers, the `## Open Questions` section, internal contradictions, and sections required by
   SPEC-FORMAT.md that are missing.
3. If it does not exist, create it at the end of round one from the skeleton in SPEC-FORMAT.md.
   Never let several rounds pass before the file exists.
4. Gather facts before asking anything: existing code, README, domain glossary, ADRs, and the
   documentation of any external protocol or provider the system will talk to. Finding facts is
   your job; making decisions is the user's.
5. Skim the headings of [EXAMPLE-SPEC.md](EXAMPLE-SPEC.md) once to calibrate depth and tone. It
   specifies a different system; never copy its content into a new spec.

## The interview

Map the design as a **design tree**: every decision branches into the decisions that hang off it.
Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already
settled: the questions you can ask now without guessing at answers you have not heard yet. Ask the
whole frontier in one round, numbered, each with your recommended answer, then wait.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, may be several paragraphs, may offer choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body>

➡️ <your recommended answer>
```

Rules of the interview:

- Seed the tree in spec order: identity and boundary first (problem statement, what the system is
  and is not, goals and non-goals), then components and layers, domain entities, the primary input
  contract, configuration, state machine, core loop, safety invariants, external protocols,
  adapters, observability, failure model, security, tests, definition of done. A question about a
  lower section belongs to a later round if the sections it depends on are still open.
- Draw questions from [QUESTION-BANK.md](QUESTION-BANK.md) and adapt them to the system. Do not
  recite the bank; skip questions the facts already answer.
- A question whose answer depends on another question still open in this round belongs to a later
  round.
- When a frontier question needs a fact from the environment, look it up (or dispatch a helper
  agent if the host supports that). Only the questions downstream of that fact wait; ask the rest
  of the frontier now.
- Every question carries a recommended answer. Prefer the answer that keeps the spec smaller,
  the core REQUIRED surface narrower, and more behavior `Implementation-defined` with a duty to
  document.
- Challenge vocabulary. When the user uses two words for one concept, or one word for two, stop
  and settle the canonical term before writing it into the spec.
- Stress-test relationships with concrete scenarios ("issue moves to Done mid-run; what happens to
  the workspace?") and turn the answer into a normative rule.
- Conduct the interview in the user's language. Write `SPEC.md` in English unless the user asks
  otherwise or an existing spec is in another language.

## Writing SPEC.md inline

After each answered round, before asking the next:

1. Update every section the answers touch. Write decisions as normative prose (`MUST`, `SHOULD`,
   `MAY`), never as "the user said" or "we decided".
2. Where a settled decision depends on one still open, leave `TBD(Qn): <one line>` at the exact
   spot the text will go, and mirror it in `## Open Questions` at the end of the file.
3. Remove `TBD` markers as they settle. Bump `Status: Draft vN` whenever a round changes normative
   content.
4. Tell the user in one or two lines what changed (`SPEC.md: added 4.1.7 Retry Entry, resolved
   TBD(Q7), bumped to Draft v3`), then ask the next round.

Writing rules (full detail in SPEC-FORMAT.md):

- RFC 2119 keywords in capitals. Use `Implementation-defined` when the spec deliberately leaves a
  policy open, and require implementations to document their choice.
- Language-agnostic: no file paths, library names, or source code. Reference algorithms are
  pseudocode in `text` fences.
- Agent-agnostic: do not assume which coding agent, IDE, or plugin host will implement or operate
  the system. Name protocols and executables only when they are the system's actual external
  dependency. A reader using a different agent must still be able to implement the spec.
- One canonical term per concept, defined once in the domain model and used everywhere after.
- Separate REQUIRED core from OPTIONAL extensions. Extensions live in their own sections or
  appendices and MUST NOT be needed for core correctness.
- Deliberate redundancy is allowed for cheat sheets and checklists; say that it is intentional.
- Point at external protocols as the source of truth instead of restating their schemas.

## Done

The session is done when all of the following hold:

- The frontier is empty and `## Open Questions` is empty (delete the section).
- Every REQUIRED section of SPEC-FORMAT.md is present, or marked "Not applicable" with a reason.
- Every normative behavior in the body appears in the test matrix and the definition of done.
- Terminology is consistent from the domain model through the reference algorithms.
- The user confirms shared understanding.

Do not start implementing. Offer follow-ups (tickets, ADRs, a prototype) only after the user
confirms the spec.

## Additional resources

- [SPEC-FORMAT.md](SPEC-FORMAT.md): section skeleton, per-section guidance, and writing rules.
- [QUESTION-BANK.md](QUESTION-BANK.md): probing questions per spec section.
- [EXAMPLE-SPEC.md](EXAMPLE-SPEC.md): a complete exemplar spec for calibrating depth and tone.
