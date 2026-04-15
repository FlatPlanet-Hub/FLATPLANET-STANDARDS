# ISO Audit Readiness — Status Report
**Last updated:** 2026-04-16
**Session:** 2
**Version:** 1.0.1
**Status:** Active
**Repo:** https://github.com/FlatPlanet-Hub/ISO-Audit-Readiness
**Owner:** Internal Audit

---

## What This Project Is
An internal web application for Round Earth Philippines (trading as Flat Planet) that tracks, manages, and measures organisational readiness for ISO/IEC 27001:2022 certification. It gives every department a structured task list mapped to specific Annex A controls, provides real-time readiness percentages and gap analysis to the Internal Auditor and CEO, and predicts whether the organisation will meet its target audit date at current velocity.

## Current State
Sessions 1–2 complete. Full system is built (139 files, .NET 10 API + React frontend), all code is on `dev`. Azure App Service is provisioned at `https://iso-audit-readiness-api.azurewebsites.net`. GitHub Actions workflow is in place to auto-deploy on push to main. PR #2 (dev → main) is open and ready to merge. The app is not yet live — Azure App Service application settings and the GitHub `AZURE_WEBAPP_PUBLISH_PROFILE` secret still need to be configured before the first deployment completes. Netlify frontend not yet connected.

## What Was Done This Session
- Pushed project status report to FLATPLANET-STANDARDS (session 1 close)
- Merged PR #1 (feature/initial-build → dev)
- Opened PR #2 (dev → main) at https://github.com/FlatPlanet-Hub/ISO-Audit-Readiness/pull/2
- Provisioned Azure App Service: `iso-audit-readiness-api` at `https://iso-audit-readiness-api.azurewebsites.net`
- Created GitHub Actions deployment workflow (`.github/workflows/deploy-backend.yml`) — auto-deploys backend to Azure on push to main
- Fixed `frontend/netlify.toml` — API proxy target updated to the live Azure URL
- Updated README.md with full deployment instructions (Azure App Service settings, GitHub secret, Netlify setup steps)
- Committed v1.0.1, pushed to dev

## What Is Next
1. Add `AZURE_WEBAPP_PUBLISH_PROFILE` secret to GitHub (download from Azure Portal → iso-audit-readiness-api → Get publish profile)
2. Configure Azure App Service Application Settings: `PlatformApi__Token`, `Anthropic__ApiKey`, `FRONTEND_URL` (see README.md for full list)
3. Merge PR #2 (dev → main) — triggers first automated Azure deployment
4. Connect frontend to Netlify (base dir: `frontend`, build: `npm run build`, publish: `dist`)
5. Verify live app at `https://iso-audit-readiness-api.azurewebsites.net/swagger`
6. Add `tasks.needs_reverification` column (deferred open issue from session 1)

## Open Issues
| ID | Priority | Description |
|----|----------|-------------|
| 001 | High | `AZURE_WEBAPP_PUBLISH_PROFILE` GitHub secret not yet added — first deployment will fail without it |
| 002 | High | Azure App Service Application Settings not yet configured — API will start but all DB calls will fail |
| 003 | High | Netlify site not yet created — frontend has no deployment target |
| 004 | Medium | PR #2 (dev → main) awaiting merge |
| 005 | Medium | `tasks.needs_reverification` column not yet added (spec section 4.5) |
| 006 | Medium | No automated tests written (unit or integration) |
| 007 | Low | dotnet build and npm run dev unverified locally (.NET SDK / Node.js not installed on dev machine) |

## Blockers
Azure App Service settings and GitHub publish profile secret must be configured manually by the user — cannot be set programmatically without Azure Portal access.

## Key Decisions on Record
- **`annex_controls` not `annex_a_controls`**: Platform API rejected `annex_a_controls` as a table name. `annex_controls` used everywhere.
- **`jsonb` for `upload_roles_allowed`**: Platform API does not support `text[]` column type; stored as JSON array instead.
- **Auth disabled**: FlatPlanet Security Platform auth is not yet enabled. A simple user switcher handles role context in the UI.
- **All DB access via Platform API proxy**: Backend never connects to Supabase directly.
- **Pre-assigned UUIDs**: Departments use `a1000000-...-000X` pattern, tasks use `b1000000-...-000X` for deterministic seeding.
- **camelCase JSON**: Platform API returns camelCase — `JsonNamingPolicy.CamelCase` used throughout.
- **GitHub Actions on main, backend/** path**: Backend deployments trigger only when backend files change, keeping CI fast.
