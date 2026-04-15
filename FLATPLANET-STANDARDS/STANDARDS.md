# FLATPLANET Standards
> Version: 3.2 | Last updated: 2026-04-15
> Repository: https://github.com/FlatPlanet-Hub/FLATPLANET-STANDARDS

---

## What This Document Is

This is the single operating standard for every project built at FlatPlanet. It covers how projects are structured, how code is written, how documentation is maintained, and how version control works.

Claude reads this file at the start of every project. Everything Claude builds at FlatPlanet must follow these rules.

---

## 1. The Golden Rules

1. One source of truth: Every project has one README and one CHANGELOG. Information does not live in emails, chats, or scattered documents.
2. Write it down: If a decision was made, it goes in the CHANGELOG. If the project changed, the README is updated. If neither happened, it did not officially happen.
3. Plain language first: All documentation is written so that any FlatPlanet member can read and understand it.
4. Claude maintains the docs: Documentation is not written manually. Claude updates it. But someone is always responsible for making sure it gets done.
5. Version everything: Every meaningful change gets a version number and a changelog entry. No exceptions.

---

## 2. How to Use Claude on Any Project

Before asking Claude anything about a FlatPlanet project, always start with this:

"Please read the FlatPlanet standards: https://raw.githubusercontent.com/FlatPlanet-Hub/FLATPLANET-STANDARDS/main/FLATPLANET-STANDARDS/STANDARDS.md -- then I will tell you what I need."

This ensures Claude knows all FlatPlanet rules before it starts working.

> **Data Dictionary:** The FlatPlanet data dictionary is no longer a static file. It lives in Supabase and is accessible via the HubApi DB Proxy (`GET /api/projects/{id}/schema/full`). Claude should always query the schema before naming tables, columns, or variables.

### Prompt Templates

Copy and paste these. Replace anything in [brackets] with your information.

To start a new project:
"Please read the FlatPlanet standards. We are starting a new project called [name]. It does [description]. The tech stack is [stack]. Please set up the project structure following FlatPlanet standards, including README.md and CHANGELOG.md."

To add a feature:
"We need to add [feature description] to [project name]. Please follow FlatPlanet standards, update the CHANGELOG.md with version [X.X.X], and update the README.md if the feature changes how the project is used."

To fix a bug:
"There is a bug in [project name]: [description]. Please fix it following FlatPlanet standards and update the CHANGELOG.md with a patch version entry."

To update documentation:
"Please update the README.md for [project name] to reflect [what changed]. Keep it in plain language. The current version is [X.X.X]."

To get a project summary:
"Please read the README.md and CHANGELOG.md for [project name] and give me a plain English summary of what this project does, what version it is on, what changed recently, and any known issues."

To generate a report:
"Here is data from [source]: [paste data]. Please create a clear summary -- what the numbers mean, what is going well, what needs attention, and what action is recommended."

To check if documentation is up to date:
"Please review the README.md and CHANGELOG.md for [project name]. Are they accurate and up to date? Give me a checklist of what needs to be updated."

---

## 3. Version Control

Version control is not optional and it is not flexible. Every FlatPlanet project follows these rules exactly. Uniformity across all projects is the goal — the same rules apply whether the project is two weeks old or two years old.

### Version Numbers

Format: `MAJOR.MINOR.PATCH`

| Number | When it changes | Example |
|--------|-----------------|---------|
| MAJOR | Fundamental change — breaks compatibility with the previous version | 1.0.0 → 2.0.0 |
| MINOR | New feature added that does not break anything existing | 1.0.0 → 1.1.0 |
| PATCH | Bug fix, copy change, or small update | 1.0.0 → 1.0.1 |

- Every project starts at `1.0.0`
- Version numbers only go up — never reset, never skip
- The version in `README.md` and `CHANGELOG.md` must always match
- Claude increments the version at the end of every session that changes the project

### Branches

Every project has exactly these branches. No others.

| Branch | Purpose | Who pushes here |
|--------|---------|-----------------|
| `main` | Production — always deployable, always protected | Nobody directly. PRs from `dev` only. |
| `dev` | Integration — all work lands here first | Claude via PRs from feature/fix/docs branches |
| `feature/short-name` | One feature or one Claude session | Claude |
| `fix/short-name` | Bug fixes | Claude |
| `docs/short-name` | Documentation-only changes | Claude |

Rules:
- Nobody pushes directly to `main` — ever. Not developers, not Claude.
- `main` only receives merges from `dev` via a pull request
- Every PR into `main` requires at least one human review and all tests passing
- Feature, fix, and docs branches are deleted after merging — they do not accumulate
- Branch names are lowercase kebab-case: `feature/invoice-export`, not `feature/InvoiceExport`

### Commits

Every commit message follows this format exactly:

  type: short description (max 72 characters)

Types:
- `feat` — new feature
- `fix` — bug fix
- `docs` — documentation only
- `style` — formatting, no logic change
- `refactor` — restructure without behaviour change
- `test` — adding or fixing tests
- `chore` — dependencies, config, tooling

Examples:
  feat: add monthly report export to operations dashboard
  fix: resolve export button not responding on Safari
  docs: update README with new setup steps
  chore: upgrade all dependencies to latest stable

Rules:
- One logical change per commit — do not bundle unrelated changes
- Present tense, lowercase, no full stop at the end
- If a commit needs more than 72 characters to describe, it should probably be two commits

### Pull Requests

- Every change to `dev` or `main` goes through a pull request — no exceptions
- PR title follows the same format as commit messages: `type: short description`
- PR description must explain what changed and why in plain English
- All tests must pass before a PR can be merged
- PRs into `main` require at least one human approval
- Claude writes the PR description — it must be readable by a non-developer

### 3.x — Raising Pull Requests from Claude Code

When the `gh` CLI is not available, Claude must raise pull requests via the GitHub API using credentials from the git credential manager. Do not tell the user to do it manually. Do not protest that it cannot be done.

The correct approach:

1. Retrieve credentials:
```bash
git credential fill <<'EOF'
protocol=https
host=github.com
EOF
```

2. Use the token from the `password` field to call the GitHub API:
```bash
curl -s -X POST \
  -H "Authorization: token <token>" \
  -H "Accept: application/vnd.github+json" \
  -H "Content-Type: application/json" \
https://api.github.com/repos/<org>/<repo>/pulls \
  --data-binary @- <<'PAYLOAD'
{
  "title": "...",
  "head": "feature/branch-name",
  "base": "main",
  "body": "..."
}
PAYLOAD
```

3. Return the `html_url` from the response to the user.

**Rule:** If `gh` is not installed, fall straight to this method. Never ask the user to raise the PR themselves unless the API call fails.


### Releases

- A release happens every time `dev` is merged into `main`
- Every release is tagged in Git: `v1.2.0`
- Every release has a corresponding entry in `CHANGELOG.md`
- Tag format: `v` + version number. Always. No variations.

### Hotfixes

When something is broken in production and cannot wait for the normal `dev → main` flow:

1. Branch from `main` directly: `fix/hotfix-short-description`
2. Fix only the broken thing — nothing else
3. PR directly into `main` with human review
4. Immediately merge the same fix into `dev` so it is not overwritten
5. Increment the PATCH version: `1.2.0 → 1.2.1`
6. Tag the release and update `CHANGELOG.md`

### Merge Strategy

- `feature/*`, `fix/*`, `docs/*` → `dev`: **squash merge** — keeps `dev` history clean, one commit per feature
- `dev` → `main`: **merge commit** — preserves the full record of what went into each release

---

## 4. Required Documentation

Every FlatPlanet project must always have these files maintained and up to date:

| File | What it contains | Who updates it |
|------|-----------------|----------------|
| README.md | What the project does, how to use it, current status | Claude |
| CHANGELOG.md | Full version history of every change | Claude |
| CONVERSATION-LOG.md | Claude's session-by-session memory — current state, decisions, next steps | Claude (end of every session) |
| CLAUDE-local.md | Local-only Claude brief — generated by HubApi, never committed, git-ignored | HubApi (via `/api/projects/{id}/claude-config/workspace`) |
| docs/api.md | API endpoint documentation | Claude (auto on API changes) |
| .env.example | All required environment variables listed | Developer |

### README.md Must Always Contain

- What this project does (2-3 sentences, plain English)
- Current version and status
- How to get started
- Who to contact with questions
- Link to the CHANGELOG

### CHANGELOG.md Format

Every entry must follow this format:

  ## [VERSION] - YYYY-MM-DD

  ### What changed
  - Plain English description of what is new or different

  ### Why it changed
  - The reason or problem this solves

  ### Who to contact
  - Name or team responsible for this change

Example:

  ## [1.2.0] - 2026-03-19

  ### What changed
  - Added the ability to generate monthly reports directly from the dashboard
  - Fixed a bug where the export button was not working on Safari

  ### Why it changed
  - The team needed to pull reports without asking developers each time
  - Safari users reported the export was broken since the last update

  ### Who to contact
  - Dashboard features: operations@flatplanet.com
  - Bug fixes: dev@flatplanet.com

### API Documentation (docs/api.md)

Every API endpoint must be documented with:
- Endpoint URL and HTTP method
- What it does in one plain English sentence
- Request parameters with types aligned to the live Supabase data dictionary
- Response shape with example
- Error codes it can return

Claude must update docs/api.md automatically whenever any API endpoint is added or changed.

---

## 5. Project Structure

Every FlatPlanet project must follow this structure:

  project-name/
    README.md              - Plain English project overview (required)
    CHANGELOG.md           - Full version history (required)
    CONVERSATION-LOG.md    - Claude's memory across sessions (required)
    CLAUDE-local.md        - DO NOT COMMIT — generated locally by HubApi (must be in .gitignore)
    src/                   - All source code, organized by domain
    tests/                 - Mirrors src/ structure
    docs/
      api.md               - Auto-generated API documentation
      decisions/           - Why key decisions were made
    scripts/               - Dev utilities and automation
    .env.example           - All required env vars (never commit .env)

---

## 6. CLAUDE-local.md — The Claude Brief

There is no `CLAUDE.md` committed to any FlatPlanet project repo. The Claude brief is `CLAUDE-local.md` — a local-only file generated by the FlatPlanet HubApi.

**How it works:**
1. The developer opens a project in the FlatPlanet Hub frontend
2. They click "Get Workspace File" and save `CLAUDE-local.md` to their local project folder
3. HubApi generates the file via `GET /api/projects/{id}/claude-config/workspace`
4. The file contains the live scoped API token + all enforced tech stack standards for that project type
5. `CLAUDE-local.md` must always be in `.gitignore` — it is never committed

**Why local-only?**
The file contains a live API token scoped to the project's database. Committing it would expose that token in the repo. Every developer generates their own copy locally.

**What CLAUDE-local.md contains (auto-generated by HubApi):**
- Project name, type, and whether auth is enabled
- Live scoped API token + expiry
- HubApi base URL and DB proxy endpoints
- Enforced tech stack standards for the project type (frontend / backend / database / fullstack)
- Security Platform auth integration instructions (if `auth_enabled = true`)
- FlatPlanet Standards URL
- Documentation rules

**To regenerate:** Call `POST /api/projects/{id}/claude-config/regenerate` or use the Hub frontend. The old token is revoked and a new file is issued.

---

## 7. Naming Conventions

All variable names must align with the FlatPlanet data dictionary where applicable. The data dictionary lives in Supabase and is queried via the HubApi DB Proxy — not a static file.

| Element | Convention | Example |
|---------|-----------|---------|
| Files | kebab-case | user-profile.ts |
| Folders | kebab-case | auth-service/ |
| Variables | camelCase | userId |
| Constants | UPPER_SNAKE_CASE | MAX_RETRY_COUNT |
| Classes | PascalCase | UserProfile |
| Functions | camelCase verb-first | getUserById |
| Database tables | snake_case | user_profiles |
| Database columns | snake_case | created_at |
| Environment vars | UPPER_SNAKE_CASE | DATABASE_URL |
| React components | PascalCase | UserProfileCard |
| React hooks | use + camelCase | useProjectData |
| Event handlers | handle + PascalCase | handleSubmit |

---

## 8. Code Style

- Indent: 2 spaces (no tabs)
- Max line length: 100 characters
- Strings: single quotes
- Trailing commas in multi-line structures: yes
- Max function length: 40 lines
- Max parameters: 4 (use an options object if more are needed)
- Always define return types explicitly in typed languages
- Comments explain WHY not WHAT
- No commented-out code in commits
- No dead code -- remove unused imports and functions

---

## 9. Testing

- Every new function must have at least one unit test
- Test files live in tests/ mirroring the src/ folder structure
- File naming: filename.test.ts or filename.spec.ts
- All tests must pass before any PR is merged
- Test names must be readable plain English:
  it('returns the user when a valid ID is provided')

---

## 10. Environment and Secrets

- NEVER commit .env files -- add to .gitignore on day one
- Always keep .env.example up to date with every required variable and a description
- No hardcoded credentials anywhere in code, ever
- All secrets go through environment variables only
- When adding a new env var: update .env.example and note it in CHANGELOG.md

---

## 11. Claude Code Session Rules

Every Claude Code session must follow these steps in order:

1. Read the conversation log first -- Open CONVERSATION-LOG.md and read the most recent entry before doing anything else. This is how Claude recovers context from the previous session.
2. Read standards -- Fetch STANDARDS.md before writing any code. Query the live schema via HubApi (`GET /api/projects/{id}/schema/full`) to read the data dictionary before naming tables, columns, or variables
3. Confirm the task -- Restate the task in plain English before starting so anyone can verify Claude understood correctly
4. Follow the Data Dictionary -- All variable names must match the dictionary where applicable
5. Update documentation -- Every session that changes code must update CHANGELOG.md and any affected docs
6. Write tests -- Every new function gets at least one test
7. Commit cleanly -- Each logical change gets its own commit with a clear message
8. Write the conversation log entry -- Update CONVERSATION-LOG.md in the project repo
9. Push the project status report -- Update this project's file in the FLATPLANET-STANDARDS/projects/ folder and push it. This is mandatory. The session is not closed until this is done.

### Conversation Log

Every FlatPlanet project must have a `CONVERSATION-LOG.md` at the repo root. This is Claude's memory across sessions. It is the single most important file for maintaining continuity on a project. Without it, every session starts cold and context is lost.

**Purpose:** The conversation log exists so that when a new Claude session opens, it can read the most recent entry and immediately understand the current state of the project — what has been built, what decisions were made and why, what is broken, what is next, and any context that would otherwise be lost.

**Rules:**
- One entry per session, appended to the top of the file (newest first)
- Never edit or delete previous entries
- Written by Claude at the end of every session, before committing
- Must be committed and pushed with every session close
- The human must not close a session without confirming the log has been written

### CONVERSATION-LOG.md Entry Template

  ---
  ## Session [number] — [YYYY-MM-DD]
  **Version at close:** [X.X.X]
  **Worked on by:** [Claude model or human name if relevant]

  ### What we did this session
  - [Plain English. Not just what changed — include why decisions were made.]
  - [If something was tried and abandoned, record that too and why.]

  ### Current state of the project
  [2-5 sentences. Describe where the project stands right now as if explaining to someone seeing it for the first time. What works, what is in progress, what is missing.]

  ### Decisions made
  - [Decision]: [Why it was made. Include alternatives that were rejected if relevant.]

  ### Open issues / known problems
  - [Anything broken, incomplete, or deferred. Be honest — this is Claude's own notes.]

  ### What to do next session
  1. [First thing to pick up — be specific enough that Claude can start without asking]
  2. [Second thing, etc.]

  ### Context that would otherwise be lost
  - [Anything that is not obvious from reading the code. Gotchas, constraints, things that were hard-won.]
  ---

### Session Summary Template

Claude must also output this to the chat at the end of every session so the human can confirm before closing:

  ## Session Summary

  Project: [name]
  Date: [YYYY-MM-DD]
  New version: [X.X.X]

  What was done:
  - [Plain English bullet points]

  Files changed:
  - [List of files modified]

  Documentation updated:
  - CHANGELOG.md: yes / no
  - README.md: yes / no
  - CONVERSATION-LOG.md: yes (required)
  - docs/api.md: yes / no / not applicable

  Next steps:
  - [Any follow-up tasks or things still to do]

---

## 12. Project Status Reports

Every FlatPlanet project must push a status report to the central FLATPLANET-STANDARDS repository at the end of every session. This gives FlatPlanet-Hub a live view of all active projects in one place — what is being built, who is working on it, and where things stand right now.

### Where the file lives

In the FLATPLANET-STANDARDS repo, under `/projects/`. One file per project, named after the project in kebab-case:

  FLATPLANET-STANDARDS/
    projects/
      tala.md
      fp-data-dictionary.md
      [project-name].md

The file is overwritten on every session close — it always reflects the current state, not a history. History lives in the project's own CONVERSATION-LOG.md.

### How to push the report

At the end of every session, Claude must:
1. Clone or pull the FLATPLANET-STANDARDS repo
2. Write or overwrite `/projects/[project-name].md` with the current status report
3. Commit with message: `chore: update [project-name] status [YYYY-MM-DD]`
4. Push to `main`

This happens after all project commits are done — it is the last action before the session closes.

### Project Status Report Template

  # [Project Name] — Status Report
  **Last updated:** [YYYY-MM-DD]
  **Session:** [number]
  **Version:** [X.X.X]
  **Status:** [Active | Paused | Blocked | Complete]
  **Repo:** [GitHub URL]
  **Owner:** [Role title or team — never personal names]

  ---

  ## What This Project Is
  [2-3 sentences. Plain English. What it does and why it exists.]

  ## Current State
  [What is working right now. What a new person would find if they opened the project today.]

  ## What Was Done This Session
  - [Bullet points — plain English, meaningful level of detail]

  ## What Is Next
  1. [Most important next task — specific enough to act on]
  2. [Second task]
  3. [Third task]

  ## Open Issues
  | ID | Priority | Description |
  |----|----------|-------------|
  | [ID] | High / Medium / Low | [Plain English description] |

  ## Blockers
  [Anything stopping progress. None if clear.]

  ## Key Decisions on Record
  - [Decision]: [Why it was made — brief]

---

## 13. Pull Request Rules

- Every PR must have a plain English description of what changed and why
- Every PR must pass all tests before merging
- Every PR that changes user-facing behavior must include updated documentation
- PRs into main require at least one review
- PR title follows commit format: type: short description

---

## 14. Tech Stack Standards

All projects created on the FlatPlanet platform follow enforced tech stack standards based on project type. These are automatically injected into every project's `CLAUDE-local.md` file.

### Project Types

| Type | Stack | Deploy Target |
|---|---|---|
| `frontend` | React.js + TypeScript (latest) | Netlify |
| `backend` | .NET 10 / C# | Azure App Service |
| `database` | Supabase / PostgreSQL | — |
| `fullstack` | Frontend + Backend + Database | Netlify + Azure |

### Frontend Standards (React / TypeScript)
- React.js with TypeScript (latest version)
- Strict TypeScript — no `any`, explicit return types on all functions
- Component naming: PascalCase, one component per file
- Hooks: prefix with `use`, keep side effects in useEffect only
- State management: follow existing pattern in the codebase
- Folder structure: feature-based (components, hooks, services, types per feature)
- API calls: always through a service layer — never fetch directly in components
- Error boundaries: wrap major sections
- No unused imports, no `console.log` in production code

### Backend Standards (.NET 10 / C#)
- .NET 10 / C# — use latest language features (primary constructors, pattern matching, etc.)
- Clean Architecture: Controller → Application Service → Domain → Infrastructure
- SOLID principles enforced at all times
- Dependency Injection for all services — never instantiate dependencies manually
- Apply design patterns where appropriate: Strategy, Chain of Responsibility, Factory, Decorator
- No EF Core — Dapper only, raw SQL via `IDbConnectionFactory`
- All `async`/`await` — no blocking calls (`.Result`, `.Wait()`)
- Platform API JSON responses use **camelCase** field names (standard .NET serialization). When deserializing with `System.Text.Json`, always configure `JsonSerializerOptions { PropertyNamingPolicy = JsonNamingPolicy.CamelCase }`. Never assume snake_case — that was the source of a production bug where all deserialized fields silently returned null.
- Always run `dotnet build` before committing

### Database Standards (Supabase / PostgreSQL)
- ALWAYS check the data dictionary before naming any table, column, variable, or function
- ALWAYS read the schema before writing any database-related code
- All DDL goes through migration endpoints — never raw DDL in query endpoints
- All queries use `@paramName` — never concatenate values into SQL strings
- `snake_case` for all table and column names
- UUID primary keys with `gen_random_uuid()`
- Always include `created_at TIMESTAMPTZ DEFAULT now()`
- Soft deletes preferred — use `is_active` boolean over hard deletes
- Always add indexes on foreign keys and frequently queried columns
- SQL `@param` names must be snake_case matching the column name exactly — camelCase params (e.g. `@businessId`) silently fail to bind in Dapper, leaving those columns NULL. Single-word params like `@name` appear to work by coincidence, making the bug hard to spot
- UUID parameters require an explicit `::uuid` cast in SQL — e.g. `WHERE id = @id::uuid`. Without it, binding silently fails against UUID columns
- JSONB columns return as raw strings through the Platform API (Dapper behaviour) — any DTO property mapping to a JSONB column must use a converter to unwrap the string before treating it as an object
- Array parameters sent to the Platform API query endpoint must be JSON arrays in the `parameters` object — e.g. `{ "ids": ["a", "b"] }`. The Platform API unwraps JSON arrays automatically; do NOT serialize them to a string first

### Authentication
When a project has authentication enabled, it must use the FlatPlanet Security Platform (SP) — projects must NOT build their own auth system.

SP Base URL: `https://flatplanet-security-api-d5cgdyhmgxcebyak.southeastasia-01.azurewebsites.net`
JWT issuer: `flatplanet-security` | JWT audience: `flatplanet-apps`

All inter-service (backend-to-backend) calls must use SP service tokens.

**JWT Business Membership Claims**

The SP JWT contains two parallel claims for business membership:

| Claim | Type | Example | Use for |
|---|---|---|---|
| `business_codes` | `string[]` | `["fp"]` | Display, filtering, feature flags |
| `business_ids` | `string[]` | `["3fa85f64-5717-4562-b3fc-2c963f66afa6"]` | Database foreign keys |

Both arrays are ordered identically — index `n` in `business_codes` matches index `n` in `business_ids`. Never hardcode either value; always read from the decoded JWT.

### File Storage

All projects that need to store files **must use the FlatPlanet Platform API storage endpoints** — teams must NOT build their own upload infrastructure (no direct Azure SDK calls, no Supabase Storage, no S3).

**Why:** The Platform API enforces per-project bucket isolation (one Supabase Storage bucket per project), safe soft-delete ordering, signed URL generation, and per-user ownership checks. Building your own upload bypasses all of these controls.

| Endpoint | Method | Description |
|---|---|---|
| `/api/v1/storage/upload` | `POST` | Upload a file (multipart, 50 MB max) |
| `/api/v1/storage/files` | `GET` | List files (filter by category, tags) |
| `/api/v1/storage/files/{fileId}/url` | `GET` | Get a fresh SAS URL (60-min lifetime) |
| `/api/v1/storage/files/{fileId}` | `DELETE` | Soft-delete a file |

Files are **automatically scoped to your app** via the `app_id` claim in your project API token — you do not need to pass any tenant or project identifier. All members of a project team share access to the same app-scoped files.

Use `businessCode` (e.g. `"fp"`) and `category` (e.g. `"documents"`, `"images"`) to organise uploads. Do not invent your own storage solution.

---

## 15. Standards Maintenance

This document is versioned. Any FlatPlanet team member can propose a change.

To propose a change:
1. Open a pull request on the FLATPLANET-STANDARDS repository
2. In the PR description: what you are changing, why, and who it affects
3. Tag the relevant team lead for review
4. Once approved, update the version number and last updated date at the top of this file

Review schedule:
- Monthly: minor review
- Quarterly: full review by all team leads
- As needed: any team member can raise an update at any time

---

## 16. File Versioning — STANDARDS.md, CLAUDE.md, and CLAUDE-local.md

Three files govern every FlatPlanet project session. Each is versioned so Claude can detect when it is working with outdated context and take action.

### The Three Files

| File | Version location | Who updates it | Update trigger |
|---|---|---|---|
| `STANDARDS.md` | Header: `> Version: X.X` | FlatPlanet team | When platform-wide rules change |
| `CLAUDE.md` | Header: `<!-- CLAUDE_MD_VERSION: X.X -->` | HubApi (auto-pushed on project update) | When the project template changes |
| `CLAUDE-local.md` | Header: `<!-- CLAUDE_LOCAL_VERSION: X.X -->` | HubApi (generated on demand) | When the local template changes |

### Current Versions

| File | Current Version |
|---|---|
| `STANDARDS.md` | 3.0 |
| `CLAUDE.md` | 1.1 |
| `CLAUDE-local.md` | 1.5 |

### What Claude Must Do — Version Checks Are Mandatory

**Checking STANDARDS.md and CLAUDE-local.md versions is not optional. Claude must check both at every session start, AND regularly throughout long sessions (e.g., whenever switching tasks or after a significant break in the conversation). If either is outdated, Claude must notify the user before doing any other work.**

**1. Check STANDARDS.md version (every session start + regularly)**

Fetch the raw file and read the version from the header:
```
https://raw.githubusercontent.com/FlatPlanet-Hub/FLATPLANET-STANDARDS/main/FLATPLANET-STANDARDS/STANDARDS.md
```
Compare against the current version in the table above. If the version has changed (even mid-session), tell the user **immediately**:
```
⚠️ FlatPlanet Standards have been updated (now vX.X, you had vX.X).
Read the changelog at the bottom of STANDARDS.md before we proceed.
```
Read the full updated file before writing any code.

**2. Check CLAUDE.md version**

At the start of every session, check if CLAUDE.md has been updated on the remote:
```bash
git fetch origin
git diff HEAD origin/main -- CLAUDE.md
```
If there are changes, pull them before starting work:
```bash
git pull origin main
```
CLAUDE.md is committed to the repo. It is the shared project brief for all team members. Always work from the latest version.

**3. Check CLAUDE-local.md version (every session start + regularly)**

Read the `<!-- CLAUDE_LOCAL_VERSION: X.X -->` comment at the top of CLAUDE-local.md.
Compare it against the current version in the table above.

If your local version is older, tell the user **immediately**:
```
⚠️ Your CLAUDE-local.md is outdated (you have vX.X, current is vX.X).
Please regenerate it from the FlatPlanet Hub to get the latest template and a fresh token.

To regenerate:
POST {hubApiBaseUrl}/api/projects/{projectId}/claude-config/regenerate
Header: Authorization: Bearer <your SP JWT>

Or use the FlatPlanet Hub frontend → your project → "Get Workspace File".
```

Do not proceed with the outdated file if the version gap is more than one minor version.

> **Why regular checks matter:** STANDARDS.md and CLAUDE-local.md can be updated at any point during the day — not just between sessions. A team lead may push a critical rule change mid-afternoon while your session is still running. Claude must re-check whenever it starts a new task block, not only at session start. If a version bump is detected mid-session, stop, notify the user, and pull the latest content before continuing.

### How Versions Are Incremented

- **STANDARDS.md**: Incremented manually by the FlatPlanet team when rules change. Minor version for additions, major version for breaking changes.
- **CLAUDE.md / CLAUDE-local.md**: Incremented in `ClaudeConfigService.cs` (`LocalFileVersion` constant). Increment when the template adds, removes, or changes instructions in a meaningful way. A version bump triggers a re-push of CLAUDE.md to all project repos on next project update.

---

## STANDARDS.md Version Changelog

| Version | Date | What changed |
|---|---|---|
| 3.2 | 2026-04-15 | Gap analysis fixes — CLAUDE-local.md template bumped to v1.7: Delete returns 200 not 204, businessCode marked required on List, response envelope shape documented, JWT normalization pattern added (single-membership = plain string not array). |
| 3.1 | 2026-04-15 | Added rule: Platform API responses are camelCase — always use `JsonNamingPolicy.CamelCase` when deserializing. Documents a real production bug where snake_case assumption caused all fields to silently return null. CLAUDE-local.md template bumped to v1.6. |
| 3.0 | 2026-04-15 | Added JWT business_ids claim documentation alongside business_codes. Added array parameter rule for Platform API query endpoint. Updated File Storage "Why" to reflect Supabase Storage per-project bucket isolation (replacing Azure Managed Identity SAS description). CLAUDE-local.md template bumped to v1.5 with business_ids guidance. |
| 2.9 | 2026-04-15 | Added File Storage rule — all projects must use Platform API storage, not build their own. Strengthened version check mandate: Claude must check STANDARDS.md and CLAUDE-local.md regularly (not just at session start). Explicit mid-session notification rule added. Updated CLAUDE-local.md current version to 1.3. |
| 2.8 | 2026-04-14 | Added Section 16 — full file versioning specification for STANDARDS.md, CLAUDE.md, and CLAUDE-local.md. Defined how versions are incremented and what Claude must do at session start. |
| 2.6 | 2026-04-06 | Added Dapper snake_case + ::uuid cast rules, retired DATA_DICTIONARY.md, updated DB proxy instructions. |

---

Last updated: 2026-04-15
Version: 3.0
Maintained by: FlatPlanet-Hub
One standard, every project, every person.
