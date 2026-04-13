# Stack-Specific Heuristics

Load only the section matching the identified stack. Ignore other sections.

---

## JAVA ENTERPRISE (Spring, EJB, Struts, Quarkus)

**Where business rules live:**
- `@Service` beans — primary business logic layer (NOT `@Controller`)
- `@Transactional` boundaries — each annotation marks an atomic business operation; the boundary IS a business rule
- `@Valid` / `@NotNull` / `@Size` annotations — required fields and constraints without reading validation code
- `persistence.xml` / Hibernate mappings / JPA `@Entity` classes — entity relationships are more reliable here than in application code

**Key patterns:**
- `@PreAuthorize` / `@Secured` — role-based access rules
- `ApplicationEvent` + `@EventListener` — hidden async business triggers (easy to miss)
- `@Scheduled` — SLA/time-based business rules
- Custom `Exception` classes in a `domain/exception/` package — each one names a business rule violation

**Priority read order:**
1. All `*Service.java` files
2. All `*Repository.java` interfaces (method names encode query semantics)
3. `@Entity` classes and their field-level annotations
4. `@ControllerAdvice` / exception handlers (map rule violations to HTTP codes)
5. `resources/db/migration/` (Flyway/Liquibase) — chronological rule evolution

---

## PYTHON (Django, Flask, FastAPI, SQLAlchemy)

**Where business rules live:**
- `models.py` — ground truth for entity structure; field validators and `Meta` class encode DB-level constraints
- Django `signals.py` — implicit side effects; each signal handler is a hidden business rule triggered by state change
- Django `Meta` class — `unique_together`, `constraints` = business invariants
- Celery task definitions — async business process flows; each task is a business step

**Key patterns:**
- `@property` methods on models — computed business values (eligibility, totals, status derivations)
- `clean()` / `validate()` methods — cross-field business validation rules
- Django `permissions` and `has_perm()` calls — access control policies
- `migrations/` folder read chronologically — full history of domain model evolution
- `settings.py` constants — hardcoded business thresholds (rate limits, expiry periods, fee percentages)

**Priority read order:**
1. All `models.py` files
2. All `signals.py` files
3. All `serializers.py` (DRF) — validation rules often live here
4. All `views.py` / `viewsets.py` — business process flows
5. Migration files (oldest first) — rule evolution history

---

## PHP (Laravel, WordPress, Symfony, Legacy Procedural)

**Laravel:**
- `app/Policies/` — access control rules (most authoritative for "who can do what")
- `app/Observers/` — side-effect rules (what happens AFTER a state change)
- `app/Rules/` — custom validation rules = explicit business constraints
- `database/migrations/` — schema evolution; read oldest-first for rule origin
- `.env.example` — reveals integration points and threshold constants

**WordPress:**
- `add_filter()` / `add_action()` — the ENTIRE business logic layer. Every hook is a rule.
- Priority parameter on hooks (third arg) — encodes rule ordering/precedence
- `wp_options` table references — hardcoded business configuration
- Custom post types and meta keys — domain model encoded as WordPress constructs

**Legacy Procedural PHP:**
- `include` / `require` chains — follow these to find the actual execution path
- `globals.php` / `config.php` / `constants.php` — almost always contains hardcoded business thresholds
- Functions prefixed with `check_`, `validate_`, `can_`, `is_` — business rule enforcement functions
- Session variable names — encode business state machine implicitly

**Priority read order (Laravel):**
1. `database/migrations/` (oldest first)
2. `app/Models/` — Eloquent relationships and casts
3. `app/Policies/` + `app/Rules/`
4. `app/Services/` or `app/Actions/`
5. `app/Observers/`

---

## COBOL / RPG (Mainframe Legacy)

**Where business rules live:**
- `WORKING-STORAGE SECTION` — treat exactly like a DB schema; fields ARE the domain model
- `PERFORM` paragraphs — each named paragraph is a business step; paragraph names encode intent
- `EVALUATE` statements — business decision rules (COBOL equivalent of switch/case)
- `COPY` books (`.cpy` files) — shared business definitions; treat like interfaces or shared entity schemas
- JCL job steps — business process orchestration; each STEP is a workflow stage

**Key patterns:**
- `88` level condition names — named business states (e.g., `88 ORDER-APPROVED VALUE 'A'`)
- `REDEFINES` clauses — variant records encoding multiple business subtypes in one structure
- Numeric field `PICTURE` clauses with implied decimals — encode precision rules for financial calculations
- `FILE SECTION` — reveals all external integrations (every file = an external system boundary)
- `SORT` verb — reveals business ordering/priority rules

**Priority read order:**
1. All COPY books first (shared definitions)
2. `WORKING-STORAGE SECTION` of main programs
3. `FILE SECTION` (integration map)
4. JCL scripts (workflow orchestration)
5. Main `PROCEDURE DIVISION` — follow PERFORM chains

---

## NODE.JS / EXPRESS / NESTJS

**Where business rules live:**
- Middleware chain ORDER — policy enforcement sequence; auth before authz before rate-limit before business logic is itself a rule
- Route definitions — API contract; treat like interface specifications
- Mongoose schema validators and `pre`/`post` hooks — business constraints and side effects
- NestJS `Guards` — access control; `Pipes` — input validation rules; `Interceptors` — cross-cutting concerns

**Key patterns:**
- `EventEmitter` patterns — implicit async business process triggers
- `Bull` / `Agenda` queue job definitions — async business workflows
- Environment variable names in `process.env.*` — business configuration thresholds
- Middleware applied only to specific route groups — scoped business policies
- Error classes in `errors/` or `exceptions/` directory — business rule violations

**Priority read order:**
1. Route definition files (`routes/`, `*.router.ts`)
2. Mongoose models or TypeORM entities
3. Service files (`*.service.ts`, `services/`)
4. Guards and validation pipes (NestJS)
5. Event emitter listener registrations

---

## RUBY ON RAILS

**Where business rules live:**
- `app/models/` — ActiveRecord validations, callbacks (`before_save`, `after_create`), and scopes ARE business rules
- `db/schema.rb` — ground truth for constraints; always prioritise over migration files for current state
- `app/policies/` (Pundit) — access control rules
- `config/initializers/` — startup-time business configuration
- `lib/` — custom business logic not tied to a model

**Key patterns:**
- `validates` macros — field-level business constraints
- `before_*` / `after_*` callbacks — side-effect rules
- Named scopes — implicit query-level business filters (e.g., `scope :active, -> { where(status: 'active') }`)
- `enum` declarations — state machine vocabulary
- `has_many :through` — join models often encode business relationships with their own rules

**Priority read order:**
1. `db/schema.rb`
2. All model files in `app/models/`
3. `app/policies/` (if Pundit) or `app/abilities/` (if CanCanCan)
4. `app/services/` or `app/interactors/`
5. `config/initializers/`
