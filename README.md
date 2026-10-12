# Skill Creator

**Create and improve skills for Grok AI agents.**

The skill factory — skills that get smarter with every session through the postmortem cycle.

## What is this?

This is the official Skill Creator skill for Grok. It guides agents (and humans) through creating high-quality, self-improving skills that extend Grok's capabilities.

### Core Principles

1. **Concise is key** — only add what the agent doesn't already know
2. **Match freedom to fragility** — specific scripts for fragile ops, guidelines for open-ended tasks
3. **Every skill gets evals** — if you can't test it, you can't improve it

## Anatomy of a Skill

```
skill-name/
├── SKILL.md              (required)
├── CHANGELOG.md          (required)
├── evals/
│   ├── evals.json        (required)
│   └── triggers.json     (required)
├── scripts/              (optional)
├── references/           (optional)
└── assets/               (optional)
```

## Quick Start

To create a new skill, use the included initializer:

```bash
python3 scripts/init_skill.py my-new-skill --path .
```

Or with minimal structure:

```bash
python3 scripts/init_skill.py my-new-skill --path . --minimal
```

Then follow the process in `SKILL.md`.

## Validation

```bash
python3 scripts/quick_validate.py path/to/skill
```

## Files

- `SKILL.md` — Full instructions for creating skills
- `scripts/init_skill.py` — Initialize a new skill directory
- `scripts/quick_validate.py` — Validate skill structure
- `references/` — Design patterns, workflows, output patterns
- `evals/` — Test cases and trigger phrases

## License

See original source for licensing. This is a release of the bundled skill-creator for Grok.

## Source

Released from the bundled skill-creator included with Grok Computer.
