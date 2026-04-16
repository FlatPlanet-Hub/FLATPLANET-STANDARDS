# ISO Audit Readiness — Status Report
**Last updated:** 2026-04-16
**Session:** 4
**Version:** 1.0.2
**Status:** Active
**Repo:** https://github.com/FlatPlanet-Hub/ISO-Audit-Readiness
**Owner:** Internal Audit

---

## What This Project Is
An internal web application for Round Earth Philippines (trading as Flat Planet) that tracks, manages, and measures organisational readiness for ISO/IEC 27001:2022 certification. It gives every department a structured task list mapped to specific Annex A controls, provides real-time readiness percentages and gap analysis to the Internal Auditor and CEO, and predicts whether the organisation will meet its target audit date at current velocity.

## Current State
Sessions 1–4 complete. CI is **green** on main (`bb153d5`). Full gap analysis performed — 20+ issues resolved across frontend services, backend controllers, and application logic. Azure App Service is provisioned. Standard environment variables (`PlatformApi__Token`, `PlatformApi__BaseUrl`, `ConnectionStrings__Default`) are configured automatically by the platform. CLAUDE-local.md updated to v1.7 with fresh token (expires 2026-05-16).

## What Was Done This Session
- Full gap analysis of the entire codebase (frontend + backend)
- **Critical fix**: `apiFetch` now unwraps `{ success, data }` envelope — every component was previously rendering `undefined` for all data
- **Frontend service fixes**: All HTTP method mismatches corrected (PATCH → PUT for tasks, settings, documents, departments, users; POST → PUT for notifications)
- **Settings service**: Added dictionary↔typed-object transformation layer; audit cycle path corrected
- **Control service**: Fixed confirm/reject routes and method; added coverage shape transformation (flat backend → nested frontend type)
- **Readiness service**: Added full shape transformation (backend field names → frontend `ReadinessDashboard` type)
- **AI service**: Fixed from GET to POST `/api/ai/suggest-mapping` with request body
- **PendingMappingQueue**: Updated to new `confirmMapping(taskId, controlId)` signature
- **Backend additions**: Department CRUD (POST, PUT, POST /deactivate), user deactivate, document version history (GET), task velocity (GET), subtask creation (POST)
- **ControlService**: `ConfirmMappingAsync` re-reads DB after UPDATE — eliminates premature `mapping_confirmed` flip
- **DocumentService**: Conditional reverification — `TriggeredReverification` flag now correctly gates mass-update and notifications
- **AuditCycleService**: Clears `needs_reverification` on cycle reset
- **NotificationDto + entity**: Added `CreatedAt` field
- **DocumentVersionDto**: Added optional `CreatedAt` field
- **CORS**: Removed `AllowCredentials()` (incompatible with wildcard origins)
- **DI**: Fixed `AnthropicService` double-registration
- **Minor fixes**: `DocumentManager` CSS template literal, `DocumentsView` loading state, `DepartmentView` error display, `VelocityChart` type reference
- CLAUDE-local.md updated to v1.7; copied into repo root (gitignored)

## What Is Next
1. **Add `AZURE_WEBAPP_PUBLISH_PROFILE` GitHub secret** — download from Azure Portal → `iso-audit-readiness-api` → Get publish profile; add to repo secrets
2. **Set `Anthropic__ApiKey`** in Azure App Service environment variables
3. **Set `FRONTEND_URL`** in Azure App Service environment variables (Netlify URL, for CORS)
4. **Connect frontend to Netlify** (base: `frontend`, build: `npm run build`, publish: `dist`, env: `VITE_API_URL=https://iso-audit-readiness-api.azurewebsites.net`)
5. **Commit `frontend/package-lock.json`** — run `npm install` in `frontend/` locally then commit
6. **Verify live app** at `https://iso-audit-readiness-api.azurewebsites.net/swagger` once deploy step completes

## Open Issues
| ID | Priority | Description |
|----|----------|-------------|
| 001 | High | `AZURE_WEBAPP_PUBLISH_PROFILE` GitHub secret not yet added — deploy step skips without it |
| 002 | High | `Anthropic__ApiKey` not yet set in Azure App Service — AI suggest-mapping will fail |
| 003 | High | `FRONTEND_URL` not yet set in Azure App Service — CORS will block Netlify requests |
| 004 | High | Netlify site not yet created — frontend has no deployment target |
| 005 | Medium | `frontend/package-lock.json` not committed — CI uses `npm install` (non-deterministic) |
| 006 | Medium | No automated tests written (unit or integration) |
| 007 | Low | `StorageService.businessCode` hardcoded to `"iso"` — should be sourced from JWT when auth enabled |
| 008 | Low | `tasks.updated_at` column not confirmed — velocity endpoint degrades gracefully if absent |

## Blockers
Items 001–004 require manual configuration in Azure Portal and Netlify — cannot be done programmatically.

## Key Decisions on Record
- **`annex_controls` not `annex_a_controls`**: Platform API rejected `annex_a_controls`.
- **`jsonb` for `upload_roles_allowed`**: Platform API does not support `text[]`.
- **Auth disabled**: FlatPlanet SP auth not yet enabled; simple user switcher in UI.
- **All DB access via Platform API proxy**: Backend never connects to Supabase directly.
- **Swashbuckle 6.9.0 + Microsoft.OpenApi 1.6.22**: `Microsoft.AspNetCore.OpenApi` 10.x removed (conflicts with Swashbuckle 1.6.x type surface).
- **Infrastructure FrameworkReference**: `Microsoft.NET.Sdk` class library needs explicit `<FrameworkReference Include="Microsoft.AspNetCore.App" />` for DI/Logging/Http types.
- **CI `continue-on-error` on deploy**: Keeps CI green before publish profile secret is configured.
- **`apiFetch` unwraps envelope**: All service functions receive unwrapped `T` from `{ success, data: T }` — do not double-unwrap.
- **Settings as dictionary**: Backend stores/returns settings as `Dictionary<string,string>` with snake_case keys; `settingsService.ts` transforms to/from typed `SystemSettings`.
- **Control confirm/reject signature**: `confirmMapping(taskId, controlId)` — uses task UUID + control string (e.g. "5.1"), NOT the task_control row UUID.
- **Standard Azure env vars auto-provisioned**: `PlatformApi__Token`, `PlatformApi__BaseUrl`, `ConnectionStrings__Default` set automatically at provision time.
