# Feature Decomposition Heuristics

This file defines how to break a refined requirement into independently deliverable stories.
Read this before running Phase 3.

Decomposition is not splitting a big thing into smaller things arbitrarily. It is finding
the natural seams — the places where a partial delivery would still be genuinely useful to a
user. A story that only makes sense when three other stories are also done is not a story;
it's a task in a bigger story.

---

## When NOT to decompose

Skip Phase 3 and tell the user explicitly when:

- The requirement is **Small** (from triage): a single actor, single screen, no integration
  complexity. One story is the correct unit. Decomposing it creates artificial fragmentation.
- The requirement is a single business rule with no variation across user types or phases.
- All the "stories" you're imagining are technically identical and differ only by data
  (e.g., "CRUD for entity A" — that's one story with four AC rows, not four stories).

When skipping, say: "This requirement is self-contained — I won't decompose it into
sub-features. Treat the REQUIREMENT artifact directly as the implementation unit."

---

## Step 1 — Choose a decomposition strategy

Pick the strategy that matches the requirement's natural structure. Do not mix strategies
in one FEATURES artifact; it produces incoherent dependency maps.

### Strategy A: By user journey phase

Use when the requirement describes a multi-step workflow (onboarding, checkout, reporting
cycle). Each phase of the journey becomes a story. Stories can ship sequentially and each
adds value alone.

Best for: workflows, pipelines, multi-step processes.

Example (multi-tenant billing):
- Story 1: View consolidated charges (read path) — admin can see totals
- Story 2: Export consolidated bill to CSV — admin can share externally
- Story 3: Dispute flagging — admin can see which line items are contested

### Strategy B: By user type

Use when the requirement has 2+ personas with distinct needs that can be delivered
independently. Build the primary persona's story first; secondary personas' stories
are additive.

Best for: admin vs. end-user features, producer vs. consumer roles.

Example (notification system):
- Story 1: User receives in-app notification (consumer role)
- Story 2: Admin configures notification rules (producer role)
- Story 3: User manages notification preferences (consumer control)

### Strategy C: By happy path + edge cases

Use when the core flow is simple but the edge cases are complex. Ship the happy path
first so users get value immediately. Edge cases are subsequent stories.

Best for: pricing logic, permission systems, validation-heavy features.

Example (billing with overage):
- Story 1: Standard plan billing (quota-within, simple case)
- Story 2: Standard plan overage (quota-exceeded, model-aware rates)
- Story 3: Premium plan split-rate billing (no quota, separate prompt/completion charges)

### Strategy D: By integration boundary

Use when the requirement spans multiple systems and each integration is a distinct
delivery risk. Stub or mock the integration in earlier stories; replace with real
integration in a later story.

Best for: any requirement that touches 2+ external systems.

Example (payment processing):
- Story 1: Billing calculation (internal only, no payment integration — can test end-to-end)
- Story 2: Stripe checkout integration (external — replaces the stub from Story 1)
- Story 3: Webhook handling for failed payments (async path)

---

## Step 2 — Apply the INVEST check

Before finalizing any story, verify it against INVEST. Every criterion must be met.
If a story fails INVEST, reshape it — don't ship a broken story just because it fits
the strategy.

| Criterion | What it means | Failure sign |
|---|---|---|
| **Independent** | Can be developed and shipped without waiting for another story | "Story 2 only makes sense after Story 1 is live" → combine or reorder |
| **Negotiable** | Not a fixed contract — implementation detail can change | Story describes a specific UI component or API call → rewrite to describe the outcome |
| **Valuable** | Delivers something a real user notices | "Backend refactor for billing service" → not a story, it's a task |
| **Estimable** | A developer can give a rough estimate | "Implement AI-based recommendations" with no definition → too vague |
| **Small** | Completable in 1–5 days | Estimate feels like >1 week → decompose further |
| **Testable** | Has at least one acceptance criterion that is falsifiable | No Given/When/Then → go back to Phase 2 |

---

## Step 3 — Story format

Each story follows this structure:

```markdown
### Story [N]: [Title — verb + outcome, e.g. "View consolidated billing charges"]

**As a** [persona]
**I want to** [action — what they do]
**So that** [outcome — why it matters to them]

**Business value:** [One sentence tying this to the parent REQUIREMENT's goal]

**Scope**
In: [what specifically is covered by this story]
Out: [what is deferred to another story or out of scope entirely]

**Acceptance criteria**

AC-[N]-1: [Name]
Given [state]
When [action]
Then [observable outcome with concrete values]

AC-[N]-2: ...

**INVEST check**
- Independent: [yes / qualified: needs Story N first only for data, not for delivery]
- Valuable: [yes — [brief statement of user value]]
- Small: [yes — estimated N days]
- Testable: [yes — AC-[N]-1 covers core flow, AC-[N]-2 covers edge case]

**REASONS sketch** *(for engineering handoff)*
R — [Summary of what this story requires + definition of done in one sentence]
E — [Key domain entities involved: names and brief role]
A — [Recommended implementation approach — one sentence, outcome-focused, no prescriptive tech]
```

The REASONS sketch in each story covers only R, E, and A — the three elements a requirements
analyst can speak to confidently. Structure, Operations, Norms, and Safeguards (the remaining
four elements of the full canvas) require architectural knowledge and are completed during the
design phase. This partial sketch is a structured prompt for the architect: here are the domain
facts we know, now you complete the picture.

---

## Step 4 — Dependency map

After writing all stories, produce a dependency map. Use a simple table:

| Story | Depends on | Reason |
|---|---|---|
| Story 1 | — | No dependencies |
| Story 2 | Story 1 | Requires the consolidated view to exist for export to work |
| Story 3 | Story 1 | Needs line items rendered before dispute flags can appear on them |

If all stories are truly independent, say so explicitly.

**Recommended ship order:** Based on dependencies and user value, state the recommended
delivery sequence. This is not a sprint plan — it's a recommendation the team can override.

---

## Common decomposition mistakes

**Mistake: Technical tasks disguised as stories.** "Refactor the billing service to support
multiple plans" is not a story — no user sees it. If refactoring is needed, it goes into
the engineering-side Notes of a story, or is flagged as a dependency in the dependency map.

**Mistake: Thin vertical slices that aren't actually usable.** "User can see an empty billing
page" is not a valuable story. The page without data is not useful. The story must include
enough to be demonstrably valuable to the persona.

**Mistake: Decomposing by layer.** "Frontend story / backend story / database story" violates
the Independent criterion by design and produces the opposite of cross-functional delivery.
Always decompose vertically (full stack per story) not horizontally (layer per story).

**Mistake: Over-decomposing to avoid hard problems.** If a story is hard, decomposing it
into ten tiny stories does not make it less hard — it makes it harder to reason about. Acknowledge
complexity in the story's scope and ACs rather than spreading it across many tiny stories.

---

## Story count guidance

| Complexity | Typical story count | Warning sign |
|---|---|---|
| Small | 1 (no decompose) | — |
| Medium | 2–5 | >7 stories suggests Medium was misclassified as Large |
| Large | 5–12 | >15 stories suggests the requirement needs phasing across releases, not decomposition |

If decomposition produces >12 stories, stop and recommend phasing: "This requirement is
large enough to warrant a Phase 1 / Phase 2 split. Phase 1 covers the core happy path and
delivers user value. Phase 2 covers edge cases, integrations, and premium capabilities.
Would you like me to restructure it that way?"
