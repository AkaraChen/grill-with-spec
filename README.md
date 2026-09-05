# grill-with-spec

An [Agent Plugin](https://agent-plugins.org) (and Cursor plugin) that runs a relentless design
interview and writes the result into a single `SPEC.md`: an RFC 2119-style specification that is
**agent-agnostic** and, by default, **language-agnostic**. The point is software that is
**auditable** and **rebuildable**: the file holds the complete spec, so any implementation can be
checked against it, and a human or any coding agent can rebuild the system from the file alone.

Name a language or framework when you invoke the skill and the spec binds to it: file layout,
framework mechanisms, typed entities, build and test commands, and reference code in that language
are all fair game. Leave the stack unnamed and the spec stays implementable in any language.

It follows the same interview pattern as
[`grill-with-docs`](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs)
(design tree, rounds, frontier, recommended answers), but the deliverable is one spec file, not a
glossary and ADRs.

The produced spec does not assume Cursor, Codex, Claude, or any other agent host. Name a coding
agent in `SPEC.md` only when that agent is an actual external dependency of the system under spec.

## Layout

```
plugin.json                         # Agent Plugins manifest (Cursor loads this)
.cursor-plugin/plugin.json          # Cursor Plugin manifest
skills/
└── grill-with-spec/
    ├── SKILL.md                    # interview protocol + inline spec-writing rules
    ├── SPEC-FORMAT.md              # section skeleton, per-section guidance, writing rules
    ├── QUESTION-BANK.md            # probing questions per spec section
    └── EXAMPLE-SPEC.md             # heading map + link to a full exemplar for calibration
```

## Install

Use the [skills CLI](https://github.com/vercel-labs/skills) to install `grill-with-spec` into your
coding agent:

```bash
npx skills add AkaraChen/grill-with-spec
```

The CLI detects installed agents and prompts for project vs global scope. Add `-g` for a user-wide
install, or `-a` to target a specific agent:

```bash
npx skills add AkaraChen/grill-with-spec -g
npx skills add AkaraChen/grill-with-spec -a cursor
```

In Cursor you can also install this repository as a plugin (Customize → Plugins, or a team
marketplace import of the GitHub repo). Cursor discovers `skills/grill-with-spec` from the root
`plugin.json`.

The skill is a standard [Agent Skill](https://agentskills.io/specification) (`SKILL.md` +
supporting files). Host-specific invocation is up to the client (`/grill-with-spec` in Cursor).

## Use

Invoke the skill by name, then describe the system you want to specify. The agent asks numbered
questions in rounds, each with a recommended answer, and updates `SPEC.md` after every round. When
the host exposes an ask-user tool (Cursor's and Claude Code's question prompts, Codex's
`request_user_input`), the questions arrive as a structured form with the recommended answer
preselected; otherwise they are posted as Markdown. After round one the agent estimates the
minimum complete length for your system and reports progress against it; the spec should be as
long as a rebuild and an audit need, and no longer. The session ends when the `## Open Questions`
section is empty and you confirm shared understanding.

The skill has `disable-model-invocation: true` in hosts that honor that field, so it only runs
when you name it.
