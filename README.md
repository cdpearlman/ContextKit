# ContextKit

A single-file bootstrap that turns any AI coding agent into one with long-term project memory.

## Motivation

Every AI conversation starts from zero. You re-explain your project, your conventions, your preferences. The agent gives generic advice because it has no context. Most developers either don't write instruction files at all, or dump everything into one massive file — wasting the agent's attention budget and degrading output quality.

ContextKit solves this by **making the agent write its own instructions** based on a conversation with the person who knows the project best.

| Principle | What it means |
|-----------|--------------|
| **Zero infrastructure** | Plain files in your repo. No database, no API, no build step |
| **Agent-agnostic** | Works with any agent that reads markdown |
| **Interview-driven** | The developer talks, the agent writes — not the other way around |
| **Progressive disclosure** | The agent loads only what it needs for the current task |
| **Self-maintaining** | The agent proposes memory updates, you approve via diff |
| **Git-native** | Every memory change is a trackable commit |

## How It Works

Drop `AGENTS.md` into your project root. Open your AI coding tool, add `AGENTS.md` to the context, and say **"set up this project."**

The agent reads the file, interviews you about your project (5-10 minutes), and generates a complete memory system:

```
AGENTS.md                          ← Becomes the project brain (routing file)
.context/
  modules/                         ← Domain-specific context (conventions, architecture, etc.)
    [agent-determined].md
  data/                            ← Accumulated knowledge (append-only)
    sessions.md                    ← Running work log
    decisions.md                   ← Decision records with reasoning
    lessons.md                     ← Hard-won knowledge
```

From that point forward, every agent session starts with full project context.

## Architecture: 3-Level Memory Hierarchy

The agent never loads everything at once. It navigates from broad routing to specific data, only loading what the current task requires.

### Level 1 — Routing (always loaded, ~100 tokens)

The routing file is the project brain: project identity, critical rules, and a map of where to find deeper context. After bootstrap, `AGENTS.md` itself *becomes* this routing file — no extra indirection. For tool-specific variants, routing content is written into the tool's native memory file instead.

| Tool | Routing file |
|------|-------------|
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursor/rules/` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Windsurf | `.windsurfrules` |
| Generic | `AGENTS.md` |

### Level 2 — Modules (loaded when relevant, ~50-200 tokens each)

Modules live in `.context/modules/` and contain instructions scoped to a specific domain. The agent decides which modules to create based on the bootstrap interview — there is no fixed list.

Writing frontend code? The agent loads the conventions module. Deploying? The workflows module. A simple CLI tool might get two modules; a complex full-stack app might get six.

### Level 3 — Data (loaded when needed)

Data files live in `.context/data/` and store institutional memory. These are **append-only** — the agent adds entries but never deletes past entries.

- **`sessions.md`** — Running work log of meaningful sessions
- **`decisions.md`** — *Why* things were decided, not just *what*
- **`lessons.md`** — Hard-won knowledge from mistakes and discoveries

The total attention cost for any task is: **routing + one or two modules + relevant data entries**. Never the entire memory system.

## The Bootstrap Interview

The bootstrap is the heart of ContextKit. When an agent reads the shipped `AGENTS.md`, it conducts a guided conversation with the project owner, then generates the entire `.context/` hierarchy and transforms `AGENTS.md` into the permanent routing file.

The interview feels like a **1-on-1 with a smart new team member** — the agent detects the AI tool in use, scans the codebase for existing context, asks broad questions and follows up on what's interesting, and summarizes what it learned before generating files. Topics covered include project purpose, tech stack, folder structure, workflow, conventions, past mistakes, and communication preferences.

## Memory Maintenance

The agent watches for moments when memory should update:

- **New conventions** emerging from your code
- **Decisions** being made during a session
- **Mistakes** worth remembering
- **Scope changes** like adding a new component or service
- **Preferences** revealed through repeated corrections

It proposes specific changes, you review the diff, and only approved changes are written. Data files are append-only — the agent can't accidentally overwrite history. The routing file is treated as high-stakes since it affects every future session.

### Skills (Claude Code)

Memory maintenance instructions embedded in the routing file can lose salience during long conversations. ContextKit ships with **skills** — workflow shortcuts that load fresh instructions on demand — to solve this:

| Skill | Trigger | What it does |
|-------|---------|-------------|
| `/checkpoint` | Auto + manual | Proposes sessions.md entry, checks for pending decisions/lessons, runs cross-file consolidation |
| `/spec` | Manual only | Generates a SPEC-*.md with problem, options, decision, and exit criteria before implementation |
| `/lessons` | Auto + manual | Reviews lessons.md for stale, contradictory, or outdated entries and proposes corrections |

**Auto-triggered** skills can be invoked by Claude when it detects relevant moments (e.g., a decision was made, a mistake was caught). **Manual** skills run only when you type the command.

Skills live in `.claude/skills/` and are customized during bootstrap. For non-Claude-Code tools, equivalent workflows are documented as natural language conventions in the routing file.

> **Migrating from `.claude/commands/`?** Your existing command files still work. To upgrade: move `commandname.md` to `.claude/skills/commandname/SKILL.md` and add YAML frontmatter. See the [Claude Code skills docs](https://code.claude.com/docs/en/skills) for details.

## Quick Start

1. Copy `AGENTS.md` into your project root
2. Open your AI coding tool and add `AGENTS.md` to the context
3. Tell the agent: **"Set up this project"**
4. Answer the interview questions (5-10 minutes)
5. Review the generated files
6. Start working — the agent now has persistent memory

For **Claude Code** users: the bootstrap also generates workflow skills in `.claude/skills/` that keep memory maintenance active even in long sessions.

## What This Is Not

- **Not a task manager** — ContextKit is context and memory, not project management
- **Not a personal OS** — it's project-scoped, not life-scoped
- **Not a database** — plain files, git-native, human-readable
- **Not opinionated about your stack** — the interview adapts to whatever you describe
- **Not agent-specific** — works with any AI coding assistant that reads markdown

## Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Bootstrap file | `AGENTS.md` | Industry standard, supported by Claude Code / Cursor / Copilot |
| Memory folder | `.context/` | Hidden folder, doesn't clutter project root |
| Routing file | Tool's native memory file | Zero indirection — the auto-discovered file IS the project brain |
| Module structure | Agent-determined | Avoids dead-weight files for projects that don't need them |
| Session logs | Single `sessions.md`, append-only | Simpler than date-stamped files |
| Update mechanism | Proactive detection + diff approval | Agent watches for update moments, user approves |
| Workflow delivery | Skills (Claude Code) / conventions (others) | Skills solve context drift — instructions load fresh on demand instead of competing with conversation |
| Interview style | Guided conversation with topic checklist | Balances structure with naturalness |

## License

MIT
