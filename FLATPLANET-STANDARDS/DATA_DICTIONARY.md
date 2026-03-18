# FLATPLANET Data Dictionary

> Single source of truth for all variable names, data types, entity fields, and naming conventions across all FlatPlanet projects. Claude Code must reference this file at the start of every project.

---

## How to Use This Dictionary

1. Before naming any variable, check this file first
2. If the concept exists here, use the exact name defined
3. If the concept does not exist, follow the naming patterns and add it via PR
4. Never invent synonyms: one concept = one name, always

---

## 1. ID Conventions (Stripe-inspired prefix system)

All IDs use a prefix so the entity type is identifiable from the ID alone.
ID format: prefix + ULID (26 chars, sortable, URL-safe)
IDs are ALWAYS strings. Never integers.

| Entity       | Prefix    | Example                            |
|--------------|-----------|------------------------------------|
| User         | fp_usr_   | fp_usr_01j9x2k4m8n3p5q7r          |
| Project      | fp_proj_  | fp_proj_01j9x2k4m8n3p5q7r         |
| Session      | fp_ses_   | fp_ses_01j9x2k4m8n3p5q7r          |
| Organization | fp_org_   | fp_org_01j9x2k4m8n3p5q7r          |
| Team         | fp_team_  | fp_team_01j9x2k4m8n3p5q7r         |
| File         | fp_file_  | fp_file_01j9x2k4m8n3p5q7r         |
| Task         | fp_task_  | fp_task_01j9x2k4m8n3p5q7r         |
| Event        | fp_evt_   | fp_evt_01j9x2k4m8n3p5q7r          |
| Webhook      | fp_whk_   | fp_whk_01j9x2k4m8n3p5q7r          |
| API Key      | fp_key_   | fp_key_01j9x2k4m8n3p5q7r          |
| Audit Log    | fp_log_   | fp_log_01j9x2k4m8n3p5q7r          |
| Notification | fp_notif_ | fp_notif_01j9x2k4m8n3p5q7r        |

Rules:
- Foreign key fields: append _id (user_id, project_id, org_id)
- Never expose sequential numeric IDs in public APIs
- Never use auto-increment integers as public identifiers

---

## 2. Canonical Data Types (Google-inspired)

One type per concept. No exceptions.

| Concept        | Type    | Format / Notes                                    |
|----------------|---------|---------------------------------------------------|
| ID             | string  | Prefixed ULID (see Section 1)                     |
| Timestamp      | string  | ISO 8601 UTC: 2026-03-18T14:30:00Z               |
| Unix timestamp | integer | Seconds since epoch. Never milliseconds.          |
| Date only      | string  | ISO 8601: 2026-03-18                              |
| Duration       | integer | Always in seconds                                 |
| Money/price    | integer | Smallest unit (cents). NEVER float.               |
| Currency       | string  | ISO 4217: USD, PHP, EUR, GBP                      |
| Percentage     | float   | 0.0 to 1.0. Not 0 to 100. (0.25 = 25%)           |
| Boolean        | boolean | true/false only. Never 0/1 or yes/no.             |
| Email          | string  | Lowercase, trimmed, validated on input            |
| Phone          | string  | E.164 format: +639171234567                       |
| URL            | string  | Always include protocol: https://                 |
| File size      | integer | Always in bytes                                   |
| Latitude       | float   | Decimal degrees: 14.5995                          |
| Longitude      | float   | Decimal degrees: 120.9842                         |
| Color          | string  | Hex: #FF5733                                      |
| Language       | string  | BCP 47: en, en-US, fil, zh-CN                     |
| Country code   | string  | ISO 3166-1 alpha-2: US, PH, GB                    |
| Timezone       | string  | IANA: Asia/Manila, America/New_York               |
| Enum/status    | string  | snake_case values only (see Section 5)            |
| Array/list     | array   | Never store as comma-separated string             |
| Password       | string  | Never stored plain. Always bcrypt/argon2 hashed.  |
| Token/secret   | string  | Never logged. Never returned after creation.      |

---

## 3. Universal Field Names

Use these names consistently across ALL entities and tables.

### Identity Fields
| Field        | Type   | Description                                       |
|--------------|--------|---------------------------------------------------|
| id           | string | Primary key with FP prefix                        |
| external_id  | string | ID from a third-party system                      |
| slug         | string | URL-safe human-readable identifier                |
| name         | string | Display name, not necessarily unique              |
| display_name | string | Formatted name for UI                             |
| full_name    | string | first_name + last_name combined                   |
| first_name   | string | Given name                                        |
| last_name    | string | Family/surname                                    |
| username     | string | Unique, lowercase, alphanumeric + hyphens         |
| email        | string | Lowercase, validated, unique per entity           |
| phone        | string | E.164 format                                      |
| avatar_url   | string | Full URL to profile image                         |
| bio          | string | Short description, max 280 chars                  |
| description  | string | Longer free-text description                      |
| notes        | string | Internal notes, not shown to end users            |

### Timestamp Fields (required on every table)
| Field        | Type   | Description                                       |
|--------------|--------|---------------------------------------------------|
| created_at   | string | ISO 8601 UTC. Set on insert, never updated.       |
| updated_at   | string | ISO 8601 UTC. Auto-updated on every change.       |
| deleted_at   | string | ISO 8601 UTC. Null if not deleted (soft delete).  |
| published_at | string | ISO 8601 UTC. When record became public.          |
| expires_at   | string | ISO 8601 UTC. When record becomes invalid.        |
| started_at   | string | ISO 8601 UTC. When a process/session started.     |
| ended_at     | string | ISO 8601 UTC. When a process/session ended.       |
| scheduled_at | string | ISO 8601 UTC. When something is scheduled to run. |
| last_seen_at | string | ISO 8601 UTC. Last activity timestamp.            |
| verified_at  | string | ISO 8601 UTC. When verification was completed.    |

### Status and State Fields
| Field       | Type    | Description                                      |
|-------------|---------|--------------------------------------------------|
| status      | string  | Current lifecycle state (see Section 5)          |
| state       | string  | Current machine/workflow state                   |
| is_active   | boolean | Whether record is currently active               |
| is_deleted  | boolean | Soft delete flag (prefer deleted_at)             |
| is_verified | boolean | Whether email/phone/identity is verified         |
| is_public   | boolean | Whether visible to unauthenticated users         |
| is_featured | boolean | Whether highlighted/promoted                     |
| is_default  | boolean | Whether this is the default option               |
| is_archived | boolean | Whether moved to archive                         |
| is_locked   | boolean | Whether prevented from edits                     |

### Relationship / Foreign Key Fields
| Field       | Type   | Description                                       |
|-------------|--------|---------------------------------------------------|
| user_id     | string | FK to users                                       |
| org_id      | string | FK to organizations                               |
| project_id  | string | FK to projects                                    |
| team_id     | string | FK to teams                                       |
| parent_id   | string | FK to same table (hierarchy/tree)                 |
| owner_id    | string | FK to owning user                                 |
| created_by  | string | FK to user who created the record                 |
| updated_by  | string | FK to user who last updated                       |
| deleted_by  | string | FK to user who deleted                            |

### Pagination Fields
| Field       | Type    | Description                                      |
|-------------|---------|--------------------------------------------------|
| page        | integer | 1-indexed page number                            |
| page_size   | integer | Items per page (default 20, max 100)             |
| total       | integer | Total count of all matching records              |
| total_pages | integer | ceil(total / page_size)                          |
| cursor      | string  | Cursor-based pagination token                    |
| sort_by     | string  | Field name to sort by                            |
| sort_order  | string  | asc or desc                                      |
| offset      | integer | Number of records to skip (offset pagination)    |

### API Response Envelope
| Field   | Type    | Description                                        |
|---------|---------|----------------------------------------------------|
| data    | any     | The actual response payload                        |
| error   | object  | Error object when success is false                 |
| message | string  | Human-readable result description                  |
| success | boolean | Whether the request succeeded                      |
| meta    | object  | Pagination, rate limits, and other metadata        |
| code    | string  | Machine-readable code: USER_NOT_FOUND              |

---

## 4. Entity Definitions

### User
| Field        | Type    | Notes                                            |
|--------------|---------|--------------------------------------------------|
| id           | string  | fp_usr_ prefix                                   |
| email        | string  | Unique, lowercase                                |
| username     | string  | Unique, lowercase, 3-30 chars                    |
| first_name   | string  |                                                  |
| last_name    | string  |                                                  |
| full_name    | string  | Computed: first_name + space + last_name         |
| avatar_url   | string  |                                                  |
| phone        | string  | E.164, optional                                  |
| role         | string  | owner, admin, member, viewer, guest              |
| status       | string  | active, inactive, suspended, pending, banned     |
| is_verified  | boolean | Email verified                                   |
| verified_at  | string  | ISO 8601                                         |
| last_seen_at | string  | ISO 8601                                         |
| timezone     | string  | IANA tz                                          |
| language     | string  | BCP 47                                           |
| created_at   | string  | ISO 8601                                         |
| updated_at   | string  | ISO 8601                                         |
| deleted_at   | string  | ISO 8601, null if active                         |

### Organization
| Field        | Type    | Notes                                            |
|--------------|---------|--------------------------------------------------|
| id           | string  | fp_org_ prefix                                   |
| name         | string  |                                                  |
| slug         | string  | URL-safe, globally unique                        |
| description  | string  |                                                  |
| logo_url     | string  |                                                  |
| website_url  | string  |                                                  |
| plan         | string  | free, starter, pro, enterprise                   |
| status       | string  | active, suspended, cancelled                     |
| owner_id     | string  | FK to users                                      |
| member_count | integer | Denormalized count                               |
| created_at   | string  | ISO 8601                                         |
| updated_at   | string  | ISO 8601                                         |

### Project
| Field        | Type    | Notes                                            |
|--------------|---------|--------------------------------------------------|
| id           | string  | fp_proj_ prefix                                  |
| org_id       | string  | FK to organizations                              |
| name         | string  |                                                  |
| slug         | string  | URL-safe, unique within org                      |
| description  | string  |                                                  |
| status       | string  | draft, active, paused, archived, cancelled       |
| visibility   | string  | public, internal, private                        |
| owner_id     | string  | FK to users                                      |
| tech_stack   | array   | ["typescript", "react", "postgres"]              |
| repo_url     | string  | GitHub/GitLab URL                                |
| created_by   | string  | FK to users                                      |
| created_at   | string  | ISO 8601                                         |
| updated_at   | string  | ISO 8601                                         |
| archived_at  | string  | ISO 8601, null if active                         |

### Task
| Field        | Type    | Notes                                            |
|--------------|---------|--------------------------------------------------|
| id           | string  | fp_task_ prefix                                  |
| project_id   | string  | FK to projects                                   |
| title        | string  |                                                  |
| description  | string  |                                                  |
| status       | string  | todo, in_progress, in_review, done, cancelled, blocked |
| priority     | string  | low, medium, high, urgent                        |
| assignee_id  | string  | FK to users                                      |
| reporter_id  | string  | FK to users                                      |
| parent_id    | string  | FK to tasks (subtask support)                    |
| due_at       | string  | ISO 8601                                         |
| completed_at | string  | ISO 8601                                         |
| sort_order   | integer | For manual ordering                              |
| created_by   | string  | FK to users                                      |
| created_at   | string  | ISO 8601                                         |
| updated_at   | string  | ISO 8601                                         |

### API Key
| Field        | Type    | Notes                                            |
|--------------|---------|--------------------------------------------------|
| id           | string  | fp_key_ prefix                                   |
| org_id       | string  | FK to organizations                              |
| name         | string  | Human label                                      |
| key_hash     | string  | Hashed — never store plain                       |
| key_prefix   | string  | First 8 chars shown in UI for identification     |
| scopes       | array   | List of permitted scopes                         |
| status       | string  | active, revoked                                  |
| last_used_at | string  | ISO 8601                                         |
| expires_at   | string  | ISO 8601, null if never expires                  |
| created_by   | string  | FK to users                                      |
| created_at   | string  | ISO 8601                                         |
| revoked_at   | string  | ISO 8601, null if active                         |

### Audit Log
| Field       | Type   | Notes                                             |
|-------------|--------|---------------------------------------------------|
| id          | string | fp_log_ prefix                                    |
| org_id      | string | FK to organizations                               |
| actor_id    | string | FK to users — who performed the action            |
| action      | string | Dot-notation verb: user.created, project.deleted  |
| entity_type | string | Resource type: user, project, task                |
| entity_id   | string | The resource ID                                   |
| changes     | object | Before/after snapshot of changed fields           |
| ip_address  | string | Actor IP at time of action                        |
| user_agent  | string | Browser/client info                               |
| created_at  | string | ISO 8601 — immutable, never updated               |

---

## 5. Standard Enum Values

These are the ONLY accepted values. Never use alternatives.

### User Status: active, inactive, suspended, pending, banned
### Project Status: draft, active, paused, archived, cancelled
### Task Status: todo, in_progress, in_review, done, cancelled, blocked
### Task Priority: low, medium, high, urgent
### User Role: owner, admin, member, viewer, guest
### Visibility: public, internal, private
### Plan: free, starter, pro, enterprise
### Notification Type: info, success, warning, error
### Sort Order: asc, desc
### Environment: development, staging, production
### Log Level: debug, info, warn, error, fatal
### HTTP Method: GET, POST, PUT, PATCH, DELETE

---

## 6. Forbidden Variable Names

Never use these. They are too vague.

| Forbidden | Use Instead                                         |
|-----------|-----------------------------------------------------|
| data      | Specific name: userData, projectList, apiResponse   |
| info      | Specific name: userInfo -> just use user            |
| temp/tmp  | Name what it actually holds                         |
| obj       | Name the entity: user, project, task                |
| item      | Name the entity: user, file, record                 |
| list      | Use plural noun: users, tasks, files                |
| arr       | Use plural noun: ids, names, results                |
| val       | Name what the value represents                      |
| res       | Use response or the entity name                     |
| cb        | Use callback or the specific handler name           |
| e         | Use err for errors, evt for events                  |
| flag      | Name the boolean: isActive, hasPermission           |
| type      | Name it: entityType, contentType, fileType          |
| mode      | Name it: viewMode, editMode, displayMode            |
| misc      | Never — split into proper fields                    |
| other     | Never — name the concept explicitly                 |

---

## 7. API Design Conventions

### URL Structure
- Collections: plural nouns (/users, /projects, /tasks)
- Multi-word paths: kebab-case (/user-profiles, /api-keys)
- No verbs in URLs — HTTP method is the verb
- Always versioned: /api/v1/users

### Standard Query Parameters
| Param      | Type    | Description                              |
|------------|---------|------------------------------------------|
| page       | integer | Page number, 1-indexed                   |
| page_size  | integer | Items per page, default 20, max 100      |
| sort_by    | string  | Field name to sort                       |
| sort_order | string  | asc or desc                              |
| search     | string  | Full-text search query                   |
| include    | string  | Comma-separated relations to expand      |

### Standard Error Codes
| Code               | HTTP | Meaning                                 |
|--------------------|------|-----------------------------------------|
| VALIDATION_ERROR   | 400  | Input failed validation                 |
| UNAUTHORIZED       | 401  | Not authenticated                       |
| FORBIDDEN          | 403  | Authenticated but not permitted         |
| RESOURCE_NOT_FOUND | 404  | Entity does not exist                   |
| CONFLICT           | 409  | Duplicate or state conflict             |
| RATE_LIMIT_EXCEEDED| 429  | Too many requests                       |
| INTERNAL_ERROR     | 500  | Unexpected server error                 |

---

## 8. Database Conventions

- Table names: plural snake_case (users, project_members, audit_logs)
- Column names: snake_case always
- Primary key: id, string ULID, always
- Foreign keys: entity_id pattern (user_id, project_id)
- Required on every table: id, created_at, updated_at
- Soft-deletable tables add: deleted_at (nullable)
- Index all foreign key columns
- Index all columns used in WHERE and ORDER BY
- Never use reserved words as column names (order, user, group, type)

---

## 9. Frontend Conventions

### React
| Concept         | Convention           | Example                   |
|-----------------|----------------------|---------------------------|
| Component       | PascalCase           | UserProfileCard           |
| Component file  | PascalCase.tsx       | UserProfileCard.tsx       |
| Hook            | use + camelCase      | useProjectData            |
| Context         | PascalCase + Context | AuthContext               |
| Event handler   | handle + PascalCase  | handleSubmit, handleClose |
| Boolean prop    | is/has/can prefix    | isLoading, hasError       |
| Callback prop   | on + PascalCase      | onSubmit, onChange        |
| Store/slice     | camelCase            | userStore, projectSlice   |

### CSS Variables (FlatPlanet prefix)
All design tokens use --fp- prefix: --fp-color-primary, --fp-spacing-md

---

## 10. Environment Variables

| Category      | Pattern          | Example                        |
|---------------|------------------|--------------------------------|
| App config    | FP_ prefix       | FP_APP_URL, FP_ENV             |
| Database      | DB_ prefix       | DB_URL, DB_POOL_SIZE           |
| Auth          | AUTH_ prefix     | AUTH_SECRET, AUTH_EXPIRY       |
| External APIs | SERVICE_API_KEY  | STRIPE_API_KEY, RESEND_API_KEY |
| Feature flags | FF_ prefix       | FF_ENABLE_DARK_MODE            |
| Infrastructure| Bare uppercase   | PORT, NODE_ENV, LOG_LEVEL      |

Required in every project:
- NODE_ENV: development, staging, production
- FP_APP_URL: Full base URL
- FP_ENV: development, staging, production
- LOG_LEVEL: debug, info, warn, error

---

## 11. Logging Standards

### Log Levels
| Level | When to Use                                            |
|-------|--------------------------------------------------------|
| debug | Dev only, never in production                          |
| info  | Normal ops: request received, job completed            |
| warn  | Unexpected but recoverable: retry, fallback triggered  |
| error | Failures needing attention: exceptions, data loss      |
| fatal | System unusable: crash, unrecoverable state            |

### Required Log Fields
| Field       | Type    | Description                                      |
|-------------|---------|--------------------------------------------------|
| level       | string  | Log level                                        |
| message     | string  | Human-readable description                       |
| timestamp   | string  | ISO 8601 UTC                                     |
| service     | string  | Service/app name                                 |
| request_id  | string  | Unique ID to trace request across services       |
| user_id     | string  | Acting user if available                         |
| org_id      | string  | Acting org if available                          |
| duration_ms | integer | Operation duration in milliseconds               |
| error       | object  | Error details when level is error or fatal       |

---

## 12. Feature Flag Conventions

| Field       | Type    | Description                                       |
|-------------|---------|---------------------------------------------------|
| key         | string  | FF_ prefix UPPER_SNAKE: FF_ENABLE_DARK_MODE        |
| enabled     | boolean | Global on/off switch                              |
| rollout_pct | float   | 0.0-1.0 percentage of users who see it           |
| target_ids  | array   | Specific user or org IDs to enable for            |
| description | string  | What it controls and why it exists               |
| owner       | string  | Team or person responsible                        |
| expires_at  | string  | ISO 8601. When flag should be cleaned up.         |

---

Last updated: 2026-03-18
Maintained by: FlatPlanet-Hub
Repository: https://github.com/FlatPlanet-Hub/FLATPLANET-STANDARDS
To propose changes: open a PR with additions following this format
