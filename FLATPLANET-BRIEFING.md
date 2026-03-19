# FLATPLANET-STANDARDS — Project Briefing

**Version:** 1.0
**Date:** 2026-03-19
**Repository:** https://github.com/FlatPlanet-Hub/FLATPLANET-STANDARDS
**Owner:** FlatPlanet-Hub

---

## What Is This?

FlatPlanet is building a new technology stack centred on Claude (Anthropic AI). The goal is to allow business line managers — not just developers — to experiment and spin up new projects quickly, while ensuring every project automatically meets the same quality, naming, and structural standards.

`FLATPLANET-STANDARDS` is the **central standards hub**. It is a public GitHub repository that:

- Defines the rules all FlatPlanet projects must follow
- Is referenced by every FlatPlanet project from day one
- Uses Claude Code as the primary enforcement engine
- Grows over time as new standards are agreed

There is no manual compliance process. Claude reads this repository at the start of every project session and enforces the rules continuously.

---

## The Problem Being Solved

Without a central standard:
- Every project names things differently
- Business managers spinning up new projects make inconsistent decisions
- Claude gives different answers in different projects
- There is no shared language between projects

With FLATPLANET-STANDARDS:
- One name for every concept, across every project
- New projects are scaffolded from templates that are already compliant
- Claude enforces the rules automatically in every session
- Business managers get guardrails without needing to understand the technical detail

---

## Current State of the Repository

The repository currently contains two documents inside `/FLATPLANET-STANDARDS/`:

### STANDARDS.md
The operating rulebook. Covers:
- Version format: `MAJOR.MINOR.PATCH`
- Required files every project must have
- Git branching strategy (`main`, `dev`, `feature/*`, `fix/*`, `docs/*`)
- Code conventions (2-space indent, 100-char line limit, camelCase, PascalCase, max 40 lines per function)
- Testing requirements

### DATA_DICTIONARY.md
The naming and data bible. Covers:
- ID format: prefixed ULIDs — always strings, never integers
- Canonical data types (money as integer cents, timestamps as ISO 8601, percentages as 0.0–1.0)
- Universal field names (`id`, `created_at`, `updated_at`, `deleted_at` on every table)
- Entity schemas: User, Organisation, Project, Task, API Key, Audit Log
- Standard enum values (closed lists — no inventing new status values)
- Forbidden variable names (`data`, `temp`, `obj`, `item`, `res`, `cb`, `e`, etc.)
- API design conventions (plural nouns, versioned URLs, standard error codes)
- Database conventions (snake_case, soft deletes, index all FKs)
- Frontend conventions (React PascalCase, `use` prefix hooks, `--fp-` CSS vars)
- Environment variable patterns (`FP_`, `DB_`, `AUTH_`, `FF_` prefixes)
- Logging standards (structured logs, required fields, five log levels)
- Feature flag conventions (`FF_` prefix, rollout percentage as 0.0–1.0)

---

## What Needs to Be Built

The repository needs to grow from documentation into a functioning system. The build order is:

### Phase 1 — Claude Integration (Highest Priority)
**`CLAUDE.md`** at the repo root.
This is the instruction file Claude Code reads automatically. It tells Claude:
- What this repo is
- That it must check DATA_DICTIONARY.md before naming anything
- That it must check STANDARDS.md before structuring anything
- How to behave in any FlatPlanet project that references this repo

### Phase 2 — Project Templates
**`/templates/`** directory.
Starter scaffolds for new projects, pre-wired with:
- The correct file structure
- A CLAUDE.md that points back to this repo
- Placeholder README, CHANGELOG, .env.example
- Correct folder layout

### Phase 3 — Shared Configs
**`/configs/`** directory.
Shareable configuration files projects can extend:
- ESLint config
- Prettier config
- TypeScript config
- Git hooks (pre-commit checks)

### Phase 4 — Reusable GitHub Actions
**`/workflows/`** directory.
Reusable CI workflows any project can call:
- Standards compliance check on every PR
- Version bump and changelog update
- Dependency audit

### Phase 5 — Compliance Checker Script
**`/scripts/compliance-check.js`**
A script that scans a project directory and reports:
- Missing required files
- Naming violations
- Version format errors
- Missing fields on DB entities

---

## How Claude Enforces the Standards

Every FlatPlanet project will have a `CLAUDE.md` at its root. That file instructs Claude Code to:

1. Read `FLATPLANET-STANDARDS/DATA_DICTIONARY.md` before naming any variable, field, or entity
2. Read `FLATPLANET-STANDARDS/STANDARDS.md` before creating any file, folder, or branch
3. Refuse to use forbidden variable names
4. Always use the correct data types (e.g. never use float for money)
5. Always use prefixed ULIDs for IDs
6. Flag any deviation from the standard and explain why

Because Claude reads these files at the start of every session, enforcement is continuous and automatic — no extra tooling required.

---

## Roles

| Role | Responsibility |
|------|---------------|
| FlatPlanet-Hub (org) | Owns and maintains the standards repo |
| Claude Code | Enforces standards in every project session |
| Business Line Managers | Consume templates to spin up new projects |
| Developers | Propose changes via PR; extend templates |

---

## Key Decisions Already Made

- Tech stack is Claude-native (Anthropic AI at the centre)
- GitHub is the hosting platform (organisation: FlatPlanet-Hub)
- Standards live in one public repo, referenced by all other repos
- Claude Code is the primary enforcement engine — not linters or CI alone
- Business managers are first-class users — simplicity is a requirement

---

## Immediate Next Steps (Checklist)

- [ ] Write `CLAUDE.md` for the FLATPLANET-STANDARDS repo root
- [ ] Update `README.md` to properly explain the repo's purpose
- [ ] Create `/templates/` with a base project scaffold
- [ ] Create `CHANGELOG.md` for the standards repo itself
- [ ] Create `/configs/` with shared ESLint + Prettier + TypeScript configs
- [ ] Create `/workflows/` with a reusable GitHub Actions compliance check
- [ ] Create `/scripts/compliance-check.js`
- [ ] Write onboarding guide: how a business manager starts a new project

---

*This briefing document is the input for generating the full system design document and implementation checklist.*
