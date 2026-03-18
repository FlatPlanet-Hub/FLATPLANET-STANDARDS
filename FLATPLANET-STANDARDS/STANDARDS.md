# FLATPLANET Coding Standards

> This file is automatically fetched when a new project is created and serves as the base context for all Claude Code sessions.

---

## 1. General Principles

- Clarity over cleverness: Write code that is easy to read, not code that shows off.
- Consistency: Follow these conventions across all projects, regardless of language.
- Small focused functions: Each function should do one thing and do it well.
- Fail loudly: Errors should be explicit and descriptive, never silent.
- No dead code: Remove commented-out code and unused imports before committing.

---

## 2. Project Structure

Every FlatPlanet project must follow this structure:

  CLAUDE.md          - Claude Code context file (REQUIRED in every project)
  README.md          - Project overview
  src/               - All source code organized by domain
  tests/             - Mirrors src/ structure
  docs/              - Architecture decisions and API docs
  scripts/           - Dev utilities and seed scripts
  .env.example       - Environment variable template (never commit .env)

---

## 3. CLAUDE.md Requirement

Every project MUST have a CLAUDE.md at the root. Claude Code reads this first.

Required contents:
- Project purpose (2-3 sentences)
- Tech stack summary
- Key commands: dev, test, build
- Links to standards:
  - STANDARDS: https://raw.githubusercontent.com/FlatPlanet-Hub/FLATPLANET-STANDARDS/main/FLATPLANET-STANDARDS/STANDARDS.md
  - DATA DICTIONARY: https://raw.githubusercontent.com/FlatPlanet-Hub/FLATPLANET-STANDARDS/main/FLATPLANET-STANDARDS/DATA_DICTIONARY.md

---

## 4. Naming Conventions

| Element          | Convention      | Example             |
|------------------|-----------------|---------------------|
| Files            | kebab-case      | user-profile.ts     |
| Folders          | kebab-case      | auth-service/       |
| Variables        | camelCase       | userId              |
| Constants        | UPPER_SNAKE     | MAX_RETRY_COUNT     |
| Classes          | PascalCase      | UserProfile         |
| Functions        | camelCase verb  | getUserById         |
| Database tables  | snake_case      | user_profiles       |
| Database columns | snake_case      | created_at          |
| Env variables    | UPPER_SNAKE     | DATABASE_URL        |

---

## 5. Code Style

- Indent: 2 spaces (no tabs)
- Max line length: 100 characters
- Strings: single quotes
- Trailing commas in multi-line structures: yes
- Max function length: 40 lines
- Max parameters: 4 (use an options object if more are needed)
- Comments explain WHY, not WHAT
- No commented-out code in commits
- Always define return types explicitly in typed languages

---

## 6. Git and Branching

Branch strategy:
- main: production-ready, protected branch
- dev: integration branch, all features merge here first
- feature/short-description: one branch per task or Claude Code session

Commit message format:
  type: short description (max 72 characters)

  Types: feat | fix | docs | style | refactor | test | chore

Examples:
  feat: add user authentication flow
  fix: resolve null pointer on empty cart
  docs: update CLAUDE.md with new env vars
  chore: upgrade dependencies to latest

---

## 7. Testing

- Every new function must have at least one unit test
- Test files live in tests/ mirroring the src/ folder structure
- File naming: filename.test.ts or filename.spec.ts
- All tests must pass before any PR is merged to dev or main

---

## 8. Environment and Secrets

- NEVER commit .env files — add to .gitignore immediately
- Always keep .env.example up to date with all required keys
- No hardcoded credentials anywhere in code
- All secrets go through environment variables only

---

## 9. Claude Code Usage Guidelines

- Always fetch this STANDARDS.md at the start of a new project
- Always fetch DATA_DICTIONARY.md for variable naming reference
- Use CLAUDE.md to provide Claude with project-specific context
- Be explicit: state the tech stack, the goal, and any constraints upfront
- Review ALL Claude-generated code before committing
- Do not commit code you do not understand

---

Last updated: 2026-03-18
Maintained by: FlatPlanet-Hub
Repository: https://github.com/FlatPlanet-Hub/FLATPLANET-STANDARDS
