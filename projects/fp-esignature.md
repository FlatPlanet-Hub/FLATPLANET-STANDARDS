# FP-ESignature — Status Report
**Last updated:** 2026-04-15
**Session:** 3
**Version:** 1.2.0
**Status:** Active
**Repo:** https://github.com/FlatPlanet-Hub/FP-ESignature
**Owner:** FlatPlanet dev team

---

## What This Project Is
FP-ESignature is an online document signing platform. Users upload documents, place signature fields, invite signers by email, and track the signing workflow. Fullstack: React + TypeScript frontend, .NET 10 C# backend, Supabase PostgreSQL.

## Current State
Frontend live on Netlify. Backend on Azure App Service (fp-esignature-api). CI deployment fixing a 503 startup crash (removed unused JwtBearer preview-3, upgraded Swashbuckle to 10.1.7) is in progress. Upload and auth working. PDF preview endpoint added this session.

## What Was Done This Session
- Diagnosed Azure 503 — unused JwtBearer preview-3 DLL conflicting with .NET 10 GA runtime at startup; removed it
- Upgraded Swashbuckle 7.2.0 → 10.1.7 (net10.0 native build)
- Added CORS to fallback startup-error app for better crash diagnostics
- Added /preview endpoint to stream documents inline for PDF viewer
- Increased Axios upload timeout 30s → 120s, global timeout 30s → 60s
- Ran compliance audit against STANDARDS.md v3.3 — identified multiple gaps
- Created CHANGELOG.md, updated README.md, updated CONVERSATION-LOG.md

## What Is Next
1. Regenerate CLAUDE-local.md to v1.7 before next session
2. Verify Azure backend is back up after CI deployment
3. Test full end-to-end: login → upload → preview → signers → fields → send → sign
4. Set up dev branch — stop committing directly to main
5. Create docs/api.md

## Open Issues
| ID | Priority | Description |
|----|----------|-------------|
| 1 | High | CLAUDE-local.md is v1.4, needs regeneration to v1.7 |
| 2 | High | Azure backend 503 — fix deployed, awaiting CI |
| 3 | Medium | All commits going directly to main — branch strategy not followed |
| 4 | Medium | No docs/api.md |
| 5 | Medium | No tests written |

## Blockers
None. Azure fix deploying via CI.

## Key Decisions on Record
- Self-contained win-x64 publish: Azure has no .NET 10; runtime bundled in publish output
- Local JWT validation: SP tokens validated locally using Jwt__SecretKey — no outbound SP call per request
- Platform API for file storage: all uploads via FlatPlanet Platform API, no direct Azure Blob SDK
- Netlify proxy for SP auth: /sp-proxy/* in netlify.toml proxies SP auth calls to avoid CORS
