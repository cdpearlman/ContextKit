---
name: defrag
description: Use when the user asks to "defrag", "defragment", "consolidate", or "tidy up" the codebase, or after a multi-session feature where duplicate utilities, dead exports, and scattered logic are likely to have accumulated ("AI sediment"). Performs a guided cleanup pass with three tiers — safe deletions auto-execute, consolidations execute per-category with approval, structural refactors are flagged and deferred to /spec. Do NOT invoke for feature work, formatting passes, or full restructures.
disable-model-invocation: true
context: fork
agent: general-purpose
---

# Defrag

Treat the codebase like a fragmented disk. Across multi-session AI work, duplicates,
dead exports, near-clones, and orphan files accumulate. Defrag consolidates them
without changing behavior.

Goal: **less code, same behavior, all tests green.** Not a redesign.

## Scope boundary

**In scope** (this skill executes):
- Delete unused imports, unreachable code, unused locals
- Delete unused exports and unused dependencies
- Merge duplicate / near-duplicate utilities into one canonical copy
- Consolidate scattered helpers within the same top-level module
- Replace local re-implementations with imports from the canonical source
- Inline trivially-wrapping single-use functions
- Move a private helper from a shared file into its sole consumer

**Out of scope** (this skill flags as DEFER, suggests `/spec`):
- Public API changes (renames, signature changes, breaking removals)
- Moving code across top-level module boundaries (e.g. `auth/` ↔ `billing/`)
- File splits driven by size alone (often imply new API decisions)
- Restructuring class hierarchies, interface designs, or layer boundaries
- Anything touching >10 call sites across >3 top-level modules
- Anything that requires updating user-facing docs

When in doubt: defer. Defrag exists to be safe and small.

## Hard rules

1. **Never operate on `main`/`master`.** Auto-create `defrag/<YYYY-MM-DD>` and
   switch to it. If a `defrag/*` branch already exists, append `-2`, `-3`, etc.
2. **Tests must be green at start.** If red, stop and report which fail.
   If no test suite exists, require build/typecheck to pass instead.
   If neither exists, refuse to run.
3. **Working tree must be clean.** Ask the user to commit or stash first.
4. **One category per commit.** No "while I'm in here" changes.
5. **Run tests after every batch.** Revert any batch that goes red.
   For slow suites, honor `defrag.fast_test_command` from CLAUDE.md if set;
   otherwise use the standard test command.
6. **Tool evidence, not intuition.** Use the project's real analyzers. If a needed
   tool isn't installed, skip the category and note it in the plan — never invent
   findings from grep alone.
7. **Log everything** to `docs/DEFRAG_LOG.md` (or `DEFRAG_LOG.md` at repo root if
   `docs/` doesn't exist).
8. **Stop and ask** before any single batch >50 LOC or >5 files.

## Workflow

### Phase 1 — Preflight
- Check current branch; auto-create and switch to `defrag/<YYYY-MM-DD>`.
- Verify clean working tree. If dirty, ask user to commit/stash.
- Read CLAUDE.md / AGENTS.md / package manifests for test, build, lint commands
  and any `defrag.*` overrides.
- Run the test (or build) command. Stop if red.
- Ensure log file exists.

### Phase 2 — Inventory (parallel, read-only)
Spawn parallel scans. Every finding cites `file:line` and the tool that flagged it.

Categories, safest → least safe:
1. Unused imports
2. Unreachable code (after unconditional return/throw, permanently-on/off flags)
3. Unused exports (zero internal references, no public-API annotation)
4. Unused dependencies (in manifest, not imported anywhere)
5. Duplicate utilities (near-identical functions across files)
6. Scattered helpers within a single top-level module
7. Trivially-wrapping single-use functions
8. Stale TODOs/FIXMEs (>6 months, no linked issue) — report only, never delete

Tooling examples (use whatever the project actually has):
- JS/TS: `knip`, `ts-prune`, `depcheck`, `eslint`, `jscpd` (for duplicates)
- Python: `vulture`, `ruff check --select F401,F811`, `pylint --disable=all --enable=W0611`
- Rust: `cargo +nightly udeps`, `cargo clippy`
- Go: `staticcheck`, `golang.org/x/tools/cmd/deadcode`

### Phase 3 — Categorize
Write `docs/DEFRAG_PLAN.md` (or repo-root fallback) with three tiers:

- **SAFE** — categories 1–4 above. Auto-execute in Phase 5.
- **CONSOLIDATE** — categories 5–7. One-line justification per item.
  Per-category approval before executing.
- **DEFER** — anything matching the out-of-scope list. Flag with a suggested
  `SPEC-*.md` filename. Do not execute.

For CONSOLIDATE merges, name the canonical location using this heuristic:
1. If one copy lives in `shared/`, `utils/`, `lib/`, `common/`, or equivalent —
   use it.
2. Else if one copy has materially more callers — use it.
3. Else — ask the user where to consolidate.

### Phase 4 — Approval
Present:
- N items SAFE (will auto-execute)
- N items CONSOLIDATE, grouped by category (will ask per category)
- N items DEFER (flagged only)

Wait for explicit approval. Do not proceed without it.

### Phase 5 — Execute
Per category, safest first:
1. State category and item count.
2. Apply changes.
3. Run tests (or fast-test if configured).
4. Green → commit `defrag: <category> (<N> items)` and append to log.
5. Red → revert the commit, log the failure, move to next category.
   Do **not** try to fix failing tests inside defrag — that's a separate task.

Split categories with >20 items into sub-batches by directory. For
CONSOLIDATE merges, do the merge and update all import sites in a single commit
so the tree is never broken between commits.

### Phase 6 — Report
Append to the log:
- Categories executed, items removed/merged per category
- Net LOC delta
- Test status
- Items deferred, with suggested `SPEC-*.md` filenames

Output the same summary to the user. Suggest the next step:
"Open a PR" or "Run `/spec` for the deferred items."

## Anti-patterns
- Mixing defrag with feature work — buries the diff, breaks the contract.
- Reformatting under the defrag banner — formatting is its own commit.
- Consolidating across module boundaries you don't fully understand — duplicated
  helpers in two services may be intentional decoupling. Defer.
- Trusting grep over real analyzers. Reflection, dynamic dispatch, and
  string-built identifiers will fool grep.
- Skipping tests "just for the safe batch." There is no safe batch.
- Picking a canonical location by aesthetic preference. Use the heuristic, or ask.

## Output artifacts
- `docs/DEFRAG_PLAN.md` (Phase 3)
- `docs/DEFRAG_LOG.md` (appended every run)
- One commit per executed category (Phase 5)
- A final summary to the user

Defrag does not modify behavior, add features, change public APIs, or open PRs.