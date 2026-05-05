---
name: spec
description: Generate a SPEC-*.md before implementing a non-trivial task. Produces a structured spec with problem framing, genuine alternatives with tradeoffs, decision and rationale, file-level implementation plan, and verifiable exit criteria. Use for any work beyond a small bug fix, copy change, or single-file tweak.
argument-hint: [task description or feature name]
disable-model-invocation: true
---

# Spec Generation

Generate a specification document before implementing a non-trivial task. The spec is a contract: the user approves it, and implementation must satisfy every exit criterion before being considered done.

## Steps

### 1. Parse the task

Read `$ARGUMENTS` as the task description.

- If `$ARGUMENTS` is empty, ask the user what they want a spec for and stop until they answer.
- If the task is vague (e.g. "improve the API"), ask 1–3 targeted clarifying questions before continuing. Don't guess at intent.

### 2. Load relevant project context

Before drafting, scan for context that should inform the spec:

- The project routing file's Module Map — identify which modules apply to this task and read them.
- `.context/data/decisions.md` — check for prior decisions in this area. If a prior decision constrains the options, surface it. If a prior decision contradicts what you're about to propose, flag the conflict explicitly in the spec.
- `.context/data/lessons.md` — check for lessons that apply. If a lesson rules out a pattern you'd otherwise consider, Options Considered must acknowledge this rather than re-proposing it.
- The existing codebase for files the implementation will likely touch.

This step is not optional. Specs that skip it recreate prior decisions, sometimes wrongly.

### 3. Scope check

If the task is genuinely trivial — single-line fix, typo, dependency bump, formatting change — say so and ask whether a spec is actually wanted. Specs have a cost; not every change earns one.

### 4. Generate the spec

<!-- BOOTSTRAP_CUSTOMIZE: spec_location -->
Create the spec file in the project root as `SPEC-[feature-name].md`.
<!-- END_BOOTSTRAP_CUSTOMIZE -->

Use kebab-case for the feature name: `SPEC-oauth-login.md`, not `SPEC-OAuthLogin.md` or `SPEC-oauth_login.md`.

Use this format:

````markdown
# SPEC: [Feature Name]

## Problem
What we're solving and why. Include user impact or technical motivation. If the problem statement isn't crisp, the spec isn't ready.

## Options Considered
Two or three genuine alternatives — not strawmen. Each option must be one a reasonable person could pick.

### Option A: [Name]
- **Approach**: How it works
- **Pros**: What's good
- **Cons**: What's bad

### Option B: [Name]
- **Approach**: How it works
- **Pros**: What's good
- **Cons**: What's bad

## Decision
Which option we're picking and *why the others were rejected*. "Option B" is not a decision; "Option B because A would couple us to X and C requires infrastructure we don't have" is.

## Implementation Plan
Step-by-step with file-level specifics:
1. [Step] — `path/to/file.ext`
2. [Step] — `path/to/file.ext`

## Exit Criteria
Concrete, verifiable conditions that must be true for completion:
- [ ] [Specific test that passes, behavior that works, or output produced]
- [ ] [Specific test that passes, behavior that works, or output produced]

Each criterion must be independently checkable. "Works correctly" is not an exit criterion. "Login redirects to /dashboard on success and shows error on 401" is.

## Out of Scope
What we explicitly are NOT doing in this spec, and why. This section prevents scope creep during implementation.

## Open Questions
Anything that needs an answer before or during implementation. Empty is fine — but don't fill it with fake questions to look thorough.
````

### 5. Stop and wait for approval

Present the spec and stop. Do not begin implementation. Do not read additional files in preparation for implementation. Do not draft code "in case it's helpful." Do not start a TODO list of next steps.

The user will approve, request changes, or reject the spec. Only after explicit approval does implementation begin — and even then, deviations from the spec require asking, not assuming.

## Gotchas

- **Pseudo-decisions**: "We'll use the best approach" or "TBD" is not a decision. If you can't make the call, surface it as an Open Question.
- **Strawman options**: If Option B exists only to make Option A look good, the deliberation isn't real. Generate genuine alternatives or acknowledge there's only one viable path.
- **Aspirational exit criteria**: "The code is clean" is not testable. "Linter passes with zero warnings" is.
- **Scope creep in implementation plan**: If steps include work not required by the Problem statement, move it to Out of Scope or cut it.
- **Premature implementation**: The strongest pull on the model after generating a spec is to start building. Resist this. The spec is the deliverable until the user approves it.