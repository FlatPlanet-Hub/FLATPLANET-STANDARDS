# FLATPLANET Standards
> Version: 2.0 | Last updated: 2026-03-19
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

"Please read the FlatPlanet standards: https://raw.githubusercontent.com/FlatPlanet-Hub/FLATPLANET-STANDARDS/main/FLATPLANET-STANDARDS/STANDARDS.md and the data dictionary: https://raw.githubusercontent.com/FlatPlanet-Hub/FLATPLANET-STANDARDS/main/FLATPLANET-STANDARDS/DATA_DICTIONARY.md -- then I will tell you what I need."

This ensures Claude knows all FlatPlanet rules before it starts working.

### Prompt Templates

Copy and paste these. Replace anything in [brackets] with your information.

To start a new project:
"Please read the FlatPlanet standards and data dictionary. We are starting a new project called [name]. It does [description]. The tech stack is [stack]. Please set up the project structure following FlatPlanet standards, including CLAUDE.md, README.md, and CHANGELOG.md."

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

Version control tracks what changed, when, and why. Claude handles Git. Everyone is responsible for knowing the current version and keeping the changelog accurate.

### Version Number Format: MAJOR.MINOR.PATCH

| Number | When it changes | Example |
|--------|-----------------|---------|
| MAJOR | Project fundamentally changed or incompatible with previous version | 1.0.0 to 2.0.0 |
| MINOR | New feature added | 1.0.0 to 1.1.0 |
| PATCH | Bug fixed or small update | 1.0.0 to 1.0.1 |

Every project starts at version 1.0.0.

### Git Branch Strategy

| Branch | Purpose |
|--------|---------|
| main | Production -- always working, always protected |
| dev | Integration -- features come here before main |
| feature/short-name | One feature or one Claude session |
| fix/short-name | Bug fixes |
| docs/short-name | Documentation-only changes |

### Commit Message Format

  type: short description (max 72 characters)
  Types: feat | fix | docs | style | refactor | test | chore

Examples:
  feat: add monthly report export for operations dashboard
  fix: resolve export button not responding on Safari
  docs: update README with new setup steps
  chore: upgrade all dependencies to latest stable

---

## 4. Required Documentation

Every FlatPlanet project must always have these files maintained and up to date:

| File | What it contains | Who updates it |
|------|-----------------|----------------|
| README.md | What the project does, how to use it, current status | Claude |
| CHANGELOG.md | Full version history of every change | Claude |
| CLAUDE.md | Technical brief for Claude Code | Developer or Claude |
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
- Request parameters with types from the Data Dictionary
- Response shape with example
- Error codes it can return

Claude must update docs/api.md automatically whenever any API endpoint is added or changed.

---

## 5. Project Structure

Every FlatPlanet project must follow this structure:

  project-name/
    CLAUDE.md              - Claude reads this first (required)
    README.md              - Plain English project overview (required)
    CHANGELOG.md           - Full version history (required)
    src/                   - All source code, organized by domain
    tests/                 - Mirrors src/ structure
    docs/
      api.md               - Auto-generated API documentation
      decisions/           - Why key decisions were made
    scripts/               - Dev utilities and automation
    .env.example           - All required env vars (never commit .env)

---

## 6. CLAUDE.md -- Required in Every Project

Claude Code reads CLAUDE.md first when opening any project. Every CLAUDE.md must contain:

  # [Project Name] -- Claude Code Brief

  ## What this project does
  [2-3 sentences in plain English]

  ## Current version
  [X.X.X]

  ## Tech stack
  - Language: [e.g. TypeScript]
  - Framework: [e.g. Next.js]
  - Database: [e.g. PostgreSQL]
  - Hosting: [e.g. Vercel]

  ## Key commands
  - Start dev server: [command]
  - Run tests: [command]
  - Build for production: [command]

  ## FlatPlanet Standards (always read before coding)
  - STANDARDS: https://raw.githubusercontent.com/FlatPlanet-Hub/FLATPLANET-STANDARDS/main/FLATPLANET-STANDARDS/STANDARDS.md
  - DATA DICTIONARY: https://raw.githubusercontent.com/FlatPlanet-Hub/FLATPLANET-STANDARDS/main/FLATPLANET-STANDARDS/DATA_DICTIONARY.md

  ## Documentation rules
  - After every feature: update CHANGELOG.md (increment MINOR) and README.md
  - After every bug fix: update CHANGELOG.md (increment PATCH)
  - After every breaking change: update CHANGELOG.md (increment MAJOR), README.md, and docs/api.md
  - After every API change: update docs/api.md
  - Commit format: type: short description

  ## Important context for Claude
  [Project-specific rules, known issues, or anything Claude must know before starting]

---

## 7. Naming Conventions

All variable names must come from the FlatPlanet Data Dictionary where applicable.

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

1. Read standards first -- Fetch STANDARDS.md and DATA_DICTIONARY.md before writing any code
2. Confirm the task -- Restate the task in plain English before starting so anyone can verify Claude understood correctly
3. Follow the Data Dictionary -- All variable names must match the dictionary where applicable
4. Update documentation -- Every session that changes code must update CHANGELOG.md and any affected docs
5. Write tests -- Every new function gets at least one test
6. Commit cleanly -- Each logical change gets its own commit with a clear message
7. End with a summary -- Every session ends with a plain English summary of what was done

### Session Summary Template

Claude must output this at the end of every session:

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
  - docs/api.md: yes / no / not applicable

  Next steps:
  - [Any follow-up tasks or things still to do]

---

## 12. Pull Request Rules

- Every PR must have a plain English description of what changed and why
- Every PR must pass all tests before merging
- Every PR that changes user-facing behavior must include updated documentation
- PRs into main require at least one review
- PR title follows commit format: type: short description

---

## 13. Standards Maintenance

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

Last updated: 2026-03-19
Version: 2.0
Maintained by: FlatPlanet-Hub
One standard, every project, every person.
