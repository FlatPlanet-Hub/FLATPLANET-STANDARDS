# FP Mission Control — Project Specification
**Version:** 1.0
**Status:** Awaiting build — Phase 1 not yet started
**Repo (to be created):** FlatPlanet-Hub/FP-MISSION-CONTROL
**Prepared by:** Claude Code — Session 1
**Date:** 2026-03-19

---

## 1. Vision

FP Mission Control is FlatPlanet's central project intelligence system. It gives leadership real-time visibility into every project across the organisation — what is being built, who is building it, how far along it is, what the blockers are, and when things are due.

It is not a manual project management tool. It is fed automatically by Claude at the end of every development session. No human data entry is required for status updates. Claude writes the data; humans read the dashboard.

---

## 2. Core Principles

- **Claude is the data entry mechanism.** At the end of every session, Claude updates Mission Control automatically as part of the session close protocol.
- **No manual status updates.** If it requires a human to type a status update, it is not working correctly.
- **One source of truth.** The Supabase database is canonical. The GitHub markdown backup is human-readable but secondary.
- **Properly architected from day one.** Built to scale to any number of projects, teams, and business units.
- **Deadlines and planning are first-class.** Not an afterthought — objectives have deadlines, checklists have owners, timelines are visible from day one.

---

## 3. System Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Claude Session                     │
│  (any FlatPlanet project — Tala, FP-DATA-DICTIONARY │
│   FP-MISSION-CONTROL, or any future project)        │
└───────────────────┬─────────────────────────────────┘
                    │ Session ends
                    ▼
┌─────────────────────────────────────────────────────┐
│              Session Close Protocol                  │
│  Claude writes structured data to:                  │
│  1. FP Mission Control API (primary)                │
│  2. GitHub /projects/ folder (markdown backup)      │
└───────────┬─────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────┐
│           FP Mission Control API                     │
│  Node.js / TypeScript                               │
│  Deployed on Netlify Functions                      │
│  Repo: FlatPlanet-Hub/FP-MISSION-CONTROL            │
└───────────┬─────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────┐
│              Supabase Database                       │
│  Dedicated FlatPlanet infrastructure instance       │
│  Shared with FP-DATA-DICTIONARY                     │
└───────────┬─────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────┐
│           Mission Control Dashboard                  │
│  React web app — hosted on Netlify                  │
│  Reads from Supabase in real time                   │
│  Repo: FlatPlanet-Hub/FP-MISSION-CONTROL            │
└─────────────────────────────────────────────────────┘
```

---

## 4. Data Model

### 4.1 Table: `projects`
The master registry of all FlatPlanet projects.

| Column | Type | Notes |
|---|---|---|
| id | text (PK) | Prefix: `fp_proj_` + ULID |
| name | text | Display name e.g. "Tala On Demand" |
| slug | text (unique) | URL-safe e.g. `tala-ondemand` |
| description | text | One paragraph summary |
| status | text | `active` \| `paused` \| `blocked` \| `complete` \| `archived` |
| repo_url | text | GitHub repo URL |
| deploy_url | text | Live URL if applicable |
| business_unit | text | e.g. "On Demand", "FlatPlanet Hub" |
| tech_stack | text[] | Array of technologies |
| current_version | text | e.g. "v1.7.1" |
| created_at | timestamptz | |
| updated_at | timestamptz | Auto-updated |

---

### 4.2 Table: `objectives`
Goals that a project is working toward. Each objective has a deadline.

| Column | Type | Notes |
|---|---|---|
| id | text (PK) | Prefix: `fp_obj_` + ULID |
| project_id | text (FK) | → projects.id |
| title | text | e.g. "Invoice System Complete" |
| description | text | What done looks like |
| status | text | `not_started` \| `in_progress` \| `complete` \| `cancelled` |
| due_at | date | Target completion date |
| completed_at | timestamptz | Set when status → complete |
| sort_order | integer | Display order within project |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### 4.3 Table: `checklist_items`
Discrete tasks under an objective. Completion drives the progress percentage.

| Column | Type | Notes |
|---|---|---|
| id | text (PK) | Prefix: `fp_chk_` + ULID |
| objective_id | text (FK) | → objectives.id |
| project_id | text (FK) | → projects.id (denormalised for query speed) |
| title | text | e.g. "Build Supabase schema" |
| status | text | `todo` \| `in_progress` \| `done` \| `cancelled` |
| completed_at | timestamptz | Set when status → done |
| sort_order | integer | Display order within objective |
| created_at | timestamptz | |
| updated_at | timestamptz | |

---

### 4.4 Table: `sessions`
One row per Claude session per project. The living history of project work.

| Column | Type | Notes |
|---|---|---|
| id | text (PK) | Prefix: `fp_ses_` + ULID |
| project_id | text (FK) | → projects.id |
| session_number | integer | Incrementing per project |
| session_date | date | Date of session |
| summary | text | What was accomplished this session |
| decisions | text | Key decisions made |
| next_steps | text | What happens next session |
| version_at_close | text | Version number at session end |
| created_at | timestamptz | |

---

### 4.5 Table: `blockers`
Anything preventing progress. Flagged by Claude, cleared by Claude.

| Column | Type | Notes |
|---|---|---|
| id | text (PK) | Prefix: `fp_blk_` + ULID |
| project_id | text (FK) | → projects.id |
| description | text | What is blocked and why |
| status | text | `open` \| `resolved` |
| raised_at | timestamptz | |
| resolved_at | timestamptz | Set when status → resolved |
| resolution_note | text | How it was resolved |

---

### 4.6 Computed Values (no stored columns needed)

| Value | Calculation |
|---|---|
| Objective % complete | `done checklist_items / total checklist_items` × 100 |
| Project % complete | Average of all objective % complete values |
| Days until due | `objective.due_at - today` |
| Overdue | `due_at < today AND status != complete` |

---

## 5. API Design

### 5.1 Base URL
```
https://fp-mission-control.netlify.app/.netlify/functions/
```

### 5.2 Endpoints

#### Projects
```
GET    /projects                  — list all projects
GET    /projects/:slug            — get project + objectives + latest session
POST   /projects                  — create project
PATCH  /projects/:id              — update project status, version, etc.
```

#### Objectives
```
GET    /projects/:id/objectives   — list objectives for a project
POST   /projects/:id/objectives   — create objective
PATCH  /objectives/:id            — update status, due date, etc.
```

#### Checklist Items
```
GET    /objectives/:id/checklist  — list items for an objective
POST   /objectives/:id/checklist  — add item
PATCH  /checklist/:id             — mark done/todo/cancelled
```

#### Sessions
```
POST   /sessions                  — log a session (called at session end)
GET    /projects/:id/sessions     — session history for a project
```

#### Blockers
```
POST   /blockers                  — raise a blocker
PATCH  /blockers/:id              — resolve a blocker
GET    /projects/:id/blockers     — all blockers for a project (open first)
```

#### Dashboard
```
GET    /dashboard                 — all projects with latest session + open blockers + objective progress
```

### 5.3 Authentication
All write endpoints (`POST`, `PATCH`) require a `x-api-key` header. API key stored in Netlify environment variables. Read endpoints (`GET`) are public (dashboard is read-only).

---

## 6. Claude Session Close Protocol

At the end of every session on any FlatPlanet project, Claude MUST:

1. **Update project status** — `PATCH /projects/:id` with current status and version
2. **Tick completed checklist items** — `PATCH /checklist/:id` for each item completed this session
3. **Log the session** — `POST /sessions` with summary, decisions, next steps
4. **Raise any new blockers** — `POST /blockers` for anything blocking progress
5. **Resolve cleared blockers** — `PATCH /blockers/:id` for anything unblocked this session
6. **Push markdown backup** — commit updated project status file to `FLATPLANET-STANDARDS/projects/{slug}.md`

### 6.1 Session Log Markdown Format (GitHub backup)

Every project maintains a file at `FLATPLANET-STANDARDS/projects/{slug}.md`:

```markdown
# {Project Name} — Status Report
**Last updated:** {date}
**Version:** {version}
**Status:** {status}

## Current Objectives
| Objective | Progress | Due | Status |
|---|---|---|---|
| {title} | {x}% | {date} | {status} |

## Open Blockers
- {blocker description}

## Latest Session ({number}) — {date}
### What was done
{summary}

### Decisions made
{decisions}

### Next session
{next_steps}

## Session History
| Session | Date | Summary |
|---|---|---|
| #{n} | {date} | {one-liner} |
```

---

## 7. Dashboard Design

### 7.1 Project Cards View (default)
One card per project. Each card shows:
- Project name + status badge
- Business unit
- Current version
- Overall % complete (progress bar)
- Nearest objective title + deadline + days remaining
- Open blocker count (red badge if > 0)
- Last session date

### 7.2 Project Drill-Down
Click a project card to see:
- Full objective list with progress bars and deadlines
- Checklist items per objective (expandable)
- Open blockers
- Session history (most recent first)

### 7.3 Planning View (Timeline)
All objectives across all projects, sorted by deadline. Colour coded:
- 🟢 Green — on track (> 14 days remaining)
- 🟡 Amber — approaching (≤ 14 days remaining)
- 🔴 Red — overdue
- ⬛ Grey — complete

### 7.4 Blockers View
All open blockers across all projects in one place. Sorted by age (oldest first).

### 7.5 Design Constraints
- Dark theme (consistent with FlatPlanet standards)
- Mobile responsive
- No login required (read-only dashboard is public)
- Real-time updates via Supabase subscriptions

---

## 8. Build Phases

### Phase 1 — Supabase Schema (START HERE)
- [ ] Create FlatPlanet Supabase instance (if not already done for FP-DATA-DICTIONARY)
- [ ] Create `projects` table
- [ ] Create `objectives` table
- [ ] Create `checklist_items` table
- [ ] Create `sessions` table
- [ ] Create `blockers` table
- [ ] Enable Row Level Security
- [ ] Create read-only public role
- [ ] Create write role (API key gated)
- [ ] Seed with existing FlatPlanet projects (Tala, FP-DATA-DICTIONARY, FP-MISSION-CONTROL)

### Phase 2 — API
- [ ] Create `FlatPlanet-Hub/FP-MISSION-CONTROL` repo
- [ ] Set up Node.js / TypeScript project structure
- [ ] Implement `/projects` endpoints
- [ ] Implement `/objectives` endpoints
- [ ] Implement `/checklist` endpoints
- [ ] Implement `/sessions` endpoint
- [ ] Implement `/blockers` endpoints
- [ ] Implement `/dashboard` aggregation endpoint
- [ ] Deploy to Netlify Functions
- [ ] Add API key authentication to write endpoints
- [ ] Write API documentation

### Phase 3 — Claude Session Protocol
- [ ] Write CLAUDE.md session close instructions
- [ ] Add API call block to FLATPLANET-STANDARDS CLAUDE.md
- [ ] Test session close write on a live project (Tala first)
- [ ] Verify GitHub markdown backup write
- [ ] Create `/projects/` folder in FLATPLANET-STANDARDS repo
- [ ] Add initial status files for existing projects

### Phase 4 — Dashboard (React)
- [ ] Set up React project in FP-MISSION-CONTROL repo
- [ ] Build project card grid
- [ ] Build project drill-down view
- [ ] Build planning / timeline view
- [ ] Build blockers view
- [ ] Connect to Supabase real-time subscriptions
- [ ] Deploy to Netlify
- [ ] Verify live dashboard updates when sessions are logged

### Phase 5 — Rollout
- [ ] Tala — add session close protocol
- [ ] FP-DATA-DICTIONARY — add session close protocol
- [ ] FP-MISSION-CONTROL — add session close protocol (self-referential)
- [ ] Document onboarding process for new projects
- [ ] Add to FLATPLANET-STANDARDS as a required standard

---

## 9. Integration with FP-DATA-DICTIONARY

FP Mission Control and FP-DATA-DICTIONARY share the same Supabase instance but are completely separate schemas. Mission Control does not depend on the Data Dictionary and vice versa. They may reference each other via project records.

---

## 10. Future Phases (not in scope for initial build)

- **Notifications** — Slack/email alerts when a blocker is raised or a deadline is missed
- **Business unit rollup** — aggregate progress by business unit (On Demand, FlatPlanet Hub, etc.)
- **Dependency mapping** — mark when one project is blocked by another
- **Budget tracking** — link objectives to estimated vs actual hours
- **Client visibility** — filtered read-only view per client showing only their relevant projects

---

## 11. Open Questions

1. Is the FlatPlanet Supabase instance already created (for FP-DATA-DICTIONARY), or does it need to be set up fresh?
2. What is the Netlify account / team the FP-MISSION-CONTROL app should deploy under?
3. Should the dashboard be password protected, or is public read-only acceptable for now?
4. Priority order — is FP-MISSION-CONTROL higher priority than FP-DATA-DICTIONARY, or do they run in parallel?

---

*This document is the source of truth for FP Mission Control. All implementation decisions should be checked against it. Update this document as decisions are made.*
