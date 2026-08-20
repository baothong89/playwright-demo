# Project Memory Index

Quick reference to persistent context for this E2E testing project.

## Memory Files

- [tech-architecture.md](tech-architecture.md) — Stack, authentication flow, API structure, roles & features
- [qa-standards.md](qa-standards.md) — Priority levels, naming conventions, test data management, selector strategy, page object patterns
- [phase-progress.md](phase-progress.md) — Implementation status (Phase 0 ✅, Phase 1-4 roadmap), completed tasks, upcoming work

## Key Context

### Project
- **Name:** E2E Testing Automation for OWT Employee App
- **Stack:** Playwright + TypeScript, Angular 16 FE, NestJS BE, Keycloak SSO
- **Environment:** DEV (`https://employee-dev.owt.vn`)
- **Status:** Phase 0 complete (auth setup), Phase 1 ready (Admin Profile & CV)

### How to Start Work
1. Read `phase-progress.md` to see what's done and what's next
2. Read `tech-architecture.md` if you need stack/auth details
3. Read `qa-standards.md` for naming & conventions before writing tests
4. Run `npm test` to set up auth (first time) or run tests

### Important Files
- `CLAUDE.md` — Full project instructions
- `README.md` — Getting started guide
- `playwright.config.ts` — Test configuration (baseURL, reporters, global setup)
- `global-setup.ts` — Keycloak login script (runs once before tests)
- `src/fixtures/auth.fixture.ts` — Auth context for tests
- `.env.example` → `.env` — Test credentials (ADMIN/EMPLOYEE passwords)

### Quick Commands
```bash
npm test              # All tests (includes auth setup 1st time)
npm run test:smoke    # Smoke tests only (fast)
npm run test:headed   # With browser visible (for development)
npm run test:debug    # Debug mode with Playwright Inspector
npm run report        # View last HTML report
```

### Decisions Logged
- **Auth:** Keycloak SSO via global-setup (login once, save storageState, reuse across tests)
- **Selectors:** Semantic locators preferred (getByRole, getByLabel, getByText); no data-testid yet in FE; incrementally add as needed
- **Data:** Unique test data on DEV (use timestamp suffix); cleanup after each test
- **Priority tags:** [HIGH] / [MEDIUM] / [LOW] aligned with QA_Agent_Instruction_v4.md
- **CI:** Nightly run via GitHub Actions (not yet enabled, needs secrets setup)

---

**Last synced:** 2026-07-15
