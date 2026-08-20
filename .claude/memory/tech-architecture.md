---
name: tech-stack-and-auth
description: Technical architecture — stack, authentication flow, environment URLs
metadata:
  type: project
---

# Technical Architecture

## Stack
- **Frontend:** Angular 16 (TypeScript), module federation, NGXS state, Bootstrap 5 + ng-bootstrap
- **Backend:** NestJS (TypeScript), TypeORM + Postgres, Redis, JWT/Keycloak
- **E2E Framework:** Playwright 1.61+ with TypeScript
- **Test Runner:** @playwright/test (built-in)
- **CI/CD:** GitHub Actions (nightly + manual trigger)

**Development URLs:**
- FE: http://localhost:4200 (local) | https://employee-dev.owt.vn (dev)
- BE API: https://dev.api.employee.openwt.vn
- Keycloak Auth: https://keycloak-test.openwt.vn (realm: owtvn-dev, client: employee-app)

## Authentication Flow

App uses **Keycloak SSO** (not local JWT). When unauthenticated:
1. FE app loads → keycloak-js detects no session → force-redirect to Keycloak login
2. User enters credentials on keycloak-test.openwt.vn
3. Keycloak creates SSO session (cookies) + issues access token
4. Redirects back to app → FE stores token in localStorage + NGXS state
5. FE calls `GET /v1/auth/me` → BE returns `UserDto` with `role: ADMIN | USER`

**In E2E tests:** Global setup handles step 1-4 once per role, saves auth state (cookies + localStorage) to JSON. All tests reuse this state via auth fixture.

## Key Roles
- **ADMIN**: Full access to `/admin/*` routes (user management, attendance oversight, device inventory, buddy program)
- **USER** (Employee): Access to `/profile/*`, `/attendance/*`, `/device/*`, `/buddy/*`, `/tasks/*` (own data only)

## API Base URL
- Environment config in FE: `src/environments/environments.ts` → `apiBaseUrl: 'https://dev.api.employee.openwt.vn'`
- All API calls use this base (no hardcoded domain in tests)

## Main Business Features (target for E2E)
1. **Profile & CV** (Admin management, user self-service)
2. **Attendance** (time-off requests, WFH, vacation balance)
3. **Device Management** (inventory, assignment, issue reporting)
4. **Buddy Program** (pairing, touchpoints/feedback)
5. **Task Assignment** (tickets, Kanban, comments)

**Phase 1 focus:** Profile & CV admin workflows
