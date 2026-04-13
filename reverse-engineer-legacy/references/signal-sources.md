# Business Rule Signal Sources — Complete Reference

## TIER 1 — Highest Reliability (mark as CONFIRMED)

These sources are enforced by the runtime or database engine. They cannot be
accidentally bypassed by application code.

| Signal | Where to Find It | What to Extract | Example |
|--------|-----------------|-----------------|---------|
| `CHECK` constraints | Schema DDL, migration files | Value range rules, format rules | `CHECK (amount > 0)` → "Order amount must be positive" |
| `NOT NULL` constraints | Schema DDL, `@NotNull`, `required: true` | Mandatory field rules | `NOT NULL` on `approved_by` → "Approval requires an approver" |
| `UNIQUE` constraints | Schema DDL, `unique: true` | Uniqueness business invariants | `UNIQUE (email)` → "Each email may only register once" |
| `FOREIGN KEY` constraints | Schema DDL, ORM relations | Entity relationship rules | FK from `orders` to `customers` → "Every order must belong to a customer" |
| DB Triggers (`BEFORE INSERT`, `AFTER UPDATE`) | `.sql` files, stored procs | Side-effect and enforcement rules the app cannot control | Trigger setting `modified_at` → SLA audit requirement |
| Stored procedures enforcing state transitions | `.sql`, `.plsql`, `.tsql` | Workflow rules embedded in DB layer | Proc rejecting status change from CLOSED to PENDING → "Closed orders cannot be reopened" |
| Passing test assertions (same-era tests) | `*Test*`, `*Spec*`, `test/`, `spec/` | Intended behavior at time of writing | `assertEquals(APPROVED, order.status)` after approve() call |

---

## TIER 2 — High Reliability (mark as INFERRED)

These sources are intentionally written to encode business rules but live
in the application layer and can theoretically be bypassed.

| Signal | Where to Find It | What to Extract | Example |
|--------|-----------------|-----------------|---------|
| Validation functions | `*Validator`, `*Guard`, `*Checker`, `validate*()` | Required field rules, format rules, range rules | `validateOrderMinimum()` → "Minimum order value is $X" |
| IF/ELSE chains on domain fields | Service layer, business logic classes | Decision rules and branching conditions | `if (customer.tier == GOLD) applyDiscount(0.15)` → "Gold tier customers receive 15% discount" |
| State machine / enum transitions | Switch statements on status fields, state enums | Lifecycle rules | `PENDING → APPROVED` allowed, `APPROVED → PENDING` throws exception |
| Calculation logic (arithmetic on domain values) | Any function performing math on business fields | Pricing, tax, SLA, eligibility formulas | `subtotal * taxRate * (1 - discount)` → tax and discount calculation rule |
| Access control checks | `@Authorize`, `hasRole()`, `can?()`, `isAuthorized()` | Role-based access policies | `hasRole('MANAGER')` before approve → "Only managers can approve orders" |
| API response codes for rule violations | Route handlers, controllers | Implicit contracts — status codes map to specific rule violations | HTTP 409 on duplicate submission → "Duplicate submissions are rejected" |
| HTTP 422 Unprocessable Entity | API error handlers | Validation rule was triggered | Maps to a specific validation rule in the application |

---

## TIER 3 — Medium/Low Reliability (mark as ASSUMED — always flag for human confirmation)

These sources are informative but unreliable as sole evidence. Always pair with
a Tier 1 or 2 source before marking CONFIRMED.

| Signal | Where to Find It | Reliability Caveat | Example |
|--------|-----------------|-------------------|---------|
| Exception messages | `catch` blocks, `throw new Error()` | Written reactively for real scenarios BUT may be outdated after refactors | `"Order cannot be cancelled after shipment"` — honest at time of writing |
| FIXME / TODO / HACK comments | Anywhere in source | Compressed incident reports — each represents a known rule violation or workaround | `// HACK: must recalculate tax here because Spain VAT rules changed in 2019` |
| Feature flags | Config files, DB flags, env vars | Reveals deferred/experimental requirements — may never be activated | `ENABLE_TIERED_PRICING=false` → future pricing model exists but is inactive |
| Magic numbers / hardcoded constants | Hardcoded values throughout code | May be a bug, a temporary fix, or a real rule — always ambiguous without context | `if (amount > 10000) requireApproval()` → $10,000 approval threshold (or is it outdated?) |
| Scheduled job intervals | Cron expressions, scheduler configs | Reveals SLA or time-based business rules | Job runs every 24h → "Daily reconciliation required" |
| Variable and method names | Everywhere | Subject to naming drift — names may no longer match behavior | `calculateFinalPrice()` may apply intermediate discounts, not "final" price |
| Logging messages | Log statements | Written for debugging, not specification — often imprecise | `log.info("Applying premium discount")` — confirms code path but not the rule |
| Migration file comments | Migration scripts | Often contain original developer intent but may be aspirational | `-- Added for GDPR compliance` — intent clear, but implementation may differ |

---

## SPECIAL SIGNALS — Dead Code Classification

Never discard dead/unreachable code without classifying it:

| Classification | Description | BRD Treatment |
|---------------|-------------|---------------|
| `[ABANDONED]` | Feature removed but code left behind. Evidence: commented-out callers, no test coverage, related feature flag = false | Exclude from BRD. Log in Assumption Log as "removed feature" |
| `[DEFENSIVE]` | Handles edge case that "should never happen". Evidence: comment like `// this should never occur`, or catch block with alert/panic | **HIGH VALUE** — include in BRD as INFERRED constraint. These document real incidents. |
| `[FUTURE]` | Feature stubbed but not yet activated. Evidence: feature flag = false, TODO comment, no production callers yet | Include in BRD as FUTURE REQUIREMENT (separate section) |
| `[UNKNOWN]` | Cannot classify with available context | Log in Assumption Log. Request human input. |
