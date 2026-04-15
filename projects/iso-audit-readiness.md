# ISO Audit Readiness — Status Report
**Last updated:** 2026-04-16
**Session:** 3
**Version:** 1.0.1
**Status:** Active
**Repo:** https://github.com/FlatPlanet-Hub/ISO-Audit-Readiness
**Owner:** Internal Audit

---

## What This Project Is
An internal web application for Round Earth Philippines (trading as Flat Planet) that tracks, manages, and measures organisational readiness for ISO/IEC 27001:2022 certification. It gives every department a structured task list mapped to specific Annex A controls, provides real-time readiness percentages and gap analysis to the Internal Auditor and CEO, and predicts whether the organisation will meet its target audit date at current velocity.

## Current State
Sessions 1–3 complete. CI is **green** — both frontend (React/Vite/TypeScript) and backend (.NET 10) build successfully in GitHub Actions. Azure App Service is provisioned at `https://iso-audit-readiness-api.azurewebsites.net`. The deploy step in CI runs with `continue-on-error: true` and will complete automatically once the `AZURE_WEBAPP_PUBLISH_PROFILE` GitHub secret and Azure App Service Application Settings are configured. Netlify frontend is not yet connected.

## What Was Done This Session
- Merged PR #2 (dev → main) — full codebase now on main
- Fixed Platform API auto-generated `ci.yml`: added `working-directory` defaults for both frontend and backend jobs (was running from repo root)
- Removed redundant `deploy-backend.yml`
- Fixed four CI build failures found through Actions logs:
  - Frontend: added `vite-env.d.ts` to expose `import.meta.env` types (TS2339)
  - Backend: fixed raw string literal in `AnthropicService` — changed `$"""` to `$$"""` (CS9006)
  - Backend: added `FrameworkReference` to Infrastructure project for `Microsoft.AspNetCore.App` (CS0234 on Microsoft.Extensions.*)
  - Backend: removed `Microsoft.AspNetCore.OpenApi` 10.x (conflicts with Swashbuckle); pinned `Microsoft.OpenApi 1.6.22`, Swashbuckle 6.9.0 (CS0234 on Microsoft.OpenApi.Models)
- CI is now green on commit `29203a7`; frontend ✅ backend ✅
- Synced `dev` to match `main`

## What Is Next
1. **Add `AZURE_WEBAPP_PUBLISH_PROFILE` GitHub secret** — download from Azure Portal → `iso-audit-readiness-api` → Get publish profile
2. **Configure Azure App Service Application Settings** (PlatformApi__Token, Anthropic__ApiKey, FRONTEND_URL, ASPNETCORE_ENVIRONMENT=Production)
3. **Re-run CI** — deploy step will complete and API goes live at `https://iso-audit-readiness-api.azurewebsites.net`
4. **Connect frontend to Netlify** (base: `frontend`, build: `npm run build`, publish: `dist`)
5. **Commit `frontend/package-lock.json`** after running `npm install` locally so CI can use `npm ci`
6. **Add `tasks.needs_reverification` column** (deferred from session 1)

## Open Issues
| ID | Priority | Description |
|----|----------|-------------|
| 001 | High | `AZURE_WEBAPP_PUBLISH_PROFILE` GitHub secret not yet added — deploy step skips without it |
| 002 | High | Azure App Service Application Settings not yet configured — API starts but DB calls fail |
| 003 | High | Netlify site not yet created — frontend has no deployment target |
| 004 | Medium | `frontend/package-lock.json` not committed — CI uses `npm install` (non-deterministic) |
| 005 | Medium | `tasks.needs_reverification` column not yet added (spec section 4.5) |
| 006 | Medium | No automated tests written (unit or integration) |

## Blockers
Azure App Service settings and GitHub publish profile secret require manual configuration in Azure Portal — cannot be set programmatically.

## Key Decisions on Record
- **`annex_controls` not `annex_a_controls`**: Platform API rejected `annex_a_controls`.
- **`jsonb` for `upload_roles_allowed`**: Platform API does not support `text[]`.
- **Auth disabled**: FlatPlanet SP auth not yet enabled; simple user switcher in UI.
- **All DB access via Platform API proxy**: Backend never connects to Supabase directly.
- **Swashbuckle 6.9.0 + Microsoft.OpenApi 1.6.22**: `Microsoft.AspNetCore.OpenApi` 10.x removed (conflicts with Swashbuckle 1.6.x type surface).
- **Infrastructure FrameworkReference**: `Microsoft.NET.Sdk` class library needs explicit `<FrameworkReference Include="Microsoft.AspNetCore.App" />` for DI/Logging/Http types.
- **CI `continue-on-error` on deploy**: Keeps CI green before publish profile secret is configured.
