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

#### Bundled Resources (optional)

##### Scripts (`scripts/`)

Executable code (Python/Bash/etc.) for tasks that require deterministic reliability or are repeatedly rewritten.

- **When to include**: When the same code is being rewritten repeatedly or deterministic reliability is needed
- **Example**: `scripts/rotate_pdf.py` for PDF rotation tasks
- **Benefits**: Token efficient, deterministic, may be executed without loading into context
- **Note**: Scripts may still need to be read by the agent for patching or environment-specific adjustments

##### References (`references/`)

Documentation and reference material intended to be loaded as needed into context to inform the agent's process and thinking.

- **When to include**: For documentation that the agent should reference while working
- **Examples**: `references/finance.md` for financial schemas, `references/mnda.md` for company NDA template, `references/policies.md` for company policies, `references/api_docs.md` for API specifications
- **Use cases**: Database schemas, API documentation, domain knowledge, company policies, detailed workflow guides
- **Benefits**: Keeps SKILL.md lean, loaded only when the agent determines it's needed
- **Best practice**: If files are large (>10k words), include grep search patterns in SKILL.md
- **Avoid duplication**: Information should live in either SKILL.md or references files, not both. Prefer references files for detailed information unless it's truly core to the skill—this keeps SKILL.md lean while making information discoverable without hogging the context window. Keep only essential procedural instructions and workflow guidance in SKILL.md; move detailed reference material, schemas, and examples to references files.

##### Assets (`assets/`)

Files not intended to be loaded into context, but rather used within the output the agent produces.

- **When to include**: When the skill needs files that will be used in the final output
- **Examples**: `assets/logo.png` for brand assets, `assets/slides.pptx` for PowerPoint templates, `assets/frontend-template/` for HTML/React boilerplate, `assets/font.ttf` for typography
- **Use cases**: Templates, images, icons, boilerplate code, fonts, sample documents that get copied or modified
- **Benefits**: Separates output resources from documentation, enables the agent to use files without loading them into context

#### What to Not Include in a Skill

A skill should only contain essential files that directly support its functionality. Do NOT create:

- README.md, INSTALLATION_GUIDE.md, QUICK_REFERENCE.md
- Setup/testing procedures, user-facing documentation
- Auxiliary context about the creation process

The skill should contain ONLY what an AI agent needs to do the job. **Exception**: CHANGELOG.md IS required — it tracks how the skill evolves through postmortems.

### Progressive Disclosure Design Principle

Skills use a three-level loading system to manage context efficiently:

1. **Metadata (name + description)** - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed by the agent (Unlimited because scripts can be executed without reading into context window)

#### Progressive Disclosure Patterns

Keep SKILL.md body to the essentials and under 500 lines to minimize context bloat. Split content into separate files when approaching this limit. When splitting out content into other files, it is very important to reference them from SKILL.md and describe clearly when to read them, to ensure the reader of the skill knows they exist and when to use them.

**Key principle:** When a skill supports multiple variations, frameworks, or options, keep only the core workflow and selection guidance in SKILL.md. Move variant-specific details (patterns, examples, configuration) into separate reference files.

**Pattern 1: High-level guide with references**

```markdown
# PDF Processing

## Quick start

Extract text with pdfplumber:
[code example]

## Advanced features

- **Form filling**: See [FORMS.md](FORMS.md) for complete guide
- **API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
- **Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
```

the agent loads FORMS.md, REFERENCE.md, or EXAMPLES.md only when needed.

**Pattern 2: Domain-specific organization**

For Skills with multiple domains, organize content by domain to avoid loading irrelevant context:

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

When a user asks about sales metrics, the agent only reads sales.md.

Similarly, for skills supporting multiple frameworks or variants, organize by variant:

```
cloud-deploy/
├── SKILL.md (workflow + provider selection)
└── references/
    ├── aws.md (AWS deployment patterns)
    ├── gcp.md (GCP deployment patterns)
    └── azure.md (Azure deployment patterns)
```

When the user chooses AWS, the agent only reads aws.md.

**Pattern 3: Conditional details**

Show basic content, link to advanced content:

```markdown
# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

the agent reads REDLINING.md or OOXML.md only when the user needs those features.

**Important guidelines:**

- **Avoid deeply nested references** - Keep references one level deep from SKILL.md. All reference files should link directly from SKILL.md.
- **Structure longer reference files** - For files longer than 100 lines, include a table of contents at the top so the agent can see the full scope when previewing.

## Success Criteria

Before creating a skill, define what "good" looks like:

| Metric | Target |
|---|---|
| **Trigger accuracy** | ≥90% — the right phrases activate the skill, wrong phrases don't |
| **Autonomous completion** | Agent finishes the task without asking for clarification |
| **Consistency** | Same input → same quality output across sessions |
| **Context efficiency** | SKILL.md < 500 lines. Total with references < 5k words loaded |

## Skill Creation Process

1. Define 2-3 concrete use cases
2. Plan reusable contents (scripts, references, assets)
3. Initialize the skill directory
4. Write SKILL.md + resources
5. Add required files (CHANGELOG, evals, triggers)
6. Iterate via `/postmortem` after real usage

Follow in order. Skip only with clear reason.

### Step 1: Understanding the Skill with Concrete Examples

Skip this step only when the skill's usage patterns are already clearly understood. It remains valuable even when working with an existing skill.

To create an effective skill, clearly understand concrete examples of how the skill will be used. This understanding can come from either direct user examples or generated examples that are validated with user feedback.

#### Knowledge Sources

When building skills for internal tools or frameworks, combine multiple knowledge sources for comprehensive coverage:

1. **Source code** (primary) - README.md files, docstrings, launcher scripts, and implementation details provide accurate technical specifications
2. **Internal documentation** (secondary) - Notion pages, Confluence, internal wikis provide workflow context and best practices
3. **User examples** (validation) - Real usage patterns confirm what's actually needed

**Pattern: Notion + Source Code**

For internal skills, this combination works well:
1. Use Playwright to navigate internal Notion/docs pages for high-level workflows
2. Read source README.md and key implementation files for technical details
3. Cross-reference to ensure accuracy (source code is authoritative for APIs, docs are authoritative for workflows)

For example, when building an image-editor skill, relevant questions include:

- "What functionality should the image-editor skill support? Editing, rotating, anything else?"
- "Can you give some examples of how this skill would be used?"
- "I can imagine users asking for things like 'Remove the red-eye from this image' or 'Rotate this image'. Are there other ways you imagine this skill being used?"
- "What would a user say that should trigger this skill?"

To avoid overwhelming users, avoid asking too many questions in a single message. Start with the most important questions and follow up as needed for better effectiveness.

Conclude this step when there is a clear sense of the functionality the skill should support.

### Step 2: Planning the Reusable Skill Contents

To turn concrete examples into an effective skill, analyze each example by:

1. Considering how to execute on the example from scratch
2. Identifying what scripts, references, and assets would be helpful when executing these workflows repeatedly

Example: When building a `pdf-editor` skill to handle queries like "Help me rotate this PDF," the analysis shows:

1. Rotating a PDF requires re-writing the same code each time
2. A `scripts/rotate_pdf.py` script would be helpful to store in the skill

Example: When designing a `frontend-webapp-builder` skill for queries like "Build me a todo app" or "Build me a dashboard to track my steps," the analysis shows:

1. Writing a frontend webapp requires the same boilerplate HTML/React each time
2. An `assets/hello-world/` template containing the boilerplate HTML/React project files would be helpful to store in the skill

Example: When building a `big-query` skill to handle queries like "How many users have logged in today?" the analysis shows:

1. Querying BigQuery requires re-discovering the table schemas and relationships each time
2. A `references/schema.md` file documenting the table schemas would be helpful to store in the skill

To establish the skill's contents, analyze each concrete example to create a list of the reusable resources to include: scripts, references, and assets.

### Step 3: Initializing the Skill

At this point, it is time to actually create the skill.

Skip this step only if the skill being developed already exists, and iteration or packaging is needed. In this case, continue to the next step.

When creating a new skill from scratch, always run the `init_skill.py` script. The script conveniently generates a new template skill directory that automatically includes everything a skill requires, making the skill creation process much more efficient and reliable.

Usage:

```bash
scripts/init_skill.py <skill-name> --path <output-directory>
scripts/init_skill.py <skill-name> --path <output-directory> --minimal
```

Options:
- `--minimal`: Create only SKILL.md without scripts/, references/, assets/ directories. Use for skills that don't need bundled resources.

The script:

- Creates the skill directory at the specified path
- Generates a SKILL.md template with proper frontmatter and TODO placeholders
- Creates example resource directories: `scripts/`, `references/`, and `assets/`
- Adds example files in each directory that can be customized or deleted

After initialization, customize or remove the generated SKILL.md and example files as needed.

### Step 4: Edit the Skill

When editing the (newly-generated or existing) skill, remember that the skill is being created for another instance of the agent to use. Include information that would be beneficial and non-obvious to the agent. Consider what procedural knowledge, domain-specific details, or reusable assets would help another agent execute these tasks more effectively.

#### Learn Proven Design Patterns

Consult these helpful guides based on your skill's needs:

- **Multi-step processes**: See references/workflows.md for sequential workflows and conditional logic
- **Specific output formats or quality standards**: See references/output-patterns.md for template and example patterns

These files contain established best practices for effective skill design.

#### Start with Reusable Skill Contents

To begin implementation, start with the reusable resources identified above: `scripts/`, `references/`, and `assets/` files. Note that this step may require user input. For example, when implementing a `brand-guidelines` skill, the user may need to provide brand assets or templates to store in `assets/`, or documentation to store in `references/`.

Added scripts must be tested by actually running them to ensure there are no bugs and that the output matches what is expected. If there are many similar scripts, only a representative sample needs to be tested to ensure confidence that they all work while balancing time to completion.

Any example files and directories not needed for the skill should be deleted. The initialization script creates example files in `scripts/`, `references/`, and `assets/` to demonstrate structure, but most skills won't need all of them.

#### Update SKILL.md

**Writing Guidelines:** Always use imperative/infinitive form.

##### Frontmatter

Write the YAML frontmatter with these **required** fields:

```yaml
---
name: skill-name
description: "What it does. Use when X, Y, Z."
type: workflow  # knowledge | workflow | orchestrator | tool
lifecycle: active
---
```

- `name`: Must use only lowercase letters (a-z), digits (0-9), and hyphens (-). Must start and end with a letter or digit. Must be 2-64 characters long (e.g. `my-skill`). Must match directory name exactly
- `description`: **Quoted string, one sentence.** What the skill does + when to use it. No inner double quotes. This is the primary triggering mechanism.
- `type`: One of `knowledge` (reference material), `workflow` (step-by-step process), `orchestrator` (multi-agent coordination), `tool` (CLI/API integration)
- `lifecycle`: Always `active` for new skills

##### Body Structure

Use this structure for the SKILL.md body:

```markdown
# Skill Name — One-Line Purpose

[1-2 sentence overview. When to use.]

## [Core Workflow / Instructions]
[Numbered steps. Specific commands. Error handling.]

## [Reference / Key Info]
[Tables, schemas, patterns — only what's essential.
Move detailed content to references/.]

## Common Issues
[Troubleshooting table if applicable]
```

**Writing rules:**
- Imperative voice: "Run X", "Check Y" — not "You should run X"
- Specific commands: `bk job log JOB_ID` — not "check the logs"
- Error handling: what to do when it fails, not just the happy path
- Link to references/: "See `references/X.md` for details" instead of inlining 200 lines

### Step 5: Add Required Files

Every skill needs these files beyond SKILL.md:

**`CHANGELOG.md`** — Track all changes:
```markdown
# Changelog
## YYYY-MM-DD
- Initial skill creation
```

**`evals/evals.json`** — 3+ test cases (NOT stubs):
```json
{
  "skill": "skill-name",
  "test_cases": [
    {"name": "test_name", "input": "user prompt", "assertions": ["expected behavior"]},
    {"name": "test_name_2", "input": "another prompt", "assertions": ["another behavior"]}
  ]
}
```

**`evals/triggers.json`** — 3+ trigger phrases:
```json
{
  "skill": "skill-name",
  "test_cases": [
    {"input": "trigger phrase", "should_trigger": true},
    {"input": "non-trigger phrase", "should_trigger": false}
  ]
}
```

**Validate** — Run before ending session:
```bash
python3 /usr/share/grok/bundled-skills/bundled__postmortem/scripts/validate-skill.py .grok/skills/skill-name
```
Must pass with 0 errors. Fix any issues the validator reports.

### Step 5b: Add Skill Graph (for skills with 3+ reference files)

When a skill has 3+ reference files, organize them as a **skill graph** — a traversable network of interconnected knowledge nodes. See the `skill-graphs` skill for the full spec.

**Quick checklist:**

1. **Add YAML frontmatter** to every reference file:
   ```yaml
   ---
   description: "What this file covers and when to read it"
   connections: [related-file-a, related-file-b]
   ---
   ```

2. **Create `references/INDEX.md`** — the graph entry point:
   ```markdown
   ---
   description: "Knowledge graph index for [skill-name]. Start here."
   ---
   # [Skill Name] Knowledge Graph
   ## Core Concepts
   - [[node-a]] — what it covers and why it matters
   - [[node-b]] — related context for understanding X
   ```

3. **Weave `[[wikilinks]]` into prose** inside each reference file:
   ```markdown
   When investigating latency, the [[architecture]] shows which
   service boundary to check, and [[common-alerts]] lists known patterns.
   ```

4. **Add cross-skill links** in SKILL.md:
   ```markdown
   ## Knowledge Graph
   Start at `references/INDEX.md` for the full map.
   ### Cross-Skill Connections
   - [[voice-knowledge:]] — foundational domain knowledge
   - [[axiom:query-patterns]] — APL query templates
   ```

**Wikilink resolution:** `[[name]]` → same skill's `references/name.md`, `[[skill:name]]` → another skill's `references/name.md`, `[[skill:]]` → another skill's `SKILL.md`.

### Step 6: Iterate via Postmortem

Skills improve through the **postmortem** cycle — run `/postmortem` after every session:

1. Use the skill on real tasks
2. Run `/postmortem` — it traces mistakes and enriches skills automatically
3. The postmortem's 6-question audit identifies what to TUNE or GROW
4. Changes persist via gcsfuse, and the skill gets smarter each session

**This is the most important step.** Skills that don't go through postmortem stay stale.

## Common Issues & Fixes

| Problem | Symptom | Fix |
|---|---|---|
| **Undertrigger** | Skill doesn't activate on valid requests | Add more trigger phrases to description + evals/triggers.json |
| **Overtrigger** | Skill activates on unrelated requests | Add "NOT for X" to description. Add `should_trigger: false` cases |
| **Ignores instructions** | Agent doesn't follow SKILL.md steps | Move critical rules to top. Use numbered steps. Bold the imperative |
| **Context bloat** | Skill loads but response is slow/truncated | SKILL.md > 500 lines → split into references/. Remove prose, keep tables |
| **Stale content** | Skill references outdated APIs/paths | Run `/postmortem` — Q1 catches this. Update, don't append |
| **No improvement** | Same mistakes repeat across sessions | Missing evals. Every postmortem TUNE needs a matching regression eval |
