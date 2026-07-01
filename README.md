# AGENT_FLOW

A collection of Claude Code skills that encapsulate reusable engineering practices. Each skill is loaded on demand per conversation.

## Structure

```
AGENT_FLOW/
├── README.md
└── skills/
    └── <skill-name>/
        ├── SKILL.md        # Skill definition and behavior
        └── agents/         # Optional auxiliary agent configs
```

## Skills

| Skill | Description |
|-------|-------------|
| [pre-coding-design-review](./skills/pre-coding-design-review/SKILL.md) | Structured design review before non-trivial code changes |

## Usage

Invoke a skill in a Claude Code conversation:

```
/<skill-name>
```

Or describe your task directly — Claude Code auto-matches skills based on context.

## Adding a New Skill

1. Create `skills/<skill-name>/` directory
2. Write `SKILL.md` (follow the existing skill format)
3. Optionally add auxiliary agent configs under `agents/`

A `SKILL.md` should include: name, trigger conditions, workflow, output spec, decision principles.
