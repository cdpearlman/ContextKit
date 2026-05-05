---
name: checkpoint
description: >
  Use at the end of a substantive work session, when meaningful decisions
  have been made, when mistakes or surprising behaviors were caught, or
  when the user says "checkpoint", "save progress", "wrap up", or "update
  memory". Proposes appended entries to decisions.md, lessons.md, and
  sessions.md (when present), runs a light staleness check on lessons
  touched by today's work, and flags promotion or consolidation
  opportunities. Do NOT invoke for trivial chats, single-question Q&A,
  read-only exploration, or before any substantive work has happened.
---

# Checkpoint

Append-only memory update for ContextKit. Captures what was decided, what
was learned, and (when applicable) what was done — then waits for approval
before writing anything.

This skill is **append-and-flag**, not consolidate. Heavy reorganization is
not in scope here. If thresholds are crossed (entry counts, contradictions,
module promotion), this skill *flags* them; it does not execute the
restructuring inline.

## When to invoke

- End of a session with substantive work
- After a decision (tool choice, architecture pick, pattern adoption, scope change)
- After a mistake was caught, an approach was abandoned, or a surprise was discovered
- User says "checkpoint" / "save progress" / "wrap up" / "update memory"

## When NOT to invoke

- Read-only exploration, Q&A, or short clarifying chats
- The user is mid-task and hasn't reached a stopping point
- Nothing has actually changed since the last checkpoint
- The user explicitly asks for code changes only ("don't bother with memory")

## Steps

### 1. Read current state

Read whatever's present:
- The routing file (`CLAUDE.md`, `.cursor/rules/contextkit.mdc`, `AGENTS.md`,
  or tool equivalent) — Critical Rules and Module Map
- `.context/data/decisions.md`
- `.context/data/lessons.md`
- `.context/data/sessions.md` — **only if it exists**. In Claude Code and
  Codex projects this file is intentionally absent (native auto-memory
  covers it). Do not create it.
- Any module files relevant to today's work

If `decisions.md` or `lessons.md` is missing, stop and ask the user — these
should always be present in a ContextKit project. Likely the routing file
points elsewhere or bootstrap was incomplete.

### 2. Propose decisions.md entries

For each decision made this session (tool choice, architecture pick, pattern
adopted, scope change, anything with "options considered"), propose:

```markdown
## [Decision title]
**Date**: [today's date]
**Context**: What prompted this decision
**Options considered**: Alternatives and why they were ruled out
**Decision**: What was chosen
**Reasoning**: Why
**Revisit if**: Conditions that would warrant reconsidering
```

Skip this step if no decisions were made. Trivial preference picks ("use
this variable name") are not decisions.

### 3. Propose lessons.md entries

For each mistake caught, surprise found, or approach abandoned, propose:

```markdown
## [today's date] — [Brief title]
**What happened**: What went wrong or what was discovered
**Root cause**: Why it happened
**Fix**: What was done about it
**Rule going forward**: What to do (or avoid) in the future
**What was ruled out**: Approaches considered and rejected, and why
```

Skip if nothing was learned the hard way. Routine successes are not lessons.

### 4. Propose sessions.md entry — *only if the file exists*

If `.context/data/sessions.md` is not present, **skip this step entirely**.

If it is present, append:

```markdown
## [today's date] — [Brief title]
**Area**: What part of the project was touched
**Work done**: Concise summary of meaningful work
**Decisions made**: Reference any decisions from step 2, or "None"
**Memory created**: New modules / data entries proposed this checkpoint, or "None"
**Open threads**: Unfinished work, unanswered questions, or "None"
```

<!-- BOOTSTRAP_CUSTOMIZE: reporting_style -->
Use neutral reporting — state what changed, what failed, and what's
uncertain. Avoid framing routine work as accomplishments.
<!-- END_BOOTSTRAP_CUSTOMIZE -->

### 5. Light lesson staleness check

For lessons that intersect today's work (same files, same domain, same
patterns), check:

- Did today's work follow the "Rule going forward"? If it silently didn't,
  flag the lesson for the user — either the lesson is wrong, or today's
  work is.
- Did today's decisions in step 2 contradict or supersede an existing
  lesson? If so, propose an inline update beneath the original:

```markdown
  [YYYY-MM-DD UPDATE]: [What changed and why]
```

This is a *narrow* check on lessons relevant to today's work. Do not audit
the full file — that's a heavier review the user runs deliberately.

### 6. Flag promotion and consolidation opportunities

Surface these as questions, not auto-edits:

- **Module promotion**: Does any domain now have 3+ decisions or lessons
  that justify a dedicated module? If so, propose a new module name and
  outline.
- **Critical Rules promotion**: Has a lesson been hit repeatedly, or is a
  decision now load-bearing enough that it belongs in the routing file's
  Critical Rules section? If so, propose the line.
- **Sessions.md consolidation**: If `sessions.md` exists and has ~30+
  entries, propose summarizing the oldest 20 into a single dated summary
  block (preserving key decisions and open threads) and replacing those
  entries with the summary.
- **Lessons consolidation**: If `lessons.md` has ~30+ entries, or two
  entries touched today's work and contradict each other, flag for
  consolidation. Do not consolidate inline — propose it as a follow-up the
  user can approve in a focused pass.

### 7. Present everything as one diff

Bundle every proposed change into a single review:

- State which files would change
- Show each diff (new entry, inline update, flag)
- Group flags separately from concrete edits

Wait for explicit approval before writing. The user can approve all, approve
selectively, edit inline, or deny. Never write silently. Never split the
review into multiple turns unless the user asks.

## Hard rules

1. **Append-only for the data files.** Never edit past entries in
   `decisions.md`, `lessons.md`, or `sessions.md`. Updates go inline beneath
   the original with `[YYYY-MM-DD UPDATE]:` prefix.
2. **Never create `sessions.md`** if it doesn't exist. Its absence is
   intentional in tools with native auto-memory.
3. **Routing file changes are high-stakes.** Propose Critical Rules
   additions explicitly, and prefer to defer them rather than push them.
4. **No silent writes.** Always present the diff, always wait for approval.
5. **One checkpoint, one review.** Don't drip-feed multiple checkpoint
   passes in a session unless the user explicitly asks.