# FP Mission Control
## Project Brief & Implementation Checklist

**A real-time project dashboard for the FlatPlanet ecosystem**

> v1.0 · 2026-03-19 · FlatPlanet-Hub

---

| | |
|---|---|
| **Service name** | FP Mission Control |
| **Repository** | https://github.com/FlatPlanet-Hub/FP-MISSION-CONTROL |
| **Database** | Supabase (shared with FP Data Dictionary — same dedicated FlatPlanet instance) |
| **Dashboard hosting** | Netlify |
| **Who writes to it** | Claude Code only — at the end of every session, automatically |
| **Who reads it** | Anyone at FlatPlanet — dashboard is accessible to all |
| **Why it exists** | Give FlatPlanet leadership real-time visibility into every project — what is being built, how far along it is, what the blockers are, and when things are due — without anyone having to manually report status |

---

## 1. The Problem This Solves

FlatPlanet is scaling from one project to many projects running in parallel across the business. Business line managers are experimenting and spinning up new projects. Without a central view, leadership has no visibility into what is happening across the portfolio.

> **Core problem:** Status reporting is manual, inconsistent, and always out of date. Nobody has time to write status reports. When they do, they are already stale.

| Approach | What happens |
|---|---|
| **Manual status reporting** | People forget. Reports are inconsistent. Language varies. Leadership chases updates. Information is always behind reality. |
| **FP Mission Control (this)** | Claude writes the status at the end of every session as a matter of standard process. The dashboard updates automatically. Leadership sees the current state of every project without asking anyone. |

The key insight: **Claude is already doing the work at session end**. Mission Control simply captures that work as structured data and makes it visible.

---

## 2. What FP Mission Control Is

FP Mission Control is a web dashboard backed by a Supabase database. It shows the live state of every FlatPlanet project in one place.

At the end of every Claude Code session, Claude writes structured data to the Mission Control API — project status, session summary, objective progress, checklist completions, and any blockers. The dashboard reflects this immediately.

There is no manual data entry. There is no separate reporting process. The act of closing a Claude session is the act of updating the dashboard.

| What you can see | Detail |
|---|---|
| All projects at a glance | Name, status, version, % complete, nearest deadline, last session date |
| Project drill-down | Full objectives list with deadlines and progress, checklist items, session history, open blockers |
| Planning timeline | All objectives across all projects on a single timeline, sorted by deadline |
| Blockers board | Everything currently blocked across all projects in one view |
| Session history | Every session logged — what was done, what version closed, who worked on it |

---

## 3. Architecture

### Separation of concerns

FP Mission Control is a separate repository and service from FP Data Dictionary. FP Data Dictionary is a read-only standards service — it tells Claude what names and types to use. FP Mission Control is a write-capable project management service — it records what Claude did. They share the same Supabase instance but have separate schemas and separate APIs.

```
FP-DATA-DICTIONARY        FP-MISSION-CONTROL
  Read-only API       +     Read/write API
  Standards data            Project management data
       ↑                          ↑  ↓
  Claude reads              Claude writes (session end)
  before coding             Dashboard reads (real-time)
```

### Database — Supabase

Shared FlatPlanet Supabase instance. Mission Control data lives in its own tables, separate from the standards tables.

#### Core tables

| Table | Purpose |
|---|---|
| `projects` | One row per FlatPlanet project — name, status, repo, owner, current version |
| `objectives` | Goals belonging to a project — what the project is trying to achieve, with a deadline |
| `checklist_items` | Tasks under an objective — each is either done or not done |
| `sessions` | One row per Claude Code session — date, version at close, summary, worked on by |
| `session_updates` | Individual bullet points from a session — what was done |
| `blockers` | Open issues flagged by Claude — description, severity, opened and resolved dates |

#### Schema detail

**projects**
| Field | Type | Notes |
|---|---|---|
| id | string | fp_proj_ prefixed ULID |
| name | string | Human display name |
| slug | string | kebab-case, unique |
| description | string | 2-3 sentences, plain English |
| status | string | active, paused, blocked, complete, archived |
| repo_url | string | GitHub URL |
| dashboard_url | string | Live URL if deployed |
| owner | string | Role title — never a personal name |
| current_version | string | MAJOR.MINOR.PATCH |
| created_at | string | ISO 8601 |
| updated_at | string | ISO 8601 |

**objectives**
| Field | Type | Notes |
|---|---|---|
| id | string | fp_obj_ prefixed ULID |
| project_id | string | FK to projects |
| title | string | Plain English goal statement |
| description | string | More detail if needed |
| status | string | not_started, in_progress, complete, cancelled |
| due_at | string | ISO 8601 — the deadline |
| sort_order | integer | Manual ordering within a project |
| completed_at | string | ISO 8601 — when status moved to complete |
| created_at | string | ISO 8601 |
| updated_at | string | ISO 8601 |

**checklist_items**
| Field | Type | Notes |
|---|---|---|
| id | string | fp_chk_ prefixed ULID |
| objective_id | string | FK to objectives |
| project_id | string | FK to projects — denormalised for query performance |
| title | string | Plain English task description |
| status | string | todo, done |
| sort_order | integer | Manual ordering within an objective |
| completed_at | string | ISO 8601 — when ticked off |
| created_at | string | ISO 8601 |
| updated_at | string | ISO 8601 |

**sessions**
| Field | Type | Notes |
|---|---|---|
| id | string | fp_ses_ prefixed ULID |
| project_id | string | FK to projects |
| session_number | integer | Incrementing per project |
| session_date | string | ISO 8601 date |
| version_at_close | string | MAJOR.MINOR.PATCH |
| summary | string | Current state of the project at close |
| worked_on_by | string | Claude model version or role title |
| created_at | string | ISO 8601 |

**session_updates**
| Field | Type | Notes |
|---|---|---|
| id | string | fp_upd_ prefixed ULID |
| session_id | string | FK to sessions |
| project_id | string | FK to projects |
| description | string | One plain English bullet point |
| sort_order | integer | Order within the session |
| created_at | string | ISO 8601 |

**blockers**
| Field | Type | Notes |
|---|---|---|
| id | string | fp_blk_ prefixed ULID |
| project_id | string | FK to projects |
| description | string | Plain English — what is blocked and why |
| severity | string | low, medium, high, critical |
| status | string | open, resolved |
| opened_at | string | ISO 8601 |
| resolved_at | string | ISO 8601 — null if still open |
| created_at | string | ISO 8601 |
| updated_at | string | ISO 8601 |

### API — REST Endpoints

The Mission Control API handles writes from Claude and reads from the dashboard. All Claude writes happen at session end via a single session-close call.

| Endpoint | Method | What it does |
|---|---|---|
| `/v1/projects` | GET | List all projects with current status and progress |
| `/v1/projects/:slug` | GET | Full project detail — objectives, checklist, sessions, blockers |
| `/v1/projects` | POST | Register a new project (first session only) |
| `/v1/projects/:slug` | PATCH | Update project status, version, description |
| `/v1/projects/:slug/objectives` | GET | List objectives for a project |
| `/v1/projects/:slug/objectives` | POST | Add a new objective |
| `/v1/objectives/:id` | PATCH | Update objective status, deadline, sort order |
| `/v1/objectives/:id/checklist` | GET | List checklist items for an objective |
| `/v1/objectives/:id/checklist` | POST | Add a checklist item |
| `/v1/checklist/:id` | PATCH | Tick or untick a checklist item |
| `/v1/projects/:slug/sessions` | POST | Log a new session — the primary session-end write |
| `/v1/projects/:slug/blockers` | GET | List open blockers |
| `/v1/projects/:slug/blockers` | POST | Add a blocker |
| `/v1/blockers/:id` | PATCH | Resolve a blocker |
| `/v1/planning` | GET | All objectives across all projects sorted by due_at |

### Session-close write

When Claude closes a session, it makes one structured API call to `/v1/projects/:slug/sessions` with a payload that includes the session summary, version at close, list of updates (bullets), checklist items to mark done, objective status changes, and any new or resolved blockers. The API handles all the writes atomically.

Claude never makes multiple separate calls at session end. One call. One payload. Done.

### Dashboard — React on Netlify

A web application reading from the Mission Control API. No authentication required for viewing — the dashboard is visible to anyone at FlatPlanet.

**Views:**

1. **Portfolio view** — grid of all project cards. Each card shows: project name, status badge, owner, current version, overall % complete (computed from checklist items), nearest objective deadline, days since last session.

2. **Project view** — drill into a single project. Shows: project description, all objectives with progress bars and deadlines, checklist items per objective, open blockers, session history timeline.

3. **Planning view** — horizontal timeline. All objectives from all projects, plotted by deadline. Colour-coded by project. Shows what is due when across the entire portfolio.

4. **Blockers view** — table of all open blockers across all projects, sorted by severity. One place to see everything that is stuck.

---

## 4. How Claude Uses Mission Control

Every FlatPlanet project CLAUDE.md includes the Mission Control API endpoint and the session-close instructions.

### Session open (first session only — registering a new project)

Call `POST /v1/projects` with the project name, slug, repo URL, owner, and description. This registers the project in Mission Control. Objectives and checklist items are added in this same session or as they become known.

### During a session

Claude may call `POST /v1/projects/:slug/blockers` if a blocker is discovered mid-session. Everything else waits for session close.

### Session close (every session, without exception)

Claude calls `POST /v1/projects/:slug/sessions` with:

```json
{
  "session_number": 42,
  "session_date": "2026-03-19",
  "version_at_close": "1.7.1",
  "worked_on_by": "claude-sonnet-4-6",
  "summary": "Current state of the project in 2-5 plain English sentences.",
  "updates": [
    "Fixed the utilisation calculation to exclude absent staff",
    "Updated index.html to v1.7.1"
  ],
  "checklist_completions": ["fp_chk_01j9x...", "fp_chk_01j9y..."],
  "objective_updates": [
    { "id": "fp_obj_01j9x...", "status": "in_progress" }
  ],
  "resolved_blockers": ["fp_blk_01j9x..."],
  "new_blockers": [
    { "description": "Awaiting Supabase credentials from Chris", "severity": "high" }
  ]
}
```

The session is not closed until this call succeeds and returns `200`. Claude confirms the write before ending the session.

---

## 5. How Objectives and Checklists Work

### Objectives

An objective is a meaningful goal with a deadline. It is not a task — it is an outcome. Examples:

- *"Invoice system fully operational"* — due 2026-04-30
- *"REST API deployed and tested"* — due 2026-04-15
- *"Admin UI live for developers"* — due 2026-05-15

Objectives are created by Claude when a project is set up, or added by a developer via the admin UI. They are never invented mid-session without the human's knowledge — Claude proposes, the human confirms.

### Checklist items

Checklist items are the tasks that must be completed for an objective to be achieved. Claude ticks them off as work is completed. Examples under *"Invoice system fully operational"*:

- [ ] Stage 0 — DB foundation complete
- [ ] Stage 1 — Timesheet reports complete
- [ ] Stage 2 — Goodwill requests complete
- [x] Stage 3 — Invoice generation complete
- [ ] Stage 4 — Lock wiring complete
- [ ] Stage 5 — Finance view complete

Progress = 1/6 = 17% complete.

### Progress computation

- Objective % = checklist items done / total checklist items
- Project % = average across all active objectives (excluding cancelled)
- Both are computed on read — never stored

---

## 6. Build Phases

### Phase 1 — Supabase Schema `[HIGHEST PRIORITY — start here]`

Build the database. Everything else depends on it. Create all tables with correct types, constraints, and row-level security. No seed data needed — projects register themselves on first session.

### Phase 2 — REST API `[HIGH]`

Build the Mission Control API in the FP-MISSION-CONTROL repo. Node.js/TypeScript. All endpoints from the architecture section. The session-close endpoint is the most important — get this right. Deploy to a stable Netlify function URL.

### Phase 3 — CLAUDE.md Integration Pattern `[HIGH]`

Write the standard CLAUDE.md block every project includes. Define the session-close payload format. Test end-to-end with the TALA project — register TALA in Mission Control, run a mock session close, confirm the data appears correctly.

### Phase 4 — Dashboard `[MEDIUM]`

Build the React dashboard on Netlify. Portfolio view first — get all projects visible. Then project drill-down. Planning timeline and blockers view come after.

### Phase 5 — Planning & Deadlines `[MEDIUM]`

Build the planning timeline view. Add deadline warning indicators to project cards (e.g. red if an objective is overdue, amber if due within 7 days). Add the ability for developers to set and update deadlines via the admin UI without needing Claude.

### Phase 6 — Notifications `[LOW — future]`

Slack or email notification when: a project is blocked, a deadline is missed, a project has had no session in X days. Keeps leadership informed without having to check the dashboard.

---

## 7. Implementation Checklist

### Phase 1 — Supabase Schema

- [ ] Create `projects` table with all fields per schema
- [ ] Create `objectives` table with all fields per schema
- [ ] Create `checklist_items` table with all fields per schema
- [ ] Create `sessions` table with all fields per schema
- [ ] Create `session_updates` table with all fields per schema
- [ ] Create `blockers` table with all fields per schema
- [ ] Index all foreign key columns
- [ ] Index `objectives.due_at` — used in planning view ORDER BY
- [ ] Index `blockers.status` — used in blockers view filter
- [ ] Index `sessions.project_id` and `sessions.session_date`
- [ ] Apply row-level security — read: public, write: authenticated service role only
- [ ] Test all table relationships with sample data

### Phase 2 — REST API

- [ ] Initialise Node.js/TypeScript project in FlatPlanet-Hub/FP-MISSION-CONTROL repo
- [ ] Connect to Supabase using service role key
- [ ] Build `GET /v1/projects` with computed progress fields
- [ ] Build `GET /v1/projects/:slug` full detail endpoint
- [ ] Build `POST /v1/projects` — register new project
- [ ] Build `PATCH /v1/projects/:slug` — update project status and version
- [ ] Build `POST /v1/projects/:slug/objectives` — add objective
- [ ] Build `PATCH /v1/objectives/:id` — update objective
- [ ] Build `POST /v1/objectives/:id/checklist` — add checklist item
- [ ] Build `PATCH /v1/checklist/:id` — tick or untick item
- [ ] Build `POST /v1/projects/:slug/sessions` — session-close write (atomic, handles all sub-writes)
- [ ] Build `POST /v1/projects/:slug/blockers` — add blocker
- [ ] Build `PATCH /v1/blockers/:id` — resolve blocker
- [ ] Build `GET /v1/planning` — all objectives sorted by due_at
- [ ] Add API versioning — `/v1/` prefix on all routes
- [ ] Add rate limiting
- [ ] Deploy to stable URL via Netlify Functions
- [ ] Write API documentation

### Phase 3 — CLAUDE.md Integration

- [ ] Write the standard CLAUDE.md Mission Control block — API URL, session-close instructions, payload format
- [ ] Define the full session-close JSON payload format
- [ ] Test with TALA — register TALA as first project in Mission Control
- [ ] Run a test session close — confirm all data writes correctly
- [ ] Update TALA's CLAUDE.md with live Mission Control API endpoint
- [ ] Document the integration pattern for all future projects

### Phase 4 — Dashboard

- [ ] Scaffold React project in FP-MISSION-CONTROL repo (same repo, `/dashboard` folder)
- [ ] Connect to Mission Control API
- [ ] Build portfolio view — grid of project cards with status, progress, nearest deadline
- [ ] Build project detail view — objectives, checklist, sessions, blockers
- [ ] Build blockers view — all open blockers across all projects
- [ ] Deploy to Netlify
- [ ] Set up auto-deploy on push to main

### Phase 5 — Planning & Deadlines

- [ ] Build planning timeline view — all objectives plotted by deadline
- [ ] Add deadline warning indicators — overdue (red), due within 7 days (amber)
- [ ] Add ability to set/update deadlines without Claude via dashboard
- [ ] Add objective completion % to project cards

### Phase 6 — Notifications (future)

- [ ] Define notification triggers — blocked, overdue, no session in X days
- [ ] Integrate with Slack or email
- [ ] Add notification preferences per project

---

## 8. Decisions Already Made — Do Not Re-Open

| Decision | What was decided |
|---|---|
| Separate repo from FP Data Dictionary | Mission Control is its own service — FP Data Dictionary stays read-only |
| Shared Supabase instance | Both services use the same dedicated FlatPlanet Supabase instance, separate schemas |
| Claude is the only writer | No manual status entry — Claude writes at session end, humans manage objectives and deadlines via admin UI |
| One session-close API call | Claude makes one atomic call at session end — not multiple separate writes |
| Progress computed on read | Checklist % and project % are never stored — always computed from live data |
| Dashboard is public within FlatPlanet | No login required to view — visibility is the goal |
| Objectives owned by humans | Claude proposes objectives, humans confirm — Claude never invents objectives without instruction |

---

*FP Mission Control v1.0 · 2026-03-19 · FlatPlanet-Hub*
