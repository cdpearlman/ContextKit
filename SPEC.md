# ContextKit — Spec v2.0

## Goal

ContextKit is a single-file bootstrap that creates a structured, self-maintaining memory system for any AI coding agent. It replaces the blank-slate problem — where every agent conversation starts from zero — with an interview-driven onboarding that builds a persistent project brain.

**The deliverable**: One file (`AGENTS.md`) that, when read by any AI coding agent, conducts an onboarding interview with the project owner and generates a complete `.context/` memory hierarchy. After bootstrap, `AGENTS.md` itself becomes the Level 1 routing file — the persistent project brain that points the agent into `.context/` for everything else. No extra indirection. For tool-specific variants, the routing content can be written into the tool's native memory file instead (e.g., `CLAUDE.md` for Claude Code, `.cursor/rules/` for Cursor).

---

## Why This Exists

Most developers either:
- Spend significant time manually writing and maintaining agent instruction files
- Don't write them at all — the agent starts blind every session
- Dump everything into one massive file — wasting the agent's attention budget and degrading output quality

ContextKit solves this by **making the agent write its own instructions** based on a conversation with the person who knows the project best.

### Design Principles

| Principle | What it means |
|-----------|--------------|
| **Zero infrastructure** | Plain files in your repo. No database, no API, no build step |
| **Agent-agnostic** | Works with any agent that reads markdown (Claude Code, Cursor, Copilot, Gemini, etc.) |
| **Interview-driven** | The developer talks, the agent writes — not the other way around |
| **Progressive disclosure** | The agent loads only what it needs for the current task |
| **Self-maintaining** | The agent proactively detects when memory should update, proposes changes, and the developer approves via diff |
| **Git-native** | Every memory change is a trackable commit |

---

## Architecture: The 3-Level Memory Hierarchy

Inspired by how expert systems and human memory work, ContextKit uses three levels of progressive disclosure. The agent never loads all memory at once — it navigates from broad routing to specific data, only loading what the current task requires.

```
Level 1 — Routing (always loaded)
  AGENTS.md (or CLAUDE.md, .cursor/rules, etc.)
    └── Project identity, critical rules, module map
    └── ~100-150 lines max — fits in any agent's attention sweet spot
    └── IS the tool's native memory file — no extra layer of indirection

Level 2 — Modules (loaded when relevant)
  .context/modules/
    └── architecture.md, conventions.md, workflows.md, ...
    └── Each module is a self-contained instruction file for a domain
    └── Agent reads the module map in Level 1 and loads only what matches the task

Level 3 — Data (loaded only when task requires)
  .context/data/
    └── sessions.md, decisions.md, lessons.md, ...
    └── Append-only logs, records, and accumulated knowledge
    └── Agent reads specific entries, not entire files when possible
```

### Why This Hierarchy Works

- **Level 1** costs ~100 tokens. The agent always knows what the project is and where to find deeper information. This is the "map."
- **Level 2** costs ~50-200 tokens per module. When writing frontend code, the agent loads the conventions module — not the database module. This is "domain context."
- **Level 3** is variable. Session logs and decision records grow over time. The agent reads what's relevant — recent sessions, decisions related to the current area. This is "institutional knowledge."

The total attention cost for any single task is: **routing + one or two modules + relevant data entries**. Never the entire memory system.

---

## Level 1: The Routing File

After bootstrap, the routing file IS the tool's native memory file — the same file the agent auto-discovers at the start of every session:

| Tool | Routing file becomes |
|------|---------------------|
| Generic / AGENTS.md standard | `AGENTS.md` at project root |
| Claude Code | `CLAUDE.md` at project root |
| Cursor | `.cursor/rules/` (always-apply rule) |
| Windsurf | `.windsurfrules` |
| Other | Whatever the tool reads on startup |

The bootstrap interview includes a tool detection step — the agent asks which tool is in use (or detects it from existing config files) and writes the routing content into the correct native file. Different branches of ContextKit can also ship pre-configured for a specific tool.

This is the single most important file. It must stay lean — under 150 lines — because everything here competes for the agent's attention on every single task.

### What Belongs Here

- **Project identity**: Name, one-line description, tech stack summary
- **Critical rules**: The 5-10 most important things the agent must *always* know (e.g., "never modify the auth module without explicit approval", "all API responses use the Result type")
- **Module map**: A directory of available modules with one-line descriptions and file paths — this is how the agent knows what to load for a given task
- **Data map**: A directory of data files with descriptions
- **Agent behavior preferences**: How the user wants the agent to work (ask before large refactors, preferred communication style, etc.)
- **Memory maintenance rules**: Instructions for how and when to propose memory updates

### What Does NOT Belong Here

- Detailed conventions (put in a module)
- Architecture deep-dives (put in a module)
- Historical records (put in data files)
- Anything the agent doesn't need for *every* task

---

## Level 2: Modules — Domain-Specific Context

Modules live in `.context/modules/` and contain instructions scoped to a specific area of the project. **The agent decides which modules to create based on what it learns during the bootstrap interview.** There is no fixed list.

### Example Module Categories

These are *examples* from real projects — not requirements. The agent should create whatever modules make sense for the project it's onboarding.

| Category | What it might contain | When the agent would load it |
|----------|----------------------|----------------------------|
| Architecture | System design, data flow, infrastructure patterns, key architectural decisions | Working on structural changes, new features touching multiple components |
| Conventions | Naming, code style, patterns to use/avoid, import ordering | Writing or reviewing any code |
| Workflows | Git strategy, commit format, PR process, deployment, CI/CD | Committing, branching, deploying |
| Testing | Framework, coverage expectations, mocking patterns, test structure | Writing or modifying tests |
| Domain-specific | Could be "frontend", "API", "database", "ML pipeline", or whatever the project needs | Working in that domain |

### Module Format

Each module is a plain markdown file. No special frontmatter required — any agent can read it. The module should be self-contained: an agent loading *only* the routing file + one module should have enough context to work effectively in that domain.

A module might look like:

```markdown
# Conventions

## Naming
- Components: PascalCase (e.g., `UserProfile`)
- Utilities: camelCase (e.g., `formatDate`)
- Constants: UPPER_SNAKE (e.g., `MAX_RETRIES`)
...

## Patterns
- Prefer composition over inheritance
- All async functions return Result<T, Error>, never throw
...

## Anti-patterns
- No `any` in TypeScript — use `unknown` if type is truly unknown
- No barrel exports (index.ts re-exports) — import directly from source
...
```

---

## Level 3: Data — Accumulated Knowledge

Data files live in `.context/data/` and store the project's institutional memory. These files grow over time and are **append-only** — the agent adds entries but never deletes or overwrites past entries. This prevents accidental data loss, which is a well-documented failure mode with LLM agents.

### Core Data Files

These are created during bootstrap and maintained throughout the project's life:

#### `sessions.md` — Running Work Log
An append-only log of meaningful work sessions. The agent appends a summary when it detects it did substantive work (not just answering a question).

```markdown
## 2026-03-01
**Area**: Authentication module
**Work done**: Implemented OAuth2 PKCE flow, added token refresh logic
**Decisions made**: Chose PKCE over implicit flow for security
**Memory updated**: Added auth patterns to conventions module
**Open threads**: Token revocation endpoint still needs implementation
```

#### `decisions.md` — Decision Record 
Captures *why* things were decided, not just *what*. This is where the agent gets smarter about the project's reasoning over time.

```markdown
## Chose PostgreSQL over MongoDB
**Date**: 2026-02-15
**Context**: Needed a database for user data and activity logs
**Options considered**: PostgreSQL (relational, strong consistency), MongoDB (flexible schema, simpler for early stage)
**Decision**: PostgreSQL
**Reasoning**: Our data model is inherently relational (users → teams → projects). Schema flexibility isn't worth giving up joins and constraints.
**Revisit if**: Schema changes become extremely frequent during early prototyping
```

#### `lessons.md` — Accumulated Wisdom
What the team learned the hard way. This is the single most valuable long-term file — it encodes judgment, not just facts.

```markdown
## 2026-02-20 — API error handling
**What happened**: Returned raw database errors to the client, leaked table names
**Root cause**: No error sanitization layer between service and controller
**Fix**: Added ErrorMapper in middleware that maps internal errors to client-safe responses
**Rule going forward**: Never return raw errors from any layer below the controller
```

The agent references these files when encountering similar situations, producing answers grounded in the project's actual history rather than generic advice.

---

## The Bootstrap Interview

The bootstrap is the heart of ContextKit. It's embedded directly in the `AGENTS.md` file that ships with the project. When an agent reads it, it conducts a guided conversation with the project owner, then generates the entire `.context/` hierarchy and transforms `AGENTS.md` itself (or the tool's native memory file) into the permanent routing file.

### Interview Philosophy

The interview should feel like a **1-on-1 with a smart new team member**, not a form to fill out. The agent:

- Detects which AI tool is in use (or asks) so it knows where to write the routing file
- Detects whether this is a new or existing project
- For existing projects: scans the codebase first (package files, README, directory structure, existing agent config) and reports what it found — then only asks about gaps
- Asks broad questions and follows up based on what's interesting, complex, or ambiguous
- Has a mental checklist of topics to cover, but doesn't march through them robotically
- Summarizes what it learned and asks "did I miss anything?" before generating files

### Topics to Cover

The agent should aim to understand these areas, adapting its questions based on the project:

- **What is this?** — Project purpose, who it's for, what problem it solves
- **What's it built with?** — Languages, frameworks, key dependencies
- **How is it organized?** — Folder structure, architectural patterns, key boundaries
- **How do you work?** — Solo or team, git workflow, deployment, CI/CD
- **What are the rules?** — Naming conventions, code patterns, things that are always or never done
- **What's been tried?** — Past mistakes, abandoned approaches, hard-won knowledge
- **How should I behave?** — Communication preferences, when to ask vs. proceed, risk tolerance

The agent should go deeper on topics where the user has strong opinions or where the project has unusual complexity. It should move quickly through areas where the user gives short answers or says "standard."

### Bootstrap Output

After the interview, the agent generates:

1. **Routing file** — Overwrites `AGENTS.md` (or writes `CLAUDE.md`, `.cursor/rules/`, etc.) with the real routing content — project identity, critical rules, module map, memory maintenance protocol
2. **`.context/modules/`** — Whatever modules are needed based on what it learned
3. **`.context/data/sessions.md`** — Initialized with the bootstrap session as the first entry
4. **`.context/data/decisions.md`** — Seeded with any decisions discussed during the interview
5. **`.context/data/lessons.md`** — Seeded with any lessons/mistakes mentioned during the interview

The agent decides the module structure. A simple CLI tool might get `conventions.md` and `workflows.md`. A complex full-stack app might get `architecture.md`, `conventions.md`, `frontend.md`, `api.md`, `database.md`, and `testing.md`. The bootstrap interview determines the shape of the output.

> **Key**: The bootstrap file doesn't become a dead pointer — it *becomes* the routing file. The same file the tool auto-discovers on every session now contains the project brain.

---

## Memory Maintenance

Memory systems are only useful if they stay current. ContextKit uses **proactive detection with user approval** — the agent watches for moments when memory should change, proposes specific updates, and the user approves via diff.

### When to Propose Updates

The agent should look for update opportunities:

- **New patterns adopted**: "We've been using this error handling pattern for the last 3 files — should I add it to conventions?"
- **Decisions made**: "We just decided to switch from REST to GraphQL — should I record this in decisions.md and update the architecture module?"
- **Mistakes caught**: "This is the second time we've hit this race condition — should I add it to lessons.md?"
- **Scope changes**: "We added a mobile app — should I create a mobile module?"
- **Preferences revealed**: "You've corrected my naming three times in the same way — should I add this convention?"

### How Updates Work

1. The agent detects that memory should change
2. It states **which file(s)** would change and **what the change would be**
3. The user sees the proposed change (as a diff in their editor) and approves, edits, or denies
4. Only approved changes are written

### Rules for Memory Updates

- **Never update mid-task** without mentioning it — finish the current work, then propose the update
- **Routing file changes are high-stakes** — the routing file affects every future session, so changes here get extra scrutiny
- **Data files are append-only** — sessions.md, decisions.md, lessons.md grow but never lose entries
- **Modules can be edited** — conventions change, architecture evolves, but changes should be surgical (targeted edits, not full rewrites)

---

## What Gets Shipped

The entire ContextKit deliverable is a single file:

```
AGENTS.md    ← Contains the bootstrap interview instructions
```

That's it. The user drops this file into their project root, opens their AI coding agent, and says "set up this project." The agent reads `AGENTS.md`, conducts the interview, and generates:

```
AGENTS.md                          ← Level 1: IS the routing file (project brain)
  (or CLAUDE.md / .cursor/rules    ← depending on detected tool)
.context/
  modules/                         ← Level 2: domain-specific context
    [agent-determined].md
    [agent-determined].md
    ...
  data/                            ← Level 3: accumulated knowledge
    sessions.md
    decisions.md
    lessons.md
```

After bootstrap, `AGENTS.md` is no longer the bootstrap — it *is* the project brain:

```markdown
# Project Name
One-line description. Built with [stack].

## Critical Rules
- Rule 1
- Rule 2
...

## Module Map
| Module | Path | Load when |
|--------|------|----------|
| Conventions | .context/modules/conventions.md | Writing or reviewing code |
| Architecture | .context/modules/architecture.md | Structural changes |
| ... | ... | ... |

## Data Files
| File | Path | Purpose |
|------|------|--------|
| Sessions | .context/data/sessions.md | Running work log |
| Decisions | .context/data/decisions.md | Decision records with reasoning |
| Lessons | .context/data/lessons.md | Hard-won knowledge |

## Memory Maintenance
[Protocol for proactive detection and update proposals]

## Preferences
[How the user wants the agent to behave]
```

---

## What This Is Not

- **Not a task manager** — ContextKit is context and memory, not project management
- **Not a personal OS** — it's project-scoped, not life-scoped
- **Not a database** — plain files, git-native, human-readable
- **Not opinionated about your stack** — the interview adapts to whatever you describe
- **Not agent-specific** — works with any AI coding assistant that reads markdown

---



## Appendix: Design Decisions Made

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Bootstrap file | `AGENTS.md` | Industry standard, supported by Claude Code / Cursor / Copilot |
| Memory folder | `.context/` | Hidden folder, doesn't clutter project root |
| Routing file | The tool's native memory file (`AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, etc.) | Zero indirection — the file the tool auto-discovers IS the project brain |
| Tool detection | Asked or inferred during bootstrap | Determines which native file receives the routing content |
| Module structure | Suggested categories, agent decides | Avoids dead-weight files for projects that don't need them |
| Session logs | Single `sessions.md`, append-only | Simpler than date-stamped files, agent appends when substantive work is done |
| Update mechanism | Proactive detection + diff approval | Agent watches for update moments, user approves via editor diff |
| Bootstrap | Single `AGENTS.md` file | Maximum portability — one file to drop into any project |
| Interview style | Guided conversation with topic checklist | Balance between structure (covers all bases) and naturalness (follows the user's lead) |
