---
name: generate-skill
description: Generate agent skills from OSS project documentation in this repository. Reads source docs from sources/{project}/docs/, synthesizes them into skill reference files under skills/{project}/. Use when the user asks to generate skills, create skills from docs, update existing skills, add a new project for skill generation, or mentions skill generation workflow. Also use when working with the skills/ or sources/ directories in this repo.
---

# Skill Generation from Project Documentation

Generate [Agent Skills](https://agentskills.io/home) from OSS project documentation following the repository conventions in AGENTS.md.

## Quick Reference

- **Source docs:** `sources/{project}/docs/`
- **Project-specific instructions:** `instructions/{project}.md` (if exists)
- **Output:** `skills/{project}/` with SKILL.md, GENERATION.md, and `references/*.md`
- **Project list:** `meta.ts` (the `submodules` object)

## Workflow: Create New Skills

1. **Read `meta.ts`** to confirm the project exists in `submodules`
2. **Read project-specific instructions** from `instructions/{project}.md` if it exists
3. **Read all docs** from `sources/{project}/docs/` thoroughly
4. **Plan the skill structure** — identify core concepts, features, best practices, and advanced topics
5. **Generate reference files** in `skills/{project}/references/`, one concept per file
6. **Generate `SKILL.md`** index with tables linking to all references
7. **Generate `GENERATION.md`** with the current git SHA of the source submodule

### Step 3: Reading Docs

Explore the docs directory structure first, then read systematically. Skip content that:
- Is user-facing getting-started or installation guides
- Covers concepts LLM agents already know from training data
- Is introductory fluff without practical value

Focus on content that:
- Describes unique APIs, patterns, or conventions specific to this project
- Shows practical usage patterns with code examples
- Explains non-obvious behavior, gotchas, or best practices

### Step 5: Writing Reference Files

Categorize references with prefixed filenames:
- `core-` — fundamental concepts and APIs
- `features-` — specific feature areas
- `best-practices-` — recommended patterns and conventions
- `advanced-` — complex or niche topics

Create additional category prefixes as needed to organize the content well.

Each reference file follows this format:

```markdown
---
name: {kebab-case-name}
description: {one-line description of what this covers}
---

# {Concept Name}

Brief description.

## Usage

Code examples and practical patterns.

## Key Points

- Important detail 1
- Important detail 2

<!--
Source references:
- {url-to-source-doc-1}
- {url-to-source-doc-2}
-->
```

**Writing guidelines:**
- Rewrite for agent consumption — don't copy docs verbatim
- Be practical — focus on usage patterns and working code examples
- Be concise — remove fluff, keep only essential information
- One concept per file — split large topics into separate files
- Explain why, not just how — help agents make informed decisions

### Step 6: Writing SKILL.md

```markdown
---
name: {project-name}
description: {what the skill enables agents to do with this project}
metadata:
  author: Minsu Lee
  version: "{YYYY.M.D}"
  source: Generated from {repo-url}, scripts located at https://github.com/amondnet/skills
---

> The skill is based on {project} v{version}, generated at {date}.

{Concise summary of the project — 2-3 sentences max}

## Core References

| Topic | Description | Reference |
|-------|-------------|-----------|
| ... | ... | [name](references/core-name.md) |

## Features

### {Feature Area}

| Topic | Description | Reference |
|-------|-------------|-----------|
| ... | ... | [name](references/features-name.md) |
```

### Step 7: Writing GENERATION.md

```bash
cd sources/{project} && git rev-parse HEAD
```

```markdown
# Generation Info

- **Source:** `sources/{project}`
- **Git SHA:** `{full-sha}`
- **Generated:** {YYYY-MM-DD}
```

## Workflow: Update Existing Skills

1. **Read `GENERATION.md`** to get the previous SHA
2. **Check what changed:**
   ```bash
   cd sources/{project}
   git diff {old-sha}..HEAD -- docs/
   ```
3. **If no changes**, report that skills are up to date
4. **If changes exist**, read the affected docs and update:
   - Affected reference files in `skills/{project}/references/`
   - `SKILL.md` version and tables if references were added/removed
   - `GENERATION.md` with new SHA and date

## Workflow: Add New Project

1. **Add entry to `meta.ts`** in the `submodules` object
2. **Run `bun start init -y`** to clone the submodule
3. **Follow the "Create New Skills" workflow** above

## Quality Checklist

Before finishing, verify:
- [ ] Each reference file covers exactly one concept
- [ ] All code examples are working and practical
- [ ] No verbatim doc copying — content is synthesized for agents
- [ ] No getting-started/install guides or common knowledge included
- [ ] Reference filenames use category prefixes (`core-`, `features-`, etc.)
- [ ] SKILL.md tables link to all reference files
- [ ] GENERATION.md records the correct git SHA
- [ ] `skills/{project}/` name matches `sources/{project}/`