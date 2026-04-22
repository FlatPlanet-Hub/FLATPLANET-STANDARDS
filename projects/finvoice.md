# Finvoice — Status Report
**Last updated:** 2026-04-22
**Session:** 3
**Version:** 1.3.2
**Status:** Active
**Repo:** https://github.com/FlatPlanet-Hub/Finvoice
**Owner:** Finance / Operations

---

## What This Project Is

Finvoice is an AU-facing invoice management system for Flat Planet Philippines. It handles staff billing invoices across multiple pricing models (new/old/offsite), salary increases, redundancy pay, overtime, night differential, and allowances. It integrates with Xero for invoice push/pull and includes a full suite of payroll calculation tools.

## Current State

The app is live on Netlify and fully functional for single-device use. All application data (invoices, clients, users, exchange rates) is stored in the browser's localStorage via Zustand persist middleware. A backend was designed (Azure Functions + Supabase, with the API client already written in src/lib/apiClient.js) but Azure has not been provisioned yet. Until the backend is connected, each user's browser has its own isolated data. Auth uses a local PBKDF2 password system with cross-device revocation via Netlify Blobs.

## What Was Done This Session

- Fixed critical auth bug: AuthContext.jsx was defaulting role to 'admin' when user record was null — changed to 'user' (most restrictive)
- Fixed invoice print route: /invoices/:id/print had no permission guard
- Fixed sidebar nav: Budget Targets section key was inconsistent; Gross Salary Solver was missing from nav entirely
- Fixed last-admin protection: deleteUser now prevents removing the last admin account
- Fixed allowlist wipe bug: empty localStorage browsers were wiping the server allowlist and locking out all users
- Created README.md, CHANGELOG.md, and docs/api.md (all were missing)
- Read full STANDARDS.md v3.3 and CLAUDE-local.md v1.7 (both current)

## What Is Next

1. Confirm admin user can log back in (if blocked: visit /api/rescue-allowlist?confirm=yes)
2. Remove unwanted user accounts via Settings > User Management
3. Backend migration: create Supabase tables and connect to existing apiClient.js
4. Fix git branch workflow: stop pushing directly to main, use dev > PR > main

## Open Issues

| ID | Priority | Description |
|----|----------|-------------|
| FIN-001 | High | Data isolation — users on different devices cannot see shared data. Requires backend. |
| FIN-002 | Medium | Git workflow — all commits pushed directly to main, violating standards. |
| FIN-003 | Medium | No unit tests exist anywhere in the project. |
| FIN-004 | Low | FlatPlanet Security Platform auth is disabled. App uses its own PBKDF2 auth. |

## Blockers

FIN-001 blocks multi-user collaboration. All other features work on a single device.

## Key Decisions on Record

- localStorage as interim storage: backend not yet provisioned, migration planned
- Local PBKDF2 password auth: SP auth disabled, to be revisited
- role defaults to 'user': safe fallback in AuthContext, not 'admin'
