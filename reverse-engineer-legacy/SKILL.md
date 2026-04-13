---
name: reverse-engineer-legacy
description: >
  Reverse-engineer legacy codebases into system architecture documentation and
  structured Business Requirements Documents (BRDs). Use this skill whenever
  the user wants to: understand what a legacy system does, extract business
  rules from code, produce architecture diagrams or component maps from an
  existing codebase, draft a BRD from source code, document an undocumented
  system, or modernize/migrate a legacy application and needs a requirements
  baseline first. Trigger even if the user says things like "figure out what
  this old code does", "what are the business rules in this system", "help me
  document this codebase", "reverse engineer this project", or "I inherited
  this code and have no docs". Always use this skill when a legacy codebase is
  the primary input and structured documentation is the goal.
---

# Reverse Engineer Legacy Code → Architecture + BRD

## Reference Files (load when needed — do NOT pre-load all)
- `references/stack-heuristics.md` — Stack-specific rules (Java/Python/PHP/COBOL/Node). Load when stack is identified in § 1.2.
- `references/signal-sources.md` — Full 3-tier business rule signal table. Load during WAVE 3.
- `references/output-schemas.md` — BR-XXX, COMP-XXX, BRD, and Assumption Log templates. Load before any output is written.

---

## § 0 — SKILL IDENTITY & FATAL GAPS

```
PURPOSE:     Produce architecture maps and BRD drafts from legacy source code
INPUTS:      Source files, DB schemas, configs, directory trees, migration files
OUTPUTS:     Component Inventory, Contract Map, Business Rules Catalog, BRD draft, Assumption Log
FATAL GAPS:  Compiled-only binaries with no schema ▸ STOP-3
             Fully obfuscated code with no DB access ▸ STOP-3
```

---

## § 1 — COLD START PROTOCOL (Always run first — no exceptions)

Do not read any logic files until §1 is complete.

### 1.1 INVENTORY
List every provided file/directory. Classify each as:
`[source | schema | config | test | migration | build | asset | unknown]`

Report:
> "I have X source files, Y schema files, Z test files, W migration files."

If zero schema files AND zero migration files → flag immediately:
> "⚠️ No DB schema detected. Entity relationships will be INFERRED only, not confirmed."

### 1.2 STACK FINGERPRINT
Scan build/config files only. Identify:
- Language(s) and version
- Framework(s)
- Persistence layer (RDBMS / NoSQL / flat-file / in-memory)
- Communication style (REST / SOAP / event-driven / batch / RPC)

→ Load `references/stack-heuristics.md` section for the identified stack NOW.

If stack cannot be identified → **STOP-1** (see § 8).

### 1.3 SCOPE DECLARATION
State explicitly:
1. What CAN be determined from available input
2. What CANNOT be determined (and what additional files would resolve it)
3. Estimated confidence ceiling given what is available

---

## § 2 — CHUNKING STRATEGY (Mandatory for any codebase >5 files)

Process in waves. Complete each wave fully before starting the next.

### WAVE 1 — STRUCTURE *(always fits in context)*
**Read:** Directory tree → Build files → Dependency manifests
**Goal:** Build system map without reading any logic
**Output:** Draft Component Inventory (COMP-XXX records, confidence = INFERRED)

### WAVE 2 — CONTRACTS *(high signal, low volume)*
**Read:** Interface definitions → API route files → DB schema → Config files → `.env.example`
**Goal:** What does this system promise the outside world?
**Output:** Contract Map — inputs, outputs, external dependencies

### WAVE 3 — DOMAIN CORE *(highest business rule density)*
**Load:** `references/signal-sources.md` now.

**Read in this priority order:**
1. Files with `Service`, `Manager`, `Policy`, `Rule`, `Engine`, `Calculator`, `Validator` in name
2. Files with highest cyclomatic complexity (most if/else nesting)
3. DB stored procedures and triggers
4. Constraint definitions (CHECK, NOT NULL, UNIQUE, FK)
5. Enum definitions and state constants

**Output:** Raw Business Rules list — unsorted BR-XXX records with source citations

### WAVE 4 — EVIDENCE LAYER *(confirm or contradict Wave 3)*
**Read:** Test files → Migration history → CHANGELOG / git log snippets → Inline comments (FIXME/TODO/HACK)
**Goal:** Verify Wave 3 extractions. Find rules missed in Wave 3.
**Output:** Confirmed rules, Contradicted rules (flag for § 5), New rules discovered

### WAVE 5 — DEAD WEIGHT *(lowest priority — run last)*
**Read:** Utility code, helpers, logging, formatters
**Goal:** Identify what is NOT business logic — exclude from BRD
**Output:** Exclusion List with classification for each item

---

## § 3 — ARCHITECTURE EXTRACTION (Non-Obvious Heuristics)

Apply these when the obvious approach fails:

**No clear entry points?**
→ Look for scheduler/cron configs, not application code. Check: `quartz.xml`, `crontab`, `.bat`/`.sh`, Windows Task Scheduler, Airflow DAGs. The job definition IS the entry point.

**God classes / spaghetti code?**
→ Do not fight the structure. Group methods by the NOUNS they operate on, not the files they live in. A 3000-line `OrderController` handling auth + pricing + shipping + reporting contains 4 hidden services. Name them by their noun group and create COMP-XXX records for each.

**Implicit data flow via side effects?**
→ Trace by MUTATION, not call chain. Find all writes to shared state (DB writes, session sets, cache sets, static variables). Each write point is a flow terminus. Work backward from terminus to origin.

**Circular dependencies?**
→ A→B→A reveals a MISSING abstraction. Both A and B depend on a concept C that was never extracted. Name C — it is a domain object the original developer didn't identify. Add it to the domain model as COMP-XXX.

**Dead code encountered?**
→ Classify before excluding. Never discard without a label:
- `[ABANDONED]` — feature removed, code left → exclude from BRD
- `[DEFENSIVE]` — handles edge case that "should never happen" → **HIGH VALUE, include**
- `[FUTURE]` — feature stubbed but inactive → include as FUTURE REQUIREMENT

---

## § 4 — BUSINESS RULE SIGNAL PRIORITY

| Tier | Source | Reliability | Action |
|------|--------|-------------|--------|
| **1** | DB constraints (CHECK, NOT NULL, UNIQUE, FK), DB triggers | Highest — enforced at storage layer | Always include, mark CONFIRMED |
| **1** | Passing test assertions (same-era tests) | High — verified contract | Include, mark CONFIRMED |
| **2** | Validation functions (`*Validator`, `*Guard`, `*Checker`) | Good — purpose-built | Include, mark CONFIRMED |
| **2** | IF/ELSE on domain fields, state machine transitions | Good | Include, mark INFERRED |
| **2** | Calculation logic (arithmetic on domain values) | Good | Include, mark INFERRED |
| **2** | Exception messages, API response codes (409, 403, 422) | Medium — written for real scenarios | Include, mark INFERRED |
| **3** | Feature flags, TODO/FIXME/HACK comments | Medium — may be outdated | Include, mark ASSUMED |
| **3** | Magic numbers/constants | Low — may be a bug | Flag as ASSUMED, note location |
| **3** | Variable/method names alone | Lowest | Use only to name known rules, never as sole evidence |

→ Full table with examples: `references/signal-sources.md`

---

## § 5 — CONFLICT RESOLUTION HIERARCHY

When sources contradict, apply in order:

```
TIER 1:  DB constraints and triggers
         "DB enforces it → it IS the rule, regardless of what code says"

TIER 2:  Passing test assertions (same era as code)
         "A passing test is a contract that was verified"

TIER 3:  Application validation logic (not DB-enforced)
         "Enforced at app layer — bypassable via direct DB access"

TIER 4:  Comments and documentation
         "May be outdated — resolve Tier 3 ambiguity only"

TIER 5:  Variable/method naming
         "Last resort only"
```

**Conflict Output Block (use exactly this format):**
```
⚠️  CONFLICT — BR-[XXX]
    Rule under dispute:  [plain English statement]
    TIER 1 says:         [DB constraint / trigger behavior]
    TIER 3 says:         [app code behavior] @ [file:line]
    Resolution:          TIER 1 applied — DB constraint governs
    Recommendation:      App code at [file:line] should be aligned to DB rule
```

---

## § 6 — OUTPUT FORMAT

Load `references/output-schemas.md` before writing any output.

**Required outputs in order:**

| Output | When | Template |
|--------|------|----------|
| Component Inventory | After WAVE 1 | COMP-XXX records |
| Contract Map | After WAVE 2 | Table format |
| Raw Business Rules | After WAVE 3 | BR-XXX records |
| Confirmed/Conflict Report | After WAVE 4 | Conflict blocks + updated BRs |
| BRD Draft | After WAVE 5 + Checkpoint C | Full BRD template |
| Assumption Log | Continuously | ASSUMPTION-XXX records |

**Confidence Labels (mandatory on every BR and COMP record):**
- `CONFIRMED` — Evidence from Tier 1 or 2 source
- `INFERRED` — Evidence from Tier 3 source or consistent across multiple Tier 3+ sources
- `ASSUMED` — Single Tier 3 source, or name/context only

---

## § 7 — CHECKPOINT PROTOCOL (Mandatory — do not skip)

### CHECKPOINT A — After WAVE 1 + 2
Present: Component Inventory + Contract Map
Ask:
> "Are these component names and responsibilities accurate? Any major components missing? Any responsibilities assigned to the wrong component?"

**Wait for confirmation before WAVE 3.**

### CHECKPOINT B — After WAVE 3
Present: Raw Business Rules list (all BR-XXX records)
Ask:
> "Do these rules look correct? Are there obvious rules missing, or rules that are clearly wrong or outdated?"

Incorporate feedback before WAVE 4.

### CHECKPOINT C — After WAVE 4 + 5
Present: Full BRD draft
Ask:
> "Which ASSUMED items can you confirm or deny? Which gaps are blockers vs. acceptable for now?"

Produce final version incorporating all feedback.

---

## § 8 — STOP CONDITIONS

Stop and report rather than speculate when:

```
STOP-1   Stack unidentifiable after scanning all build/config files
         Report: "Cannot identify technology stack. Please provide package.json,
                  pom.xml, requirements.txt, build.gradle, or equivalent."

STOP-2   No DB schema AND no ORM model files exist
         Report: "Cannot confirm data model. All entity relationships will be
                  speculative. Provide schema DDL or ORM model files to proceed
                  with higher confidence."

STOP-3   >40% of source is compiled binaries or minified with no source map
         Report: "Majority of codebase is non-human-readable. Analysis is limited
                  to [X]% of the system. Results may miss core business logic."

STOP-4   Circular contradictions at Tier 1 (DB constraint contradicts DB constraint)
         Report: "Irresolvable conflict at [location]. Two DB constraints enforce
                  mutually exclusive rules. Human review required before BRD
                  can be drafted for this domain area."
```

A skill that stops and asks is safer than one that guesses confidently. Stakeholders build roadmaps on BRDs — wrong outputs compound.
