# Output Schemas — Mandatory Templates

Use these exact formats for all output. Do not invent alternate formats.
Consistency enables stakeholder review, diffing between sessions, and traceability.

---

## COMP-XXX — Architecture Component Record

```
COMP-[001]
  Name:            [Exact name as it appears in codebase]
  Alias:           [Business-friendly name, inferred from purpose]
  Type:            [UI | API | Service | Repository | Queue | Scheduler | Gateway | Utility]
  Responsibility:  [One sentence — what business function does it serve?]
  Depends on:      [COMP-XXX, COMP-XXX, ...]
  Exposes:         [List of operations, endpoints, or methods the rest of the system calls]
  Owns data:       [Entity names this component is the primary owner of]
  Confidence:      [CONFIRMED | INFERRED]
  Notes:           [Optional — dead code classification, violations, anomalies]
```

**Example:**
```
COMP-007
  Name:            OrderApprovalService
  Alias:           Order Approval Workflow
  Type:            Service
  Responsibility:  Enforces multi-level approval rules for orders exceeding value thresholds
  Depends on:      COMP-003 (OrderRepository), COMP-011 (NotificationService)
  Exposes:         approve(orderId, approverId), reject(orderId, reason), escalate(orderId)
  Owns data:       ApprovalRecord, ApprovalHistory
  Confidence:      CONFIRMED
  Notes:           Dead code path for "auto-approve" exists — classified [ABANDONED]
```

---

## BR-XXX — Business Rule Record

```
BR-[001]
  Statement:    [One sentence, plain English — no code references, no jargon]
  Source Tier:  [1 | 2 | 3]
  Location:     [filename:line_number OR table_name:constraint_name OR test_file:line]
  Confidence:   [CONFIRMED | INFERRED | ASSUMED]
  Trigger:      [What condition or event activates this rule?]
  Actors:       [Who or what is subject to this rule — roles, entities, systems]
  Conflicts:    [BR-XXX if this rule contradicts another — else "None"]
  NFR:          [YES | NO — is this a non-functional requirement (performance, security, etc.)?]
  Future:       [YES | NO — is this rule currently inactive / flagged for future activation?]
```

**Example:**
```
BR-042
  Statement:    Orders with a value above $10,000 require approval from a manager before processing.
  Source Tier:  1
  Location:     orders_approval_trigger.sql:34, OrderApprovalService.java:112
  Confidence:   CONFIRMED
  Trigger:      Order total exceeds $10,000 at time of submission
  Actors:       Customer (submits), Manager (approves/rejects), System (enforces hold)
  Conflicts:    None
  NFR:          NO
  Future:       NO
```

---

## ASSUMPTION-XXX — Assumption Log Record

```
ASSUMPTION-[001]
  Statement:          [What is being assumed?]
  Based on:           [What signal led to this assumption? File, line, pattern]
  Confidence:         [LOW | MEDIUM]
  Verification from:  [Who or what could confirm this? e.g., "Domain expert", "DB admin", "original developer"]
  If wrong, impacts:  [Which BR-XXX records depend on this assumption?]
```

**Example:**
```
ASSUMPTION-007
  Statement:          The 30-day window in the refund logic refers to calendar days, not business days.
  Based on:           RefundService.java:78 — uses ChronoUnit.DAYS without a business-calendar check.
  Confidence:         MEDIUM
  Verification from:  Finance team or original product specification
  If wrong, impacts:  BR-019, BR-020
```

---

## CONFLICT-XXX — Conflict Report Record

```
⚠️  CONFLICT — BR-[XXX]
    Rule under dispute:  [Plain English statement of the rule]
    TIER [N] says:       [Behavior from higher-tier source] @ [location]
    TIER [N] says:       [Behavior from lower-tier source] @ [location]
    Resolution:          TIER [N] applied — [reason]
    Recommendation:      [What should be aligned or investigated]
```

---

## BRD — Full Document Template

```markdown
# [SYSTEM NAME] — Business Requirements Document
**Extraction Date:** [DATE]
**Analyst:** Claude (automated reverse-engineering)
**Codebase Coverage:** [X% of files analyzed]
**Confidence Summary:** [X CONFIRMED | Y INFERRED | Z ASSUMED]
**Version:** Draft 1.0 — Pending human review at Checkpoint C

---

## 1. SYSTEM PURPOSE
[2–3 sentences. What does this system do for the business? Who depends on it?]

---

## 2. ACTORS

| Actor | Type | Primary Role | Source Evidence |
|-------|------|-------------|-----------------|
| [name] | [Human \| System \| External] | [what they do] | [file or pattern] |

---

## 3. BUSINESS PROCESSES

### Process: [Name]
- **Trigger:** [What initiates this process?]
- **Steps:**
  1. [Step with actor]
  2. [Step with actor]
  3. ...
- **Business Rules Applied:** BR-XXX, BR-XXX
- **Outcome:** [What is the result?]
- **Confidence:** [CONFIRMED | INFERRED]

[Repeat for each major process]

---

## 4. BUSINESS RULES CATALOG

[All BR-XXX records, sorted: CONFIRMED first, then INFERRED, then ASSUMED]
[Group by owning component (COMP-XXX) within each confidence tier]

---

## 5. DATA REQUIREMENTS

| Entity | Key Attributes | Constraints | Primary Owner |
|--------|---------------|-------------|---------------|
| [name] | [fields] | [BR-XXX refs] | [COMP-XXX] |

---

## 6. INTEGRATION CONTRACTS

| External System | Protocol | Data Exchanged | Business Purpose | Confidence |
|----------------|----------|----------------|-----------------|------------|
| [name] | [REST/SOAP/queue/file] | [what flows] | [why] | [CONFIRMED/INFERRED] |

---

## 7. NON-FUNCTIONAL REQUIREMENTS

| ID | Category | Statement | Evidence | Current Baseline |
|----|----------|-----------|----------|-----------------|
| NFR-001 | [Performance/Security/Compliance/Availability] | [statement] | [BR-XXX or source] | [observed value if detectable] |

---

## 8. FUTURE REQUIREMENTS (Inactive Features Detected)

| ID | Statement | Evidence | Status |
|----|-----------|----------|--------|
| FR-001 | [feature description] | [feature flag / stub location] | [STUBBED / FLAGGED-OFF] |

---

## 9. ASSUMPTION LOG

[All ASSUMPTION-XXX records]

---

## 10. CONFLICTS DETECTED

[All CONFLICT-XXX records, with resolutions applied]

---

## 11. GAPS — Items Requiring Human Input

| Gap | Why It Cannot Be Resolved Automatically | Information Needed From |
|-----|----------------------------------------|------------------------|
| [description] | [reason — missing schema, compiled binary, etc.] | [who can answer] |

---

## 12. EXCLUSION LIST (Dead / Utility Code — Not Business Logic)

| Component | Classification | Reason Excluded |
|-----------|---------------|-----------------|
| [name] | [ABANDONED / UTILITY / INFRASTRUCTURE] | [brief reason] |
```
