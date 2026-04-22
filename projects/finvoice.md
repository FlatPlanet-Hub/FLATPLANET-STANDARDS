# Finvoice — Status Report
**Last updated:** 2026-04-22
**Session:** 4
**Version:** 1.3.3
**Status:** Active
**Repo:** https://github.com/FlatPlanet-Hub/Finvoice
**Owner:** Finance / Operations

---

## What This Project Is

Finvoice is an AU-facing invoice management system for Flat Planet Philippines. It handles staff billing invoices across multiple pricing models (new/old/offsite), salary increases, redundancy pay, overtime, night differential, and allowances. It integrates with Xero for invoice push/pull and includes a full suite of payroll calculation tools.

## Current State

The app is live on Netlify and fully functional for single-device use. All application data (invoices, clients, users, exchange rates) is stored in the browser's localStorage via Zustand persist middleware. A backend was designed (Azure Functions + Supabase, with the API client already written in `src/lib/apiClient.js`) but Azure has not been provisioned yet. Auth uses a local PBKDF2 password system with cross-device revocation via Netlify Blobs.

The project now has 67 unit tests (Vitest), a proper branch workflow (`dev` branch created, feature branches → dev → main), three Architecture Decision Records in `docs/decisions/`, and an accurate CLAUDE.md.

## What Was Done This Session

- Installed Vitest 4.1.5; added `npm test` / `npm run test:watch` / `npm run test:coverage` scripts
- Wrote 67 unit tests across `tests/authUtils.test.js` and `tests/calculations.test.js` — all passing
- Fixed package.json version from `0.0.0` to `1.3.3` (was never updated from Vite scaffold default)
- Created `dev` branch on remote; established feature/fix/* → dev → main branch strategy
- Created `docs/decisions/` with three ADRs documenting: localStorage interim storage, PBKDF2 local auth, Netlify Blobs revocation
- Corrected `CLAUDE.md`: accurate roles (`admin|user`), correct supabase.js/apiClient.js descriptions, branch workflow, test instructions
- Updated `.env.example` with Supabase env var placeholders
- Fixed `supabase.js`: replaced broken import of uninstalled `@supabase/supabase-js` with documented stub
- Removed `console.warn` from `deleteUser` last-admin guard (silent protection, no console output)
- Updated CHANGELOG.md and CONVERSATION-LOG.md per STANDARDS.md session-close requirements

## What Is Next

1. **Merge PR**: `fix/standards-compliance → dev` on GitHub, then `dev → main`
2. **Confirm Rica can log in** — if blocked, visit `/api/rescue-allowlist?confirm=yes`
3. **Remove unwanted user accounts** — Rica to do via Settings → User Management
4. **Backend migration**: Create Supabase tables, connect existing `apiClient.js` so data syncs across all devices

## Open Issues

| ID | Priority | Description |
|----|----------|-------------|
| FIN-001 | High | Data isolation — users on different devices cannot see each other's data. Requires backend database migration. |
| FIN-002 | Low | PR `fix/standards-compliance → dev` open but not yet merged. Needs review and merge, then `dev → main`. |
| FIN-004 | Low | FlatPlanet Security Platform auth is disabled. App uses its own PBKDF2 auth. Migration pending decision. |

**Resolved this session:**
- FIN-003 ✅ — Unit tests added (67 tests, all passing)
- FIN-005 ✅ — CLAUDE.md corrected (roles, supabase.js, apiClient.js, branch strategy)

## Blockers

FIN-001 (data isolation) blocks multi-user collaboration. All other features work correctly on a single device.

## Key Decisions on Record

- **localStorage as interim storage:** Chosen because the Azure backend was not provisioned. Not a permanent architecture — migration planned. See `docs/decisions/001-localstorage-interim-storage.md`.
- **Local PBKDF2 password auth:** Built because FlatPlanet Security Platform auth was disabled. To be revisited when SP auth is enabled. See `docs/decisions/002-pbkdf2-local-auth.md`.
- **Netlify Blobs for cross-device revocation:** Five-layer revocation system. Fail-open design. See `docs/decisions/003-netlify-blobs-revocation.md`.
- **role defaults to 'user':** AuthContext uses 'user' as the safe fallback when a user record cannot be resolved.
- **supabase.js as documented stub:** Package not installed, not imported anywhere. File kept as migration reference.
