# Gap Analysis Rubric

This file defines the eight dimensions used in Phase 1. For each dimension, check the
requirement against the criteria, assign a status, and — if the status is not Clear —
produce one specific, targeted question that would close the gap.

Never produce vague follow-up questions ("can you clarify the scope?"). Every question
must be answerable with a concrete, bounded answer.

---

## How to score each dimension

| Status | Symbol | Meaning |
|---|---|---|
| Clear | ✓ | Enough information to proceed. No question needed. |
| Partial | ⚠ | Something is stated but incomplete or ambiguous. One question resolves it. |
| Missing | ✗ | No information present. One question is required before refinement can proceed reliably. |

A dimension rated ✗ is a **critical gap** — flag it in the Critical Gaps section of the
output. A dimension rated ⚠ becomes an **open question** in the REQUIREMENT artifact
(does not block refinement, but must be surfaced).

---

## The Eight Dimensions

### 1. Core Intent

**What to check:** Is the underlying outcome — the *why* — stated or clearly inferable?
If the requirement describes a solution (what to build) rather than an outcome (what
changes for whom), the intent is not captured.

**Clear:** "Finance managers need to reconcile billing across accounts without exporting
to spreadsheets manually." — outcome is explicit.

**Partial:** "Add an export button." — the action is clear but the outcome (who needs
this, and why) is not stated.

**Missing:** "Improve the dashboard." — no outcome, no actor, no change described.

**Diagnostic question when Partial/Missing:**
> "What outcome does this change create for the user — what will they be able to do,
> or stop doing, once this exists?"

---

### 2. Primary Persona

**What to check:** Is there at least one named user type with enough context to reason
about their needs? Check for: role/job title, their current situation, what they
struggle with, what success looks like for them. Anonymous "users" are not personas.

**Clear:** "A billing admin at a mid-size SaaS company who currently manually reconciles
invoices across 3–10 sub-accounts at end of month."

**Partial:** "Finance team" — a role exists but no context about their workflow or pain.

**Missing:** "Users" or "customers" with nothing else.

**Diagnostic question when Partial/Missing:**
> "Who specifically experiences this problem — what is their role, and what does their
> current workaround look like?"

**Secondary personas:** If the requirement implies more than one actor (e.g., an admin
who configures + a user who consumes), flag secondary personas even if the requirement
only describes one. Note them as "identified but not detailed" in the output.

---

### 3. Problem Statement

**What to check:** Is the pain or opportunity grounded in something observable? Look for:
current state description, evidence of the problem (metrics, user quotes, frequency),
and cost of inaction (churn, revenue, compliance risk, wasted time).

**Clear:** "Customers who manage sub-accounts currently export CSVs from three different
pages, merge them in Excel, and spend 2–4 hours per month on reconciliation. Two customers
mentioned this as a cancellation risk in Q4 churn interviews."

**Partial:** "It's tedious to manage billing across accounts." — problem exists but no
evidence or cost.

**Missing:** No problem described — only a solution.

**Diagnostic question when Partial/Missing:**
> "What is the evidence this problem exists — do you have usage data, support tickets,
> or user research that shows the frequency or cost?"

---

### 4. Business Value and Success Metric

**What to check:** Is there a measurable business outcome? This must be quantifiable
(even if approximate) and time-bound. Vague value statements ("improves UX") do not count.

**Clear:** "Reduce billing-related support tickets by 30% and cut monthly reconciliation
time from ~3h to <15 min, measured 60 days post-launch."

**Partial:** "Saves time for finance teams." — directional but not measurable.

**Missing:** No business value stated.

**Diagnostic question when Partial/Missing:**
> "How will you know in 60 days that this was worth building — what number moves, or
> what user behavior changes?"

---

### 5. Scope (In and Out)

**What to check:** Is it clear what is included and what is explicitly excluded? The
absence of scope-out items is itself a risk — it means adjacent features are implicitly
in scope.

**Clear:** "In scope: consolidated view, CSV export of consolidated bill. Out of scope:
individual account invoice editing, payment processing, email delivery of invoices."

**Partial:** In scope described but no out-of-scope items.

**Missing:** No scope stated. The requirement could mean almost anything.

**Diagnostic question when Partial/Missing:**
> "What are 2–3 adjacent things users might *assume* are included that you want to
> explicitly exclude from this version?"

---

### 6. Acceptance Criteria and Falsifiability

**What to check:** Can you write a test for this requirement? At least one scenario must
be falsifiable — a human must be able to run it and produce a pass or fail definitively.
Vague outcomes ("works correctly", "is fast", "is intuitive") are not criteria.

**Clear:** Given/When/Then scenarios with concrete values. Example: "Given an admin with
3 sub-accounts totaling $4,200 in charges, when they view the consolidated billing page,
then they see $4,200 total broken down by account with individual line items."

**Partial:** Success described in outcome terms but no concrete scenario.

**Missing:** No success criteria or acceptance condition stated.

**Diagnostic question when Partial/Missing:**
> "Describe one concrete scenario you'd use to verify this works — what input, what
> does the user do, and what result do they see?"

---

### 7. Constraints

**What to check:** Are there hard constraints that limit the solution space? Types to
look for: technical (must integrate with X, must not break Y), regulatory (GDPR, SOC 2,
financial compliance), timeline (must ship by date Z), budget, performance (latency,
scale), accessibility, or organizational (must reuse existing auth system).

**Clear:** "Must integrate with Stripe billing API, must comply with SOC 2 data residency
requirements, target 60-day delivery."

**Partial:** One constraint mentioned but likely others exist (e.g. Stripe mentioned but
no compliance or timeline constraints).

**Missing:** No constraints mentioned.

**Diagnostic question when Partial/Missing:**
> "Are there any hard constraints — technical integrations you must use, compliance
> requirements, or deadline that can't move?"

Note: Absence of constraints is not automatically ✗ — some requirements genuinely have
none. If the domain (fintech, healthcare, enterprise) implies constraints, flag ⚠ and ask.

---

### 8. Assumptions and Dependencies

**What to check:** What must be true — either in the world or in the product — for this
requirement to be valid? Dependencies on other features, other teams, or external systems
should be named. Unstated assumptions compound into delivery surprises.

**Clear:** "Assumes sub-account structure already exists in our data model. Depends on
billing team delivering the account aggregation API (Q2 milestone) before this can be
built."

**Partial:** Some dependencies mentioned but others are inferable and unstated (e.g. an
auth requirement that assumes SSO exists).

**Missing:** No assumptions or dependencies mentioned for a requirement that clearly has them.

**Diagnostic question when Partial/Missing:**
> "What needs to already exist — in your system or externally — for this to be
> buildable? Are there other teams or timelines you're depending on?"

---

## Output assembly

After scoring all eight dimensions, produce the `GAP-ANALYSIS` artifact in the format
defined in SKILL.md. Use this table structure for the dimension checks:

| Dimension | Status | Gap / observation | Question to resolve |
|---|---|---|---|
| Core intent | ✓/⚠/✗ | [what was found] | [targeted question or "—"] |
| Primary persona | ... | ... | ... |
| Problem statement | ... | ... | ... |
| Business value | ... | ... | ... |
| Scope | ... | ... | ... |
| Acceptance criteria | ... | ... | ... |
| Constraints | ... | ... | ... |
| Assumptions/dependencies | ... | ... | ... |

**Bias toward proceeding.** If 5 of 8 dimensions are Clear and the 3 that are Partial
are genuinely resolvable with an open question, do not block — produce the REQUIREMENT
artifact with those three as open questions. Only truly block when a critical gap would
make the requirement self-contradictory or unscope-able.
