# ContextKit

A single-file bootstrap that turns any AI coding agent into one with long-term project memory.

## The Problem

Every AI conversation starts from zero. You re-explain your project, your conventions, your preferences. The agent gives generic advice because it has no context. Some developers write detailed instruction files; most don't have the time.

## The Solution

Drop `AGENTS.md` into your project root. Open your AI coding tool. Say **"set up this project."**

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

From that point forward, every agent session starts with full project context. The memory system self-maintains — the agent proactively suggests updates when it detects new patterns, decisions, or lessons, and you approve changes via diff.

## How It Works

### 3-Level Progressive Disclosure

The agent never loads everything at once. It follows a hierarchy:

1. **Routing file** (~100 tokens) — Always loaded. Project identity, critical rules, and a map of where to find deeper context.
2. **Modules** (~50-200 tokens each) — Loaded when relevant. Writing frontend code? Load the conventions module. Deploying? Load the workflows module.
3. **Data** (variable) — Loaded when needed. Recent session logs, related decisions, relevant lessons.

This keeps the agent's attention focused on what matters for the current task.

### Agent-Agnostic

ContextKit works with any AI coding tool that reads markdown. During bootstrap, the agent detects your tool and writes the routing file in the right place:

| Tool | Routing file location |
|------|----------------------|
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursor/rules/` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Windsurf | `.windsurfrules` |
| Generic | `AGENTS.md` |

### Self-Maintaining Memory

The agent watches for moments when memory should update:
- New conventions emerging from your code
- Architectural decisions being made
- Mistakes worth remembering
- Project scope changes

It proposes specific changes, you review the diff, and only approved changes are written. Data files (sessions, decisions, lessons) are append-only — the agent can't accidentally overwrite history.

## Quick Start

1. Copy `AGENTS.md` into your project root
2. Open your AI coding tool
3. Tell the agent: **"Set up this project"**
4. Answer the interview questions (5-10 minutes)
5. Review the generated files
6. Start working — the agent now has persistent memory

## What This Is Not

- **Not a task manager** — ContextKit is context and memory, not project management
- **Not a personal OS** — it's project-scoped, not life-scoped
- **Not a database** — plain files, git-native, human-readable
- **Not opinionated about your stack** — the interview adapts to whatever you describe

## Design

See [SPEC.md](SPEC.md) for the full design specification, architecture decisions, and rationale.

## License

MIT
