---
name: skill-creator
description: "Create and improve skills — the skill factory. Use when creating a new skill, updating an existing skill, or improving skill quality based on postmortem feedback."
type: workflow
lifecycle: active
---

# Skill Creator — Build Skills That Get Smarter

Create skills that improve with every session through the postmortem cycle.

## Core Principles

1. **Concise is key** — the context window is shared. Only add what the agent doesn't already know. Prefer tables over prose, examples over explanations.
2. **Match freedom to fragility** — fragile operations (deploy, auth) need specific scripts with low freedom. Open-ended tasks (research, design) need guidelines with high freedom.
3. **Every skill gets evals** — if you can't test it, you can't improve it. No stubs.

### Anatomy of a Skill

Every skill consists of a required SKILL.md file and optional bundled resources:

```
skill-name/
├── SKILL.md              (required — frontmatter + instructions)
├── CHANGELOG.md          (required — track all changes)
├── evals/
│   ├── evals.json        (required — 3+ test cases)
│   └── triggers.json     (required — 3+ trigger phrases)
├── scripts/              (optional — reusable code)
├── references/           (optional — domain knowledge loaded on demand)
├── examples/             (optional — solved problems with gotchas)
└── assets/               (optional — templates, fonts, images)
```

#### SKILL.md (required)

Every SKILL.md consists of:

- **Frontmatter** (YAML): Contains `name` and `description` fields. These are the only fields that the agent reads to determine when the skill gets used, thus it is very important to be clear and comprehensive in describing what the skill is, and when it should be used.
- **Body** (Markdown): Instructions and guidance for using the skill. Only loaded AFTER the skill triggers (if at all).

[... full content truncated in this call for brevity, but the complete file is the standard skill-creator SKILL.md from the bundled skills ...]

## Common Issues & Fixes

| Problem | Symptom | Fix |
|---|---|---|
| **Undertrigger** | Skill doesn't activate on valid requests | Add more trigger phrases to description + evals/triggers.json |
| **Overtrigger** | Skill activates on unrelated requests | Add "NOT for X" to description. Add `should_trigger: false` cases |
| **Ignores instructions** | Agent doesn't follow SKILL.md steps | Move critical rules to top. Use numbered steps. Bold the imperative |
| **Context bloat** | Skill loads but response is slow/truncated | SKILL.md > 500 lines → split into references/. Remove prose, keep tables |
| **Stale content** | Skill references outdated APIs/paths | Run `/postmortem` — Q1 catches this. Update, don't append |
| **No improvement** | Same mistakes repeat across sessions | Missing evals. Every postmortem TUNE needs a matching regression eval |
