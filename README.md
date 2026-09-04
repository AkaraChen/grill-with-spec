# grill-with-spec

A Cursor Agent Skill that runs a relentless design interview and writes the result into a single
`SPEC.md`: an RFC 2119-style, language-agnostic system specification that a human or a coding
agent can implement from the file alone.

It follows the same pattern as [`grill-with-docs`](https://github.com/mattpocock/skills/tree/main/skills/engineering/grill-with-docs)
(design tree, rounds, frontier, recommended answers), but instead of producing `CONTEXT.md` and
ADRs as you go, it produces and continuously sharpens one spec file shaped like the
[openai/symphony `SPEC.md`](https://github.com/openai/symphony/blob/main/SPEC.md).

## Layout

```
.cursor/skills/grill-with-spec/
├── SKILL.md           # interview protocol + inline spec-writing rules
├── SPEC-FORMAT.md     # section skeleton, per-section guidance, writing rules
├── QUESTION-BANK.md   # probing questions per spec section
└── EXAMPLE-SPEC.md    # complete exemplar spec (Symphony, Apache-2.0) for calibration
```

## Use

In this repo the skill is already discoverable as a project skill. Invoke it explicitly:

```
/grill-with-spec
```

Describe the system you want to specify. The agent asks numbered questions in rounds, each with a
recommended answer, and updates `SPEC.md` at the repo root after every round. The session ends when
the `## Open Questions` section is empty and you confirm shared understanding.

To make it available across all your projects, copy the directory to your personal skills folder:

```bash
cp -r .cursor/skills/grill-with-spec ~/.cursor/skills/
```

The skill has `disable-model-invocation: true`, so it only runs when you name it.
