---
name: adversarial-review
description: >
  Run an adversarial board review on a spec or research document before
  high-stakes commitments. Convenes three independent reviewers — Builder,
  Stakeholder, Adversary — each with a defined lens and known biases. Reviews
  run as parallel sub-agents, then synthesize into agreement, tension, the
  question nobody asked, and a verdict with explicit severity calibration.
  Use for work that's expensive to reverse; skip for routine tasks.
disable-model-invocation: true
---

# Adversarial Board Review

Convene three independent reviewers to critically evaluate a document before
high-stakes commitments. Each operates through a distinct lens with acknowledged
biases. Tension between reviewers is the point — not a flaw.

## Usage
/adversarial-review [path to document or document name]

## When to use this skill

Run adversarial review when:
- A spec describes work with multi-day implementation cost
- A research brief will drive a commitment that's expensive to reverse
- A decision affects multiple people, systems, or downstream work
- Being wrong is materially costly (time, money, trust, momentum)

Skip adversarial review when:
- The work is small or quickly reversible (bug fixes, exploratory drafts, tweaks)
- You'd iterate fast anyway and discover problems cheaply through doing
- The spec is a one-person experiment with no downstream commitments
- The document is still early-draft and the user wants feedback, not a verdict

If unsure, ask the user before running. Adversarial review is heavy — running
it on small work buries the signal in process.

## What this skill actually delivers

Three sub-agents using the same base model are *structurally* independent
(separate contexts, separate prompts) but not *epistemically* independent —
they share training data and therefore share blind spots. Convergence between
reviewers is meaningful but not conclusive. This skill catches things a
single-pass review misses, and it forces lateral thinking that LLMs typically
skip. It does not replicate genuine adversarial review by independent humans.
The synthesis acknowledges this directly rather than claiming more than it can.

## The Board

| Reviewer | Question they ask | Lens | Known Bias |
|----------|-------------------|------|------------|
| **The Builder** | "Can this actually be built?" | Implementation reality — feasibility, edge cases, missing failure modes, scope creep, effort vs. value, technical debt | May accept complexity that should be simplified; may miss user-facing problems |
| **The Stakeholder** | "Does this matter?" | Problem-solution fit — right problem, user impact, adoption friction, missing user scenarios, misaligned priorities | May deprioritize technical constraints for user wants; may miss feasibility issues |
| **The Adversary** | "What's wrong or unsaid here?" | Precision and adversarial pressure — ambiguous requirements, pseudo-decisions, vague exit criteria, internal contradictions, hidden assumptions, steelmanned counter-arguments, cognitive biases | May challenge for the sake of challenging; must remain constructive |

The Adversary absorbs both spec-precision concerns (is this written clearly
enough to be unambiguously implementable?) and devil's-advocate concerns (what
assumptions are we making that could sink this?). Both are flavors of "what's
wrong with what's on the page or what's missing from it."

## Steps

### 1. Load the document and context

Read `$ARGUMENTS` as the document path or name. If a name is given without a
path, search for it in the project directories.

Also load:
- The relevant project overview for constraints
- `.context/data/lessons.md` for past mistakes that could recur

Detect the document type:
- **Spec**: filename matches `SPEC-*` or content has spec structure (Problem, Options, Decision, Exit Criteria)
- **Research brief**: filename matches `RESEARCH-*` or content has research structure (Context, Unknowns, Findings, Recommendation)

### 2. Adapt reviewer focus to document type

**For specs**, each reviewer focuses on:

- **The Builder**: Can this be built as described? What happens at boundaries — empty inputs, max scale, concurrent access, error states? Are failure modes addressed? Does the implementation plan introduce work beyond what the problem statement requires? Are there "while we're at it" additions?
- **The Stakeholder**: Does the Problem section accurately describe a real user need? Would the proposed solution actually address it? Are user scenarios missing? Would someone outside the team understand why this matters?
- **The Adversary**: Are requirements open to multiple interpretations? Could two developers read this and build different things? Are there pseudo-decisions ("we'll use the best approach", "TBD", "as appropriate")? Can each exit criterion be independently verified? Does the document contradict itself or known lessons? What does it assume without evidence? What's the strongest argument for *not* doing this? What cognitive biases might have shaped this spec?

**For research briefs**, each reviewer focuses on:

- **The Builder**: Are the proposed options actually buildable within stated constraints? Are effort estimates grounded in evidence? Are there technical constraints missing from the evaluation criteria? Could any option fail in ways not discussed?
- **The Stakeholder**: Does this research address the right question? Are user needs represented in the evaluation criteria? Would the recommended direction actually solve the user's problem? Is there a simpler framing that was overlooked?
- **The Adversary**: Are unknowns actually resolved or just relabeled as assumptions? Are confidence claims calibrated to evidence? Does "What Was Ruled Out" have real reasoning or just a list? Are sources diverse, or do they trace back to a single origin (echo chamber risk)? What's the strongest case for the direction that was ruled out? What assumptions are embedded in the evaluation criteria themselves?

### 3. Define severity before launching reviews

Each reviewer must classify findings using these definitions:

- **Blocking**: Would cause the work to fail, produce wrong output, or require major rework. The document cannot be approved without addressing this.
- **Warning**: Significantly increases risk or reduces quality. Should be addressed before approval but doesn't make the work impossible.
- **Nit**: Minor improvement, polish, or taste preference. Optional.

Reviewers must justify severity, not just assign it. "This is blocking because X" — if the justification is weak, the severity is wrong. A review with everything marked "blocking" is as useless as one with everything marked "nit."

### 4. Launch independent reviews as parallel sub-agents

Launch three sub-agents in parallel using the Agent tool — one per reviewer.
Each agent receives:

- The full document content
- Relevant project overview and lessons.md content
- The document type (spec or research brief)
- That reviewer's specific lens, focus areas, and known bias (from Step 2)
- The severity definitions (from Step 3)
- Instructions to produce findings in the format below

Each agent must produce:

```markdown
## [Reviewer Name] Review

### Findings

For each finding:
- **Issue**: What the problem is
- **Location**: Where in the document it appears (section name or quote)
- **Severity**: blocking / warning / nit
- **Severity justification**: Why this severity, not higher or lower
- **Suggestion**: How to fix it

### Bias Disclosure
Where this reviewer's known bias may have distorted findings above. Flag
specific findings that should be weighted carefully because of this bias.
```

**Why parallel sub-agents**: Sequential reviewers in one context contaminate
each other — later reviewers see earlier findings and anchor to them. Sub-agents
ensure structural independence: each sees only the document and project context,
not what others found. This is necessary but not sufficient — see the note on
shared blind spots in synthesis.

### 5. Synthesize (after all agents return)

Collect all three sub-agent results. Produce two outputs:

**(a) An HTML review report** at `REVIEW-[doc-name]-[YYYY-MM-DD].html`, in the same directory as the source document (or project root if the source isn't in a clear location). This is the artifact the user actually reviews. It is ephemeral — read once, informs the next iteration of the document, then becomes irrelevant. Do not commit it. The decisions and lessons that come out of the review go into `decisions.md` and `lessons.md` in markdown — those are durable.

**(b) A brief in-chat markdown summary**: verdict label, issue counts, recommended next action, and the path to the HTML report. This is what the agent (and you in the next turn) react to. Keep it under 10 lines. The full reasoning lives in the HTML; the summary exists so the next turn doesn't have to re-parse it.

The HTML report contains these sections, in this order:

- **Verdict** (top, prominent) — SOLID / SHAKY / RED FLAG as a large colored badge (green / amber / red) with confidence level and total blocking/warning/nit counts adjacent. The reader sees this without scrolling.
- **Reviewer findings** — three columns or three stacked panels (Builder / Stakeholder / Adversary). Each finding renders as: issue title, location, a colored severity dot (red=blocking, amber=warning, gray=nit), the severity justification, and the suggestion. Each reviewer's bias disclosure appears at the bottom of their column.
- **Agreement** — where 2+ reviewers flagged the same concern. Show which reviewers agreed and link to the relevant findings rather than restating them. Convergence is the strongest signal in the report; it should read that way visually.
- **Tension** — where reviewers disagreed. Present both sides side-by-side, not stacked. Tension is signal, not noise — the layout should make that clear.
- **The Question Nobody Asked** — single section, prose, visually set apart so it isn't lost.
- **Shared Blind Spots** — honest section. Include "I don't know what I'm missing" when that's the truth.
- **If You Proceed Anyway** — top 2–3 risks with mitigations, rendered as small cards or rows.

HTML requirements:
- **Self-contained**: inline CSS, inline SVG for any diagrams, no external dependencies, no JavaScript unless interactivity adds review value (collapsing long finding lists in a reviewer column is the one case where it does).
- **Severity treatment must be unmistakable**: red for blocking, amber for warning, gray for nit. Apply this consistently — colored dot, colored left border, colored badge. Don't mix conventions across reviewers.
- **Three-reviewer layout must make agreement legible**: if Builder and Adversary both flagged the same issue, the layout should let the reader see that without page-flipping. Side-by-side columns or aligned rows work; stacked sections do not.
- **No ASCII art, no faked tables**: use real `<table>`, real SVG. Render `file:line` as a monospace span, not a code fence.
- **Scannable, not a wall**: visual density matters. If a section can't be scanned in 5 seconds, it's too long. The point of HTML here is information density — use it.
- **No prose duplication between sections**: each finding lives in one place. Agreement and Tension reference findings; they don't restate them.

### 6. Verdict

Apply this rubric:

| Verdict | Criteria |
|---------|----------|
| **SOLID** | 0 blocking issues. ≤2 warnings. No unresolved fundamental tension between reviewers. The Question Nobody Asked surfaced nothing material. |
| **SHAKY** | 1 blocking issue OR 3+ warnings OR significant unresolved tension between reviewers. Fixable but not approval-ready. |
| **RED FLAG** | 2+ blocking issues OR fundamental disagreement about whether to proceed at all OR The Question Nobody Asked reveals a missing premise that invalidates the document's framing. |

The verdict appears in two places:

- **In the HTML report**: at the top, as a prominent colored badge with confidence level and issue counts adjacent.
- **In the in-chat markdown summary**: as the first line. The summary should contain the verdict label with confidence (high / medium / low), total blocking/warning/nit counts, a one-line explanation of why this verdict (not the next one up or down), the recommended next action, and the path to the HTML report.

Wait for the user to decide how to proceed.

## Iteration

This skill is one-shot per invocation. After a SHAKY verdict:
1. Address the blocking issues and most consequential warnings
2. Re-run the skill on the revised document
3. The board has no memory of prior runs — each run is fresh

For non-trivial work, expect 1-2 iterations to reach SOLID. If you're on
iteration 3+ and still not SOLID, that's a signal to step back and reconsider
the framing rather than keep grinding on detail.

## Gotcha Section

Guardrails against degeneration:

- **Don't manufacture consensus.** If the three reviewers agree, fine. But
  don't smooth over disagreements for a cleaner output. Tension between
  reviewers is the primary value of the board structure.
- **Each reviewer stays in character**, including acknowledging their biases.
  The Builder doesn't suddenly care about user adoption. The Stakeholder
  doesn't suddenly worry about implementation complexity. The Adversary
  doesn't suddenly become diplomatic. Crossover happens in synthesis only.
- **The Adversary must be constructive.** Challenge what could actually sink
  this, not everything. "This might fail because the rate limit assumption
  isn't validated" is useful. "Have you considered that nothing matters?"
  is not.
- **Severity calibration matters.** A review with everything blocking is as
  useless as one with everything nit. If everything looks blocking, recheck —
  usually some are warnings. Use the Step 3 definitions strictly.
- **Convergence is not proof.** Three reviewers agreeing is meaningful but may
  also reflect shared model biases. Note this in synthesis where relevant.
- **The Question Nobody Asked is mandatory.** It's the section LLMs most often
  skip or fill with throwaway content. Do not skip it.
- **Prioritize the 2-3 critiques that could actually sink this** over
  comprehensive nitpicking. A review with 20 nits and no blocking issues
  buries the signal.
- **Never rubber-stamp.** If something is genuinely SOLID, say SOLID — but say
  specifically *why* it's solid, not "looks good."
- **Never end with "but overall this looks good" unless it genuinely does.**
  That phrase is a reflex, not an assessment.
  - **Don't commit the HTML review.** It's a review surface, not a record. The durable outputs of an adversarial review live in the revised spec, in `decisions.md` (if any decisions came out of it), and in `lessons.md` (if the review caught something that should be a lesson going forward). The HTML itself is read once and discarded.
- **Don't put the full reasoning in the chat summary.** The summary's job is to give the next turn enough to act on (verdict, counts, recommended action, path). The full reasoning lives in the HTML where it can be read once and properly. Restating it in chat defeats the format split.