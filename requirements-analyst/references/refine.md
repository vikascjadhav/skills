# Refinement Rules

This file defines the `REQUIREMENT` artifact template and the rules for populating
each field. Read this before running Phase 2.

The REQUIREMENT artifact is the canonical source of truth. Everything downstream —
architecture, feature stories, acceptance tests — must be traceable to it. Treat it
as a living document: open questions in it are explicitly labeled, not silently omitted.

---

## When to run refinement

Run Phase 2 when:
- Phase 1 (gap analysis) has been completed and critical gaps are either resolved or
  have been accepted as open questions by the user.
- The user invokes `/req-refine` explicitly.

If you're running Phase 2 without having run Phase 1, do a lightweight gap check inline
(10-second scan of the eight dimensions) and flag any critical gaps as open questions
in the artifact. Do not stall on them.

---

## Field-level instructions

### Executive Summary

Write 2–3 sentences maximum. Answer: what is this, who is it for, and what outcome
does it create? Do not describe the solution — describe the outcome.

Good: "Billing admins at multi-account organizations need a consolidated view of charges
across all sub-accounts. Today they reconcile manually across multiple pages. This
requirement captures a consolidated billing dashboard that reduces monthly reconciliation
from hours to minutes."

Bad: "We will add a new billing dashboard page with a consolidated view and export
functionality that integrates with Stripe."

---

### Problem Statement

Ground the problem in observable evidence. Include at minimum:
- Current state: what does the user do today and why is it painful?
- Evidence: where does this come from (user research, support data, churn interviews)?
- Cost of inaction: what happens if this is not built?

If evidence is absent from the input, write: "Evidence not provided — [specific question]
should be validated before launch." Do not fabricate evidence.

---

### Personas

For each persona, write a brief profile. Include:
- Role / job title
- Their context (company size, team, frequency of the task)
- Their current pain or workaround
- What success looks like for them specifically

**Primary persona:** The actor who benefits most directly.
**Secondary persona(s):** Other actors the requirement affects (even passively). Name
them even if underdeveloped — "Admin who configures [X]: not detailed in this version."

Never leave this section as "users." If the persona is unknown, write the clearest
inference and mark it with [ASSUMPTION - needs validation].

---

### Business Value and Success Metrics

State the business goal (one sentence) and at least one measurable success metric.
Metrics must be:
- Quantified (a number or percentage, even if approximate)
- Time-bound (when will it be measured)
- Attributable (how will you know *this* caused the change)

Acceptable: "Reduce billing-related support tickets by 30% within 60 days of launch."
Not acceptable: "Improve user satisfaction with billing."

If no metric was provided, write the most plausible proxy metric and mark it
[PROPOSED — needs sign-off from product owner]. Always offer one, even if provisional.

---

### Scope

**In scope** and **Out of scope** are both required. This is not optional.

Format as two lists. Be concrete — not "future features" but "invoice editing" and
"payment processing" and "email delivery."

Minimum two items per list. If you cannot find two real out-of-scope items, you have
not thought hard enough about adjacent features. Ask yourself: what would a user
naturally assume is included but is not?

---

### Functional Requirements

Number each requirement. Use outcome-focused language, not implementation language.

Format: "The system shall [observable outcome] when [condition]."

Good: "The system shall display a consolidated total charge across all sub-accounts when
a billing admin views the billing page."

Bad: "The system shall call the account aggregation API and render the response as a
table component."

Order by user journey — what happens first in the user's experience comes first in
the list. This makes the list reviewable by non-technical stakeholders.

---

### Business Rules

List conditional logic, pricing rules, permission structures, limits, and validations
that the functional requirements depend on. These are distinct from functional requirements
because they define *how* the system behaves under specific conditions, not just *what*
it does.

Examples:
- "Only billing admins (admin role) may view the consolidated billing page. Regular
  account users may not access sub-account charges."
- "If a sub-account has disputed charges, flag the line item with a ⚠ indicator.
  Do not exclude it from the total."
- "Consolidated totals exclude tax — tax is displayed separately below the total."

If no business rules were stated, infer the obvious ones from the functional requirements
and mark them [INFERRED — confirm with stakeholder].

---

### Acceptance Criteria

Write at minimum three scenarios using Given/When/Then. Include:
1. Happy path (the core workflow, nominal data)
2. At least one boundary condition or edge case
3. At least one error state or failure condition

**Each criterion must use concrete values.** Replace "large number of accounts" with "20
sub-accounts." Replace "overage charges" with "$0.01 per 1,000 tokens beyond quota."
Concrete values make the criterion independently testable.

Template per criterion:
```
**AC-[N]: [Short name]**
Given [precondition with concrete state]
When [user action or system event]
Then [observable outcome with concrete values]
```

---

### Constraints

List hard constraints from the gap analysis. Categories:
- **Technical:** integrations that are required or forbidden, platforms, infrastructure
- **Regulatory/compliance:** GDPR, SOC 2, HIPAA, financial regulations, accessibility
- **Performance:** latency budgets, scale requirements, SLAs
- **Timeline:** hard deadlines, dependency milestones
- **Organizational:** team size, reuse mandates, handoff constraints

If no constraints were stated in a domain where they commonly apply (fintech → compliance,
healthcare → HIPAA), flag it: "No [compliance] constraints stated — [specific check]
recommended before architecture."

---

### Assumptions

List what must be true for this requirement to be valid. Examples:
- "Sub-account data model already exists with a parent–child account relationship."
- "Users' billing role is stored in the existing permissions system."
- "Stripe's API supports account-level charge aggregation (not verified — needs spike)."

Unstated assumptions are the most dangerous kind. If you infer an assumption from the
requirement that was not explicitly stated, mark it [INFERRED].

---

### Open Questions

List everything that is unresolved but not blocking. Format each as:

```
**OQ-[N]:** [Question]
Owner: [who should resolve this — product, engineering, legal, etc.]
Impact if unresolved: [what goes wrong or is delayed]
```

Pull open questions from:
- ⚠ Partial dimensions in the gap analysis
- Business rules marked [INFERRED]
- Assumptions marked [INFERRED]
- Acceptance criteria that could not be made concrete

---

## Full REQUIREMENT template

Use this exact structure. Omit no sections — if a section truly has no content, write
"None identified." rather than removing it. Absence of content in a section is itself
information.

```markdown
## REQUIREMENT: [Title]

**Version:** 1.0 — [Date]
**Owner:** [Requestor or product owner name if known, otherwise "TBD"]
**Status:** Draft | Pending review | Approved

---

### Executive summary
[2–3 sentences: what, for whom, what outcome]

### Problem statement
[Current state, evidence, cost of inaction]

### Personas
**Primary — [Role name]**
[Profile: context, pain, workaround, success definition]

**Secondary — [Role name]** *(if applicable)*
[Brief profile or "Not detailed in this version"]

### Business value and success metrics
**Goal:** [One sentence business goal]
**Metric:** [Quantified, time-bound metric]
**Leading indicator:** [What you'll track before the metric moves — optional but valuable]

### Scope
**In scope**
- [Item 1]
- [Item 2]
- ...

**Out of scope**
- [Item 1]
- [Item 2]
- ...

### Functional requirements
FR-1: The system shall [outcome] when [condition].
FR-2: ...

### Business rules
BR-1: [Rule]
BR-2: ...

### Acceptance criteria
**AC-1: [Name]**
Given [state]
When [action]
Then [outcome with concrete values]

**AC-2: [Name]**
...

**AC-3: [Name]**
...

### Constraints
- **Technical:** [...]
- **Regulatory:** [...]
- **Performance:** [...]
- **Timeline:** [...]

### Assumptions
- [Assumption 1] [INFERRED or stated]
- ...

### Open questions
**OQ-1:** [Question]
Owner: [who]
Impact if unresolved: [consequence]
```

---

## Quality checklist before presenting the artifact

Run through this before delivering the REQUIREMENT:

- [ ] Executive summary describes an outcome, not a solution
- [ ] At least one primary persona named with context
- [ ] Problem statement has evidence or flags its absence
- [ ] Success metric is quantified and time-bound (or flagged as [PROPOSED])
- [ ] Scope has at least two items in each list
- [ ] Every functional requirement is outcome-focused (no API calls, no component names)
- [ ] Business rules cover the conditional logic visible in the ACs
- [ ] Each AC uses concrete values (no "large", "many", "quickly")
- [ ] At least one AC covers an error or edge case
- [ ] All open questions have an owner and impact statement
