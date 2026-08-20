---
name: implementation-phase-progress
description: Phase-by-phase implementation status, completed tasks, upcoming work
metadata:
  type: project
---

# Phase Progress & Roadmap

## Phase 0: Scaffold & Auth — ✅ COMPLETE

**Goal:** Establish Playwright project with Keycloak SSO authentication.

**Completed:**
- ✅ Project initialization (`npm init playwright`)
- ✅ TypeScript config + path aliases
- ✅ `playwright.config.ts` (baseURL, reporters, global setup)
- ✅ `global-setup.ts` (Keycloak login for Admin + Employee, storageState save)
- ✅ `auth.fixture.ts` (authenticated page fixtures)
- ✅ Smoke test: Admin login + Profile List page load
- ✅ Package.json scripts (`npm test`, `npm run test:smoke`, etc.)
- ✅ Documentation (README.md, QUICKSTART.md, CLAUDE.md)
- ✅ CI/CD workflow skeleton (`.github/workflows/e2e-tests.yml`)
- ✅ .gitignore, .env.example
- ✅ Memory system (.claude/memory/)

**Status:** Ready to move to Phase 1. Smoke test should pass once `.env` has real credentials and global setup runs successfully.

---

## Phase 1: Admin Profile & CV Management — ⚡ IN PROGRESS (Tasks 1.1-1.3 DONE)

**Goal:** Automate core Admin workflow — create employees, edit profiles, manage CV sections.

**Prerequisite:**
- Smoke test passes (validates auth + basic page load)
- `.env` has real `ADMIN_PASSWORD` and `EMPLOYEE_PASSWORD`
- `npm test` has created `auth/storageState/admin.json`

### Tasks to Implement

#### 1.1 Profile List Page Object & Tests — ✅ COMPLETE
- **Page Object:** ✅ `src/pages/admin/profile/profile-list.page.ts` 
  - ✅ Methods: `searchByName()`, `filterByPosition()`, `getEmployeeByName()`, `clickEmployeeRow()`, `sortByColumn()`, `setPageSize()`, `verifyTableStructure()`
  - ✅ All selectors for table rows, search, filters, pagination

- **Tests:** ✅ `src/tests/admin/profile/profile-list.spec.ts` (8 tests)
  - ✅ `[HIGH]` View employee profile list with table loaded
  - ✅ `[HIGH]` See employee count
  - ✅ `[MEDIUM]` Search employees by name
  - ✅ `[MEDIUM]` Clear search filter
  - ✅ `[MEDIUM]` See Create button
  - ✅ `[MEDIUM]` Navigate to create form
  - ✅ `[LOW]` Verify table columns
  - ✅ `[LOW]` Navigate back from create

**Status:** Ready to run `npm run test:admin -- --grep "Profile List"`

#### 1.2 Create Employee Workflow — ✅ COMPLETE
- **Page Object:** ✅ `src/pages/admin/profile/profile-create.page.ts`
  - ✅ All form fields: First Name, Last Name, Email, Company Email, Gender, Contract Type, Position, Level, Start Date, DOB, Phone, Address
  - ✅ Methods: `fillEmployeeForm()`, `selectOption()`, `clickSave()`, `clickCancel()`, `getErrorMessage()`, `verifyRequiredFields()`, `hasValidationError()`

- **Tests:** ✅ `src/tests/admin/profile/create-employee.spec.ts` (8 tests)
  - ✅ `[HIGH]` Create employee with minimal required fields
  - ✅ `[HIGH]` Create employee with all fields filled
  - ✅ `[MEDIUM]` Reject empty required fields
  - ✅ `[MEDIUM]` Reject invalid email format
  - ✅ `[MEDIUM]` Cancel create without saving
  - ✅ `[MEDIUM]` Newly created employee appears in list
  - ✅ `[LOW]` Required fields are marked
  - ✅ `[LOW]` Duplicate email validation

- **Test Data:** ✅ Using `createMinimalEmployee()` / `createCompleteEmployee()` factories with unique timestamp emails
- **Cleanup:** ✅ `afterEach()` deletes created employee via API

**Status:** Ready to run `npm run test:admin -- --grep "Create Employee"`

#### 1.3 Edit Profile Information — ✅ COMPLETE
- **Page Object:** ✅ `src/pages/admin/profile/profile-detail.page.ts` (view detail)
  - ✅ Methods: `goto(id)`, `getFirstName()`, `getLastName()`, `getEmail()`, `getPosition()`, `getLevel()`, `clickEdit()`, `clickEmployeeCV()`, `verifyProfileData()`
  
- **Page Object:** ✅ `src/pages/admin/profile/profile-edit.page.ts` (edit form)
  - ✅ All form fields + optional fields (Phone, Address, DOB, University, ID Number)
  - ✅ Methods: `fillForm()`, `updateField()`, `getFieldValue()`, `clickSave()`, `clickCancel()`, `clickReset()`, `hasUnsavedChanges()`

- **Tests:** ✅ `src/tests/admin/profile/edit-profile.spec.ts` (6 tests)
  - ✅ `[HIGH]` Edit employee basic information (name, email)
  - ✅ `[MEDIUM]` Edit optional fields (DOB, phone, address)
  - ✅ `[MEDIUM]` Cancel edit without saving
  - ✅ `[MEDIUM]` Edit form shows current values pre-populated
  - ✅ `[LOW]` Form validates required fields on edit
  - ✅ `[LOW]` Admin can reset unsaved changes

- **Test Data Setup:** ✅ Create test employee once in `beforeAll()`, cleanup in `afterAll()`, reuse across all tests

**Status:** Ready to run `npm run test:admin -- --grep "Edit Profile"`

#### 1.4 Employee CV — Add/Edit/Delete Sections
- **Page Object:** `src/pages/admin/profile/employee-cv.page.ts`
  - Methods per section: `addEducation()`, `editEducation()`, `deleteEducation()`, etc.
  - Sections: Education, Certifications, Skills, Employment History, Experiences
  - Generic method: `addSection(sectionName, fields)`, `deleteRow(sectionName, rowIndex)`

- **Tests:** `src/tests/admin/profile/employee-cv.spec.ts`
  - `[HIGH]` Add education entry to CV
  - `[HIGH]` Edit education entry
  - `[HIGH]` Delete education entry
  - `[MEDIUM]` Add certification, employment history, experience (repeat for each section)
  - `[MEDIUM]` Reorder CV sections (drag-and-drop)
  - `[MEDIUM]` Toggle "include in PDF" checkbox
  - `[LOW]` Invalid dates rejected (end before start)

**Effort:** ~12 tests, 1 page object (complex due to drag-and-drop, multiple sections)

#### 1.5 Save to PDF
- **Page Object:** Extend `employee-cv.page.ts` with `clickSaveToPDF()`, `waitForDownload()`

- **Tests:** `src/tests/admin/profile/save-to-pdf.spec.ts`
  - `[MEDIUM]` Save CV to PDF with all sections
  - `[MEDIUM]` Save CV to PDF with selected sections only (via checkboxes)
  - `[LOW]` Verify PDF file downloaded (or response headers if mocked)

**Effort:** ~3 tests, minimal page object work

### Summary for Phase 1
- **Test files:** 5 new files (~25 test cases)
- **Page objects:** 4 new files
- **Timeline estimate:** 2–3 weeks (including refinement, selector adjustments)
- **Success criteria:**
  - All tests pass locally with `npm test:admin`
  - Cleanup removes all test data from DEV
  - No data pollution detected on Profile List

---

## Phase 2: Employee Workflows — PLANNED

**Goal:** Time-off requests, WFH requests, attendance, vacation balance.

### Tasks (sketch)
- **Time-off request:** Employee submit → list → Admin approve/refuse → balance calculation
- **WFH request:** Similar to time-off, no balance interaction
- **Attendance overview:** Check-in/check-out, tabs for In-Office / On Vacation / WFH / Others
- **Vacation balance detail:** View balance numbers, request history

**Effort estimate:** 3–4 weeks, ~10 test cases, 5–6 page objects

---

## Phase 3: Device & Buddy Management — PLANNED

**Goal:** Device CRUD, issue reporting; Buddy pairing, touchpoints.

### Tasks (sketch)
- **Device management:** List, detail, assign/unassign, repair history
- **Issue reporting:** Employee reports problem → Admin triages → close
- **Buddy program:** Create buddy, create pair, write touchpoint (public/private)

**Effort estimate:** 3–4 weeks, ~15 test cases, 8–10 page objects

---

## Phase 4: Task Management — PLANNED

**Goal:** Task list, Kanban board, comments, subtasks, timer.

**Effort estimate:** 2–3 weeks, ~10 test cases, 3–4 page objects

---

## CI/CD Setup — PENDING

**Current state:**
- GitHub Actions workflow written (`.github/workflows/e2e-tests.yml`)
- **Not yet enabled** — needs:
  1. GitHub secrets configured: `ADMIN_USERNAME`, `ADMIN_PASSWORD`, `EMPLOYEE_USERNAME`, `EMPLOYEE_PASSWORD`
  2. Actions enabled in GitHub repo settings
  3. First manual trigger to verify

**Task:** Set up GitHub secrets, enable workflow, test nightly run.

**Timeline:** ~30 min (manual GitHub setup)

---

## Technical Debt & Future Improvements

| Item | Priority | Notes |
|------|----------|-------|
| Add `data-testid` to FE | Medium | Incrementally, as tests are written. Improves selector robustness. |
| Parallel test execution | Low | Currently `fullyParallel: false` due to shared DEV. Consider data isolation if needed. |
| Performance baseline | Low | Track average test runtime, alert on regressions. |
| Cross-browser testing | Low | Add Firefox/Safari if needed for production. Currently Chrome only. |
| Visual regression | Low | Screenshot comparisons for UI changes (e.g., form layouts). |
| Reusable test fixtures | Medium | If Phase 2/3 need shared test data setup (e.g., pre-created employee). |

---

## Next Immediate Action

**Start Phase 1.1 (Profile List):**
1. Expand `src/pages/admin/profile/profile-list.page.ts` with additional selectors
2. Create `src/tests/admin/profile/profile-list.spec.ts`
3. Run `npm run test:headed` to develop + debug
4. Commit & document

**Blocker check:**
- [ ] `.env` has real credentials
- [ ] `npm test` runs global setup successfully
- [ ] Smoke test passes (`npm run test:smoke`)
- [ ] `auth/storageState/admin.json` exists

Once unblocked, start Phase 1 implementation.
