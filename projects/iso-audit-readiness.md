# ISO Audit Readiness — Status Report
**Last updated:** 2026-04-15
**Session:** 1
**Version:** 1.0.0
**Status:** Active
**Repo:** https://github.com/FlatPlanet-Hub/ISO-Audit-Readiness
**Owner:** Internal Audit

---

## What This Project Is
An internal web application for Round Earth Philippines (trading as Flat Planet) that tracks, manages, and measures organisational readiness for ISO/IEC 27001:2022 certification. It gives every department a structured task list mapped to specific Annex A controls, provides real-time readiness percentages and gap analysis to the Internal Auditor and CEO, and predicts whether the organisation will meet its target audit date at current velocity.

## Current State
Session 1 complete. Full system is built and pushed to GitHub on branch `feature/initial-build`. PR #1 to `dev` is open and awaiting human review. Database is fully built and seeded (11 tables, 93 Annex A controls, 8 departments, 59 tasks, 211 control mappings, 272 subtasks, 38 collaborator links, 7 system settings, 5 documents). The .NET 10 Web API (20 endpoints, Clean Architecture) and React 18 + TypeScript frontend (5 main views) are written but have not been executed locally — Node.js and .NET SDK are not installed on the dev machine. Azure App Service is not yet provisioned.

## What Was Done This Session
- Cloned repo, read full spec (ISO27001_Readiness_System_Spec.docx), seed files, and STANDARDS.md
- Created 11 database tables via FlatPlanet Platform API (annex_controls, departments, users, tasks, task_controls, task_collaborators, subtasks, documents, document_versions, notifications, system_settings)
- Seeded all data — 93 Annex A controls, 8 departments, 59 tasks, 211 task_controls, 38 collaborators, 272 subtasks, 7 settings, 5 documents (all verified by row count)
- Built .NET 10 Web API (70 files, Clean Architecture): Domain, Application, Infrastructure, API layers — 20 REST endpoints covering tasks, departments, controls, readiness, documents, notifications, settings, users, AI mapping, and audit cycle management
- Built React 18 + TypeScript + Vite frontend (63 files): Executive View, Auditor View, Department View, Settings Panel, Documents View — role-gated editing, Recharts velocity chart, SVG readiness gauge
- Implemented readiness engine: only confirmed tasks count; PC7 acknowledgment task handled proportionally; velocity-based timeline projection with 4 signals
- Integrated Anthropic claude-sonnet-4-6 for AI control mapping suggestions (non-blocking)
- Wrote README.md, CHANGELOG.md, docs/api.md, updated .gitignore
- Committed 139 files (7,757 insertions) to `feature/initial-build` and raised PR #1 to `dev`

## What Is Next
1. Review and merge PR #1 (feature/initial-build → dev) then raise PR dev → main
2. Install .NET 10 SDK and run `dotnet build` to verify backend compiles
3. Install Node.js, run `npm install` and `npm run dev` to verify frontend renders
4. Provision Azure App Service (`POST /api/projects/{id}/provision-azure`)
5. Deploy frontend to Netlify (connect GitHub repo, set VITE_API_URL env var)
6. Enable FlatPlanet Security Platform auth when ready (update CLAUDE-local.md, re-run codegen)

## Open Issues
| ID | Priority | Description |
|----|----------|-------------|
| 001 | High | .NET 10 SDK not installed on dev machine — backend compilation unverified |
| 002 | High | Node.js not installed on dev machine — frontend untested locally |
| 003 | High | Azure App Service not provisioned — backend has no deployment target yet |
| 004 | Medium | `tasks.needs_reverification` column not yet added — spec section 4.5 describes flagging tasks after document upload |
| 005 | Medium | No automated tests written (unit or integration) |
| 006 | Low | ISMS department has 7 collaborating departments — notification volume will be high; no batching in place |

## Blockers
None blocking code delivery. .NET SDK and Node.js required to run locally; Azure provisioning required before backend can go live.

## Key Decisions on Record
- **`annex_controls` not `annex_a_controls`**: Platform API rejected `annex_a_controls` as a table name. `annex_controls` used everywhere.
- **`jsonb` for `upload_roles_allowed`**: Platform API does not support `text[]` column type; stored as JSON array instead.
- **Auth disabled**: FlatPlanet Security Platform auth is not yet enabled. A simple user switcher handles role context in the UI. Enable via Hub when ready.
- **All DB access via Platform API proxy**: Backend never connects to Supabase directly.
- **Pre-assigned UUIDs**: Departments use `a1000000-...-000X` pattern, tasks use `b1000000-...-000X` for deterministic seeding and cross-reference.
- **camelCase JSON**: Platform API returns camelCase — `JsonNamingPolicy.CamelCase` used throughout; SnakeCaseLower explicitly excluded.
- **Readiness % formula**: `(confirmed tasks with status 'done') / (total confirmed tasks) × 100`. PC7 contributes proportionally via `acknowledgment_current / acknowledgment_target`.
- **Non-blocking AI mapping**: Tasks work without confirmed control mappings. AI suggestions queue for auditor review and do not block task creation or status updates.
