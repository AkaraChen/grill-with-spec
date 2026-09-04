# grill-with-spec

An [Agent Plugin](https://agent-plugins.org) (and Cursor plugin) that runs a relentless design
interview and writes the result into a single `SPEC.md`: an RFC 2119-style specification that is
**language-agnostic** and **agent-agnostic**. A human, or any coding agent, should be able to
implement the system from the file alone.

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

### Cursor

Install this repository as a plugin (Customize → Plugins, or a team marketplace import of the
GitHub repo). Cursor discovers `skills/grill-with-spec` from the root `plugin.json`.

Local development:

```bash
ln -s /path/to/grill-with-spec ~/.cursor/plugins/local/grill-with-spec
```

Then reload the window and confirm the skill appears in Customize.

### Any Agent Skills host

Copy or symlink the skill directory into the host's skills root:

```bash
cp -r skills/grill-with-spec ~/.agents/skills/
```

The skill is a standard [Agent Skill](https://agentskills.io/specification) (`SKILL.md` +
supporting files). Host-specific invocation is up to the client (`/grill-with-spec` in Cursor).

## Use

Invoke the skill by name, then describe the system you want to specify. The agent asks numbered
questions in rounds, each with a recommended answer, and updates `SPEC.md` after every round. The
session ends when the `## Open Questions` section is empty and you confirm shared understanding.

The skill has `disable-model-invocation: true` in hosts that honor that field, so it only runs
when you name it.
