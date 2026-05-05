# ContextKit Bootstrap

You are setting up ContextKit — a structured memory system and skill catalog for this project. Run this bootstrap exactly once. Do not write code or perform other tasks until bootstrap is complete.

After bootstrap, this file (or the tool's native equivalent) becomes the permanent project routing file — loaded at the start of every session.

ContextKit ships with four skills already drafted in `.claude/skills/`. Bootstrap does not generate skill content; it **customizes** the shipped templates by filling in `BOOTSTRAP_CUSTOMIZE` blocks based on what the user tells you in the interview. For tools without first-class skill support, the same workflows are documented in the routing file as named conventions.

---

## Phase 1 — Detect Environment

### Tool Detection

Determine which AI coding tool is running this session. Check for existing configuration:

| Signal | Tool |
|--------|------|
| `CLAUDE.md` or `.claude/` | Claude Code |
| `AGENTS.md` (with `[memories]` section, or alongside `~/.codex/`) | Codex |
| `.cursor/rules/` | Cursor |
| `.windsurf/rules/` or `.windsurfrules` | Windsurf |
| `.github/copilot-instructions.md` or `.github/instructions/` | GitHub Copilot |
| None of the above | Generic / `AGENTS.md` |

If unclear, ask:
> "Which AI coding tool are you using? (Claude Code, Codex, Cursor, Copilot, Windsurf, or other)"

This determines where the routing file gets written and which skill format applies:

| Tool | Routing file | Skill format |
|------|--------------|--------------|
| Claude Code | `CLAUDE.md` (project root) | `.claude/skills/<name>/SKILL.md` (auto-invocable when frontmatter allows) |
| Codex | `AGENTS.md` (project root) | Documented as named workflow conventions in the routing file |
| Cursor | `.cursor/rules/contextkit.mdc` (`alwaysApply: true`) | Documented as named workflow conventions |
| Windsurf | `.windsurf/rules/contextkit.md` (no frontmatter; activation set in UI) | Documented as named workflow conventions |
| Copilot | `.github/copilot-instructions.md` | Documented as named workflow conventions |
| Generic | `AGENTS.md` (overwrite this file) | Documented as named workflow conventions |

**Tools with native auto-memory** — Claude Code (CLAUDE.md memory v2.1.59+) and Codex (`[memories]` opt-in table). For these tools, **do not create `sessions.md`**; the native feature covers it. For all other tools, create `sessions.md` as part of the data hierarchy.

### Project Detection

Determine whether this is a **new project** (greenfield) or an **existing project** (brownfield).

**If existing**: Before asking questions, scan the codebase to build context:
- Package/dependency files (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.)
- `README.md`
- Directory tree (top 2 levels)
- Any existing agent configuration files
- Key configuration files

Then tell the user what you found and what you still need to ask. **Skip questions you can answer from the codebase. Only ask what you can't infer.**

**If new**: Proceed directly to the interview.

---

## Phase 2 — Interview

Conduct this as a **conversation, not a questionnaire**. Ask broad questions, follow up on what's interesting or complex, and move quickly through areas where the user gives short answers. You have a checklist of topics to cover, but you don't march through them robotically.

### Topics to Cover

Work through these naturally, adapting order and depth based on the conversation.

**Project identity**
> "What is this project? One sentence — what it does and who it's for."

Follow up if the project is complex: what problem does it solve, what does success look like, who are the users.

**Tech stack**
> "What's the stack? Just the load-bearing stuff — language, framework, key libraries."

Follow up on: libraries deliberately avoided, version-specific gotchas, anything being migrated.

**Project structure**
> "Walk me through how the codebase is organized. What are the main areas and what lives in each?"

For brownfield: "I see [inferred structure]. Does this match how you think about it? Anything I'm missing?"

Follow up on: unusual architectural patterns, fragile areas needing extra care, where shared code lives.

**Workflow**
> "How do you work? Solo or team? What's the git workflow, deployment setup?"

Follow up if team: who owns what, who reviews what, how are architecture decisions made.

**Conventions & rules**
> "Any strong conventions? Naming patterns, things you always do or never do, patterns to prefer or avoid."

Follow up on: error handling, logging approach, patterns learned the hard way to avoid.

**Testing**
> "What's the testing setup, if any? Framework, what gets tested, any expectations on coverage?"

If the user opts into the `/defrag` skill later, also ask:
> "Defrag runs the test suite after every batch of changes. If your full suite is slow, do you have a faster command (a smoke subset, a single package, etc.) it should use instead?"

Record this as `defrag.fast_test_command` in the routing file's Skill Configuration section.

**Essential commands**
> "What are the essential commands? Install, dev, build, test — anything I'll need regularly."

**Past experience**
> "Anything you've learned the hard way on this project? Mistakes made, abandoned approaches, things that look tempting but don't work?"

These seed `lessons.md`. Push for specifics — vague "be careful with X" entries don't help the agent later.

**Agent behavior**
> "How do you want me to work? When should I ask before proceeding vs. just do it? Any preferences on how I communicate?"

Follow up:
> "When I summarize a session, what tone do you prefer — neutral (what changed, what failed, what's uncertain) or accomplishment-focused (what was achieved, what's done)?"

Record the user's preference. This becomes the `reporting_style` value used to customize the `/checkpoint` skill. If they have no preference, ask which they'd like to try first.

**Spec workflow**
> "Before I write code on anything non-trivial, do you want me to draft a spec first — or dive straight into implementation? And if specs, where should they live? (project root, `docs/`, `specs/`, somewhere else)"

Two outcomes:
- **If yes**: include the Spec Workflow section in the routing file and customize the `/spec` skill with `spec_location` set to the user's choice.
- **If no**: omit the Spec Workflow section entirely, and skip installing the `/spec` skill.

**Skills & workflow shortcuts**
> "ContextKit ships four optional workflow skills. Want me to install them?
>
> - **`/checkpoint`** — append-only memory updates at session end. Auto-invocable: I'll propose it when work has accumulated. Recommended for everyone.
> - **`/spec`** — drafts a `SPEC-*.md` before non-trivial work. Manual only. Recommended if you said yes to the spec workflow.
> - **`/defrag`** — codebase cleanup pass. Removes dead code, consolidates duplicates. Manual only. Useful after multi-session feature work.
> - **`/adversarial-review`** — three-perspective board review of a spec or research doc before high-stakes commitments. Manual only. Heavyweight; skip if your work is mostly small or quickly reversible.
>
> Together they form a chain: `/spec` → optional `/adversarial-review` → implement → occasional `/defrag` → `/checkpoint`. You can install any subset."

For Claude Code: explain that auto-invocable skills (`/checkpoint`) keep their description in context so Claude knows when to suggest them; manual-only skills (`/spec`, `/defrag`, `/adversarial-review`) don't auto-fire and require explicit invocation.

For other tools: skills install as named workflow conventions in the routing file (e.g., "say `run checkpoint` to update memory").

**Wrap-up**
After covering the topics, summarize what you learned and ask: *"Did I miss anything important?"*

### Interview Guidelines

- **Go deeper** where the user has strong opinions or the project has unusual complexity.
- **Move faster** where the user gives brief answers or says "standard" / "nothing special."
- **Don't ask about areas that don't apply** — if there's no frontend, skip frontend questions; if there are no tests, skip the defrag fast-test question.
- **For brownfield projects**, frame questions as confirmation of what you inferred, not blank interrogation.

---

## Phase 3 — Generate Memory System and Customize Skills

Using everything gathered, generate the `.context/` hierarchy, the routing file, and the customized skill set. Do not leave placeholders — write real content based on what you learned. If information is missing for a section, omit that section rather than filling it with generic content.

### Step 1: Create modules

Create `.context/modules/` with whatever modules the project actually needs. Each module is a self-contained markdown file covering one domain.

Example module categories (use as inspiration, not a checklist):

| Example | What it might contain | Create it if... |
|---------|----------------------|-----------------|
| `architecture.md` | System design, data flow, component boundaries, infrastructure | Project has meaningful architectural patterns or complexity |
| `conventions.md` | Naming rules, code style, patterns to use/avoid | User described specific conventions or strong preferences |
| `workflows.md` | Git strategy, commit format, PR process, CI/CD | User described a workflow beyond the basics |
| `testing.md` | Framework, coverage expectations, mocking patterns | User described a testing setup |
| Domain-specific | Frontend patterns, API design, database conventions, ML pipeline rules | Project has distinct domains with their own conventions |

Each module should be **self-contained**: an agent loading only the routing file plus that one module should have enough context to work effectively in that domain.

### Step 2: Initialize data files

Create `.context/data/` with:

**`decisions.md`** (always) — seed with any decisions discussed during the interview:
```markdown
# Decision Record

<!-- Append new entries for new decisions. When a "Revisit if" condition is met, add
     a follow-up note inline beneath the original entry, prefixed [YYYY-MM-DD UPDATE]:.
     Never delete entries. -->
<!-- Format:
## [Decision title]
**Date**: YYYY-MM-DD
**Context**: What prompted this decision
**Options considered**: What alternatives were evaluated and why they were ruled out
**Decision**: What was chosen
**Reasoning**: Why
**Revisit if**: Conditions that would warrant reconsidering

[YYYY-MM-DD UPDATE]: Follow-up note if conditions changed or decision was revised.
-->
```

**`lessons.md`** (always) — seed with any lessons or hard-won knowledge from the interview:
```markdown
# Lessons Learned

<!-- Append new entries for new lessons. When a lesson is superseded or was wrong,
     add a correction inline beneath the original, prefixed [YYYY-MM-DD UPDATE]:.
     After ~30 entries or when entries have grown contradictory or redundant, propose
     consolidation — never consolidate silently. -->
<!-- Format:
## YYYY-MM-DD — [Brief title]
**What happened**: What went wrong or what was discovered
**Root cause**: Why it happened
**Fix**: What was done about it
**Rule going forward**: What to do (or avoid) in the future
**What was ruled out**: Approaches considered and rejected, and why

[YYYY-MM-DD UPDATE]: Correction or superseding note if this lesson was wrong or outdated.
-->
```

**`sessions.md`** — **only for tools without native auto-memory** (i.e., not Claude Code, not Codex). Initialize with the bootstrap session:
```markdown
# Session Log

<!-- Append new entries after each substantive work session. Never edit past entries.
     After ~30 entries, propose consolidation: summarize the oldest 20 into a dated
     summary block preserving key decisions and open threads, then replace those
     entries with the summary. Never consolidate silently. -->

## [today's date] — Bootstrap
**Area**: Project setup
**Work done**: Ran ContextKit bootstrap interview, generated memory system
**Decisions made**: [decisions made during interview, or "None"]
**Memory created**: [modules and data files generated]
**Open threads**: [anything flagged for follow-up, or "None"]
```

If you're in Claude Code or Codex, skip this file. The `/checkpoint` skill detects the file's absence at runtime and skips its sessions step automatically.

### Step 3: Customize the shipped skills

For each skill the user opted into, locate the shipped template in `.claude/skills/<name>/SKILL.md` and replace the `BOOTSTRAP_CUSTOMIZE` blocks. The shipped skills and their customization points:

| Skill | Customize | Source |
|-------|-----------|--------|
| `/checkpoint` | `reporting_style` block — fill with user's tone preference (neutral / accomplishment-focused / their own description) | Step under "Propose sessions.md entry" |
| `/spec` | `spec_location` block — fill with user's chosen spec directory | Step under "Generate the spec" |
| `/defrag` | None at the SKILL.md level. Set `defrag.fast_test_command` in the routing file's Skill Configuration section if the user provided one | — |
| `/adversarial-review` | None | — |

For tools without native skill support (everything except Claude Code), do not modify the skill files — instead, document the workflows as named conventions in the routing file's Skills section. The skill files remain on disk as reference but are not invoked through a slash-command interface.

### Step 4: Write the routing file

Write the routing file to the location determined in Phase 1. The routing file should stay **under 200 lines** (excluding frontmatter). Use the body template below, then wrap it with the correct format for the detected tool.

#### Routing file body

```markdown
# [Project Name]
[One-line description]. Built with [stack summary].

## Critical Rules
<!-- The 5–10 most important things the agent must ALWAYS know -->
- [Rule derived from interview]
- [Rule derived from interview]
...

## Module Map
<!-- Load ONLY the modules relevant to the current task — never load everything -->
Before starting work, scan this table to determine which modules apply. Also check
Data Files for relevant history.

| Module | Path | Load when |
|--------|------|----------|
| [Name] | .context/modules/[file].md | [Specific trigger from interview] |
...

## Data Files
<!-- Accumulated knowledge — consult relevant entries, not entire files -->

| File | Path | Purpose |
|------|------|---------|
| Decisions | .context/data/decisions.md | Decision records with reasoning |
| Lessons | .context/data/lessons.md | Hard-won knowledge and past mistakes |
[| Sessions | .context/data/sessions.md | Running work log |  ← include only if sessions.md was created]

**Append-only rules**:
- All three files are append-only. Never edit past entries. When something is
  superseded or revised, add a follow-up beneath the original prefixed with
  `[YYYY-MM-DD UPDATE]:`.
- After ~30 entries in `sessions.md` or `lessons.md`, propose consolidation
  (never consolidate silently).
- The `/checkpoint` skill (when installed) handles routine appends; for everything
  else, propose the diff and wait for approval before writing.

## Memory Maintenance

Always look for opportunities to update the memory system:
- **New patterns**: "We've been doing X consistently — should I add it to conventions?"
- **Decisions made**: "We decided Y — should I record this in decisions.md?"
- **Mistakes caught**: "This went wrong because Z — should I add it to lessons.md?"
- **Scope changes**: "The project now includes W — should I create a new module?"
- **Preferences revealed**: "You've corrected me on this pattern — should I update conventions?"

Before any memory update: state which file(s) would change, show the diff, wait
for approval. Never update memory mid-task — finish current work first. Routing
file changes are high-stakes; propose them carefully.

## Preferences
<!-- How the user wants the agent to behave — generated from interview -->
- Reporting style: [neutral / accomplishment-focused / user's custom description]
- [Other preferences from interview]
...

[## Spec Workflow  ← include only if user opted into specs]
Non-trivial work gets a spec first. Specs live in [location from interview].
Use `/spec` to generate one (Claude Code), or say "run spec" (other tools).
The skill owns the template and the step-by-step generation logic — see
`.claude/skills/spec/SKILL.md`. Implementation does not begin until the spec
is approved.

## Skills
<!-- Include only if user opted into any skills -->

[For Claude Code, list the installed skills:]
ContextKit ships skills in `.claude/skills/`. Customized during bootstrap.
- `/checkpoint` — Append-only memory updates. Auto-invokes after substantive work.
- `/spec` — Generates `SPEC-*.md` before non-trivial implementation. Manual only.
- `/defrag` — Codebase cleanup (dead code, duplicates, stale imports). Manual only.
- `/adversarial-review` — Three-perspective board review of high-stakes documents. Manual only.

[For other tools, document as named conventions:]
- "run checkpoint"        → Memory update at session end.
- "run spec"              → Generate a SPEC-*.md before implementation.
- "run defrag"            → Codebase cleanup pass with safety rails.
- "run adversarial review" → Board review of a spec or research doc.

## Skill Configuration
<!-- Only include if any skill needs project-specific config -->
- `defrag.fast_test_command`: [user's faster test command, e.g. `pnpm test:unit`]
- [Other skill-level overrides as they accumulate]
```

#### Tool-specific wrapping

The body above is the same regardless of tool. Wrap it with the right format:

**Claude Code** (`CLAUDE.md`) — plain markdown, no frontmatter required. Use `@.context/modules/<file>.md` syntax to import module files instead of inlining them. Personal/local overrides go in `CLAUDE.local.md` (auto-gitignored).

**Codex** (`AGENTS.md`) — plain markdown, no frontmatter. If the user wants Codex's native `[memories]` feature on, add a brief instruction at the top: "Codex memories are opt-in; treat memories as a recall layer, not as the source of truth. Required guidance lives in this file and in `.context/`." Document workflow shortcuts as natural-language conventions.

**Cursor** (`.cursor/rules/contextkit.mdc`) — wrap the body with this frontmatter:
```yaml
---
description: "ContextKit routing file — project memory and conventions"
alwaysApply: true
---
```
For path-scoped rules that complement the routing file, create additional `.mdc` files with `globs: <pattern>`. Note: Cursor removed Memories in v2.1.x; sessions.md is required here.

**Windsurf** (`.windsurf/rules/contextkit.md`) — plain markdown, no frontmatter. Activation modes are set in the Windsurf UI. After writing the file, instruct the user: *"Set this rule's activation mode to 'Always On' in the Windsurf Customizations panel."* Max 12,000 chars per file.

**Copilot** (`.github/copilot-instructions.md`) — plain markdown, no frontmatter. The first 4,000 characters are used for code review, so put Critical Rules near the top. For path-specific rules, create files in `.github/instructions/` with `applyTo:` frontmatter.

**Generic** (`AGENTS.md`) — plain markdown, no frontmatter. Overwrites this bootstrap file.

---

## Phase 4 — Confirm

After generating all files, report to the user:

> "Bootstrap complete. Here's what was generated:
>
> **Routing file**: [location]
> **Modules**: [list with one-line descriptions]
> **Data files**: decisions.md, lessons.md[, sessions.md]
> **Skills installed**: [list of skills the user opted into, or "none"]
>
> Review the generated files and let me know if anything needs adjusting. The memory system is now active — I'll proactively suggest updates as we work together."

If `/checkpoint` was installed, also note: *"I'll propose `/checkpoint` at the end of substantive sessions. You can also run it manually anytime."*

If `/spec` was installed, also note: *"For non-trivial work, I'll suggest running `/spec` before writing code."*