# FP DATA DICTIONARY
## Project Brief & Implementation Checklist

**A queryable standards service for the FlatPlanet project ecosystem**

> v1.0 · 2026-03-19 · FlatPlanet-Hub

---

| | |
|---|---|
| **Service name** | FP Data Dictionary |
| **Repository** | https://github.com/FlatPlanet-Hub/FP-DATA-DICTIONARY |
| **Database** | Supabase (dedicated instance, separate from TALA) |
| **Who can update** | Developers via admin UI · Claude Code via pull request |
| **Primary consumers** | Claude Code sessions running in any FlatPlanet project |
| **Why it exists** | Replace a static markdown file with a queryable service — so Claude never has to re-read and re-interpret a growing document on every session |

---

## 1. The Problem This Solves

FlatPlanet is moving from one project (TALA, ~20,000 lines) to many TALA-style projects running in parallel across the business. Each project is built in its own Claude Code session. Without a shared standard enforced by a real system, naming drift is inevitable.

> **Core problem:** A static markdown data dictionary does not scale. Every Claude Code session must re-read the entire document, re-process it, and hope nothing was missed or misinterpreted. As the dictionary grows, this gets worse — not better.

| Approach | What happens |
|---|---|
| **Static markdown file** | Claude reads the whole document on every session start. Context window fills up. Rules get misinterpreted. No way to verify compliance. Dictionary grows, problem worsens. One update requires every project to re-read. |
| **FP Data Dictionary (this)** | Claude queries only what it needs, when it needs it. Precise answers in milliseconds. Updates propagate instantly to every project. Compliance is verifiable. Scales to any number of projects. |

---

## 2. What FP Data Dictionary Is

FP Data Dictionary is a small, purpose-built web service with a Supabase database and a REST API. It is the single source of truth for all naming, data type, and structural standards across every FlatPlanet project.

Claude Code sessions do not read a document. They call the API. They ask a precise question and get a precise answer back. The API never gives an ambiguous response — a name is either in the dictionary or it is not.

| Operation | Example |
|---|---|
| Look up a field name | Is `user_id` the correct canonical name for this field? |
| Validate a proposed name | I want to call this column `staff_reference_id` — is that compliant? |
| Get the correct data type | What data type should I use for a money field? |
| List valid enum values | What are the allowed values for task status? |
| Check entity schema | What fields must every Project entity include? |
| Register a new standard | Add `invoice_id` as a new canonical ID field (via PR, not direct write) |

---

## 3. Architecture

### Database — Supabase

Dedicated Supabase instance, separate from TALA. The dictionary must not share infrastructure with any project it governs — that would create a circular dependency. Supabase provides the database, row-level security, and authentication for the admin UI.

#### Core tables

| Table | Purpose |
|---|---|
| `entities` | Canonical entity names (User, Project, Task, etc.) with descriptions and ownership |
| `fields` | Canonical field names, their entity, data type, whether required, and description |
| `data_types` | Approved data type definitions — what they are, when to use them, examples |
| `enums` | Closed-list enum definitions — the entity, the field, and all allowed values |
| `forbidden_names` | Variable and field names that must never be used in any project |
| `standards` | Structural rules (naming conventions, file requirements, branch strategy, etc.) |
| `change_log` | Every update to the dictionary — what changed, when, who approved it |

### API — REST Endpoints

A lightweight REST API that Claude Code (and any other consumer) calls. Read-only for Claude. Writes only via the admin UI or approved pull requests.

| Endpoint | What it does |
|---|---|
| `GET /v1/fields/:name` | Look up a canonical field by name — returns type, required status, description |
| `GET /v1/fields/validate` | Pass a proposed name, get back compliant/non-compliant + reason |
| `GET /v1/entities/:name` | Get the full schema for a named entity including all required fields |
| `GET /v1/enums/:entity/:field` | Get all allowed values for a given enum field |
| `GET /v1/data-types/:type` | Get the definition and usage rules for a data type |
| `GET /v1/forbidden` | Get the full list of forbidden variable names |
| `GET /v1/standards` | Get all structural standards (supports `?category=` filter) |
| `POST /v1/fields` | Add a new field definition (admin/PR only — not available to Claude) |

### Admin UI

A simple web interface for developers to manage the dictionary without writing SQL. Built on top of the same Supabase instance. Access controlled — only authorised developers can make changes. All changes are logged automatically in `change_log`.

- Browse and search all entities, fields, data types, and enums
- Add new entries or edit existing ones
- View the full change history for any entry
- Propose changes via pull request (for standards that require wider review)
- See which projects are actively consuming which standards

---

## 4. How Claude Code Uses the Service

Every FlatPlanet project has a `CLAUDE.md` at its root. That file tells Claude Code the FP Data Dictionary API endpoint and instructs it to query the service before making any naming or structural decision. Claude never guesses — it always checks.

### Step 1 — Before naming anything
Call `GET /v1/fields/validate` with the proposed name. If non-compliant, call `GET /v1/fields` to find the correct canonical name and use that instead. Never invent a name that is not in the dictionary.

### Step 2 — Before defining a data type
Call `GET /v1/data-types` to confirm the correct type for the concept. Money is always integer cents. Timestamps are always ISO 8601. Percentages are always 0.0–1.0. The API enforces this — Claude does not decide.

### Step 3 — Before creating an entity or table
Call `GET /v1/entities/:name` to get the required schema. Every entity must include `id` (prefixed ULID), `created_at`, `updated_at`, and `deleted_at`. Build from what the API returns — not from memory.

### Step 4 — When encountering an enum field
Call `GET /v1/enums/:entity/:field` to get the closed list of allowed values. Never invent a new status value or enum entry. If the value is not in the list, flag it and ask for clarification rather than proceeding.

---

## 5. How Standards Get Updated

The dictionary is a live system — it will grow as FlatPlanet grows. But changes must be deliberate.

| Change type | Process |
|---|---|
| **Small changes** (new field, new enum value) | Developer logs into the admin UI, adds the entry with description and rationale. Change is logged automatically. Propagates to all projects instantly on next API call. |
| **Standard changes** (new rule, rename, deprecation) | Developer opens a pull request against the FP-DATA-DICTIONARY repo with the proposed change and rationale. Reviewed and approved before merging. |

> **Claude Code can never write to the dictionary directly. It can only read. Standards are owned by people, not by AI sessions.**

---

## 6. Build Phases

### Phase 1 — Supabase Schema & Seed Data `[HIGHEST PRIORITY — start here]`

Build the database first. Everything else depends on it.

Create the Supabase project (separate from TALA). Define all tables with correct column types, constraints, and row-level security. Seed the dictionary with the existing standards from the current `DATA_DICTIONARY.md` — entities, fields, data types, enums, forbidden names, and structural rules.

### Phase 2 — REST API `[HIGH]`

Build the read API. This is what Claude Code actually calls.

A lightweight Node.js/TypeScript service deployed to a stable URL. All GET endpoints from the architecture section. Read-only — no write endpoints exposed publicly. Returns clean JSON. Versioned from day one (`/v1/`). Rate limited to prevent abuse.

### Phase 3 — CLAUDE.md Integration Pattern `[HIGH]`

Define how every FlatPlanet project connects to the service.

Write the standard `CLAUDE.md` block that every project includes. This block tells Claude Code the API base URL, the query pattern for each type of check, and the rule that Claude must always query before naming or typing anything. Test this in TALA first before rolling out to new projects.

### Phase 4 — Admin UI `[MEDIUM]`

Give developers a way to manage the dictionary without SQL.

Simple web UI built on the Supabase instance. Browse, search, add, and edit entries. Full change log visible. Access controlled via Supabase auth. Functional and reliable is the priority — this is an internal developer tool.

### Phase 5 — Migration & Rollout `[MEDIUM]`

Connect TALA and all new projects to the live service.

Audit the existing TALA codebase against the dictionary. Identify any naming drift from before the standards existed. Update TALA's `CLAUDE.md` to point to the FP Data Dictionary API. Document the integration pattern so every new project starts connected from day one.

---

## 7. Implementation Checklist

Work through phases in order. Phase 1 is the hard dependency for everything else.

### Phase 1 — Supabase Schema & Seed Data

- [ ] Create new Supabase project — separate from TALA, do not share the same instance
- [ ] Create `entities` table — `id` (ULID), `name`, `description`, `owner`, `created_at`, `updated_at`, `deleted_at`
- [ ] Create `fields` table — `id`, `entity_id` (FK), `name`, `data_type`, `required` (bool), `description`, `example`, `created_at`, `updated_at`, `deleted_at`
- [ ] Create `data_types` table — `id`, `name`, `definition`, `when_to_use`, `example`, `forbidden_alternatives`, `created_at`, `updated_at`
- [ ] Create `enums` table — `id`, `entity_id` (FK), `field_name`, `allowed_values` (array), `created_at`, `updated_at`
- [ ] Create `forbidden_names` table — `id`, `name`, `reason`, `created_at`
- [ ] Create `standards` table — `id`, `category`, `rule`, `rationale`, `created_at`, `updated_at`, `deleted_at`
- [ ] Create `change_log` table — `id`, `table_name`, `record_id`, `change_type`, `changed_by`, `changed_at`, `before` (JSON), `after` (JSON)
- [ ] Apply row-level security to all tables — read: public, write: authenticated developers only
- [ ] Seed all entities from existing DATA_DICTIONARY.md — User, Organisation, Project, Task, API Key, Audit Log
- [ ] Seed all canonical fields including universal fields: `id`, `created_at`, `updated_at`, `deleted_at`
- [ ] Seed all data types — money (integer cents), timestamps (ISO 8601), percentages (0.0–1.0), IDs (prefixed ULIDs), booleans, strings
- [ ] Seed all enums — all closed-list status values from the current dictionary
- [ ] Seed all forbidden names — `data`, `temp`, `obj`, `item`, `res`, `cb`, `e` and all others from the dictionary
- [ ] Seed all structural standards — branching strategy, code conventions, file requirements, API conventions

### Phase 2 — REST API

- [ ] Initialise Node.js/TypeScript project in FlatPlanet-Hub/FP-DATA-DICTIONARY repo
- [ ] Connect to Supabase using service role key (read-only scope)
- [ ] Build `GET /v1/fields/:name` endpoint
- [ ] Build `GET /v1/fields/validate` endpoint — accepts proposed name, returns `compliant: true/false` + reason + canonical alternative if false
- [ ] Build `GET /v1/entities/:name` endpoint — returns full required schema
- [ ] Build `GET /v1/enums/:entity/:field` endpoint
- [ ] Build `GET /v1/data-types/:type` endpoint
- [ ] Build `GET /v1/forbidden` endpoint — returns full forbidden names list
- [ ] Build `GET /v1/standards` endpoint — supports `?category=` filter
- [ ] Add rate limiting — generous limits for legitimate Claude Code use
- [ ] Add API versioning from day one — `/v1/` prefix on all routes
- [ ] Write API documentation — endpoint reference Claude Code can be pointed to
- [ ] Deploy to stable URL — must be accessible from any Claude Code session

### Phase 3 — CLAUDE.md Integration Pattern

- [ ] Write the standard CLAUDE.md dictionary block — API base URL, query instructions, enforcement rules
- [ ] CLAUDE.md must instruct Claude to validate all names via `/v1/fields/validate` before use
- [ ] CLAUDE.md must instruct Claude to get entity schemas via `/v1/entities/:name` before building tables
- [ ] CLAUDE.md must instruct Claude to check enums via `/v1/enums` before using any status value
- [ ] CLAUDE.md must instruct Claude to never write to the dictionary — read only
- [ ] Test the integration pattern in TALA first
- [ ] Update TALA CLAUDE.md to point to live FP Data Dictionary API
- [ ] Document the integration pattern for all future projects

### Phase 4 — Admin UI

- [ ] Scaffold web UI project — React, connects to same Supabase instance
- [ ] Build entities list and detail view
- [ ] Build fields list with search and filter by entity
- [ ] Build add/edit form for fields
- [ ] Build enums management view
- [ ] Build forbidden names management view
- [ ] Build change log view — filterable by table, date, and user
- [ ] Add Supabase auth — only authorised developers can write, anyone can read
- [ ] Deploy UI to stable URL

### Phase 5 — Migration & Rollout

- [ ] Audit TALA codebase against the live dictionary — identify naming drift from pre-standards development
- [ ] Document all drift found — what it is, how significant, whether to fix now or flag for next sprint
- [ ] Update all new project templates to include FP Data Dictionary CLAUDE.md block
- [ ] Write onboarding note for business managers — what the dictionary is and why Claude uses it
- [ ] Write developer guide — how to add a new standard, how to propose a change, how to read the change log

---

## 8. Decisions Already Made — Do Not Re-Open

| Decision | What was decided |
|---|---|
| Service name | FP Data Dictionary |
| Database | Supabase — dedicated instance, not shared with TALA |
| API access for Claude | Read only — Claude can never write to the dictionary |
| Write access | Developers via admin UI, or via pull request for structural changes |
| API style | REST — simple, stateless, easy for Claude to call |
| ID format | Prefixed ULIDs — strings, never integers |
| Money type | Integer cents — no floats, ever |
| Enum values | Closed lists only — Claude cannot invent new values |

---

*FP Data Dictionary v1.0 · 2026-03-19 · FlatPlanet-Hub*
