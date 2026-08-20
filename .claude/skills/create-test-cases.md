# Skill: Create Test Cases for E2E Testing

**Purpose:** Draft test cases as CSV before implementing Playwright scripts, following the MANDATORY workflow in CLAUDE.md.

**Trigger:** Use this skill whenever you need to plan a new test flow (new `.spec.ts` file).

---

## Workflow (MANDATORY per CLAUDE.md)

### Step 1: Gather Requirements
Before drafting, clarify:
- **Feature/Flow name:** What are we testing? (e.g., "Admin Create Employee")
- **Scope:** Which scenarios? (Happy path? Validation? Edge cases?)
- **Business rules:** What makes this pass/fail? (from business-flows.md, API docs)

### Step 2: Draft Test Cases as CSV

**Location:** `Testcase/{testsuite_name}/{testsuite_name}.csv`  
**Example:** `Testcase/admin-create-employee/admin-create-employee.csv`

**CSV Columns (MUST match this order):**
```
Test Case ID,Feature,Priority,Category,Test Title,Pre-conditions,Steps,Expected Result
```

**Column Details:**

| Column | Format | Example |
|--------|--------|---------|
| **Test Case ID** | `E2E_{FEATURE}_{NUMBER}` | `E2E_ADMIN_CREATE_001` |
| **Feature** | Feature name | `Admin - Create Employee` |
| **Priority** | `[HIGH]` / `[MEDIUM]` / `[LOW]` | `[HIGH]` |
| **Category** | `Positive` / `Negative` / `Edge` | `Positive` |
| **Test Title** | What does this TC test? | `Create employee with required fields only` |
| **Pre-conditions** | State before test starts | `Admin logged in, on profile list page` |
| **Steps** | Numbered actions (UI-level, not code) | `1. Click Create\n2. Fill First Name='Test'\n3. Click Save` |
| **Expected Result** | What should happen? | `Toast 'Employee created' + redirect to detail page` |

### Step 3: Coverage & Balance

Before finalizing CSV, verify:
- ✅ **Positive cases** (happy path) — main success flow
- ✅ **Negative cases** (error scenarios) — validation, edge cases, invalid input
- ✅ **Edge cases** (boundary values) — empty strings, special chars, max length
- ✅ **Priority balance** — HIGH for critical, MEDIUM for important, LOW for edge
- ✅ **No duplicates** — each TC tests something unique

**Typical distribution:**
- ~40% Positive
- ~40% Negative  
- ~20% Edge/Boundary

### Step 4: Save CSV & Wait for Review

**Do NOT** write any `.spec.ts` or page object files yet.

**Prompt user:**
```
I've drafted {N} test cases for {feature} covering:
- {X} Positive cases (happy path)
- {Y} Negative cases (validation/errors)
- {Z} Edge cases

CSV saved: Testcase/{testsuite_name}/{testsuite_name}.csv

Please review and approve before I implement the Playwright scripts.
```

### Step 5: Wait for Approval

- ✅ User approves → proceed to implement `.spec.ts` + page objects
- ⚠️ User requests changes → update CSV, re-submit for approval
- ❌ User rejects → refactor TC strategy, re-draft

### Step 6: Implement After Approval

Once approved, create:
1. **Page Objects** (`src/pages/{path}/*.page.ts`)
2. **Test Specs** (`src/tests/{path}/*.spec.ts`)
3. **Test Data** (update `src/test-data/employee-factory.ts` if needed)

Each test case in CSV maps to one `test()` or `test.describe()` block in `.spec.ts`.

---

## CSV Example

**File:** `Testcase/admin-create-employee/admin-create-employee.csv`

```csv
Test Case ID,Feature,Priority,Category,Test Title,Pre-conditions,Steps,Expected Result
E2E_ADMIN_CREATE_001,Admin - Create Employee,[HIGH],Positive,Create employee with required fields,"Admin logged in, on /admin/profile page","1. Click 'Create' button
2. Fill First Name='TestUser'
3. Fill Last Name='QA'
4. Fill Email='testuser-{timestamp}@openwt.vn'
5. Select Gender='Male'
6. Select Contract Type='Full-time'
7. Select Position='QA Engineer'
8. Select Level='Junior'
9. Fill Start Date='2026-07-15'
10. Click 'Save'","Toast 'Employee created successfully' appears
Form closes
Redirected to /admin/profile/{employeeId}
Employee details page shows correct data"
E2E_ADMIN_CREATE_002,Admin - Create Employee,[HIGH],Negative,Reject creation with empty First Name,"Admin on create form, form is empty","1. Leave First Name empty
2. Fill other required fields correctly
3. Click 'Save'","Form validation error appears below First Name: 'First Name is required'
Form does NOT submit
User remains on create page"
E2E_ADMIN_CREATE_003,Admin - Create Employee,[MEDIUM],Negative,Reject invalid email format,"Admin on create form","1. Fill First Name='Test'
2. Fill Email='not-an-email'
3. Fill other required fields
4. Click 'Save'","Validation error: 'Please enter a valid email'
Form does NOT submit"
E2E_ADMIN_CREATE_004,Admin - Create Employee,[MEDIUM],Negative,Reject duplicate email,"Admin on create form
An employee with email 'existing@openwt.vn' already exists","1. Fill all fields
2. Fill Email='existing@openwt.vn'
3. Click 'Save'","Validation error: 'Email already exists'
Form does NOT submit"
E2E_ADMIN_CREATE_005,Admin - Create Employee,[MEDIUM],Positive,Cancel creation without saving,"Admin on create form, filled some fields","1. Fill First Name, Email, etc.
2. Click 'Cancel' button","Form closes
Redirected to /admin/profile (list page)
No employee created (verify via list)"
E2E_ADMIN_CREATE_006,Admin - Create Employee,[LOW],Edge,Create with maximum field lengths,"Admin on create form","1. Fill First Name with 100 chars
2. Fill Last Name with 100 chars
3. Fill Email with max valid length
4. Fill other fields
5. Click 'Save'","Employee created successfully
All fields preserved without truncation"
E2E_ADMIN_CREATE_007,Admin - Create Employee,[LOW],Positive,Create with all optional fields included,"Admin on create form","1. Fill all required fields
2. Fill optional: Phone, Address, DOB
3. Click 'Save'","Employee created successfully
All fields (required + optional) saved"
```

---

## When to Use This Skill

✅ **Use for:**
- Planning a new test flow (e.g., "Create Employee", "Edit Profile", "Approve Time-off")
- Before touching any `.spec.ts` or page object files
- Revising test strategy (rebuild CSV, get approval again)

❌ **Don't use for:**
- Updating an already-approved & implemented test
- Quick fixes to selectors (edit `.spec.ts` directly)
- Documentation-only tasks

---

## Integration with CLAUDE.md Workflow

This skill enforces the MANDATORY workflow in CLAUDE.md § "Test Case Workflow":

1. **Draft** (this skill) → CSV approved
2. **Implement** → `.spec.ts` + page objects
3. **Test** → Run and verify
4. **Iterate** → Fix failures, refactor

CSV acts as the **gate**: no code without approved test cases.

---

## Tips & Best Practices

### Tips

1. **Reference business-flows.md:** Each TC should tie to a real business rule from `business-flows.md` (e.g., "§8.2 - Admin Profile Management").
2. **Use test data placeholders:** `{timestamp}`, `{random}`, `{id}` — actual generation happens in code.
3. **Keep steps UI-level:** Don't mention code/selectors (e.g., NOT "click CSS selector `.btn-save"`; use "click Save button").
4. **Expected result is atomic:** One assertion per TC (one reason to fail).

### Checklist Before Submitting CSV

- [ ] All TC IDs unique and sequential
- [ ] Coverage: ≥1 Positive, ≥1 Negative, optional Edge cases
- [ ] Priority balanced (not all HIGH)
- [ ] Expected results specific & measurable (not "verify it works")
- [ ] No TC longer than 10 steps (if longer, split into 2)
- [ ] Pre-conditions are realistic (logged in, data exists, etc.)
- [ ] No hardcoded test data (use `{timestamp}`, `{email}`, etc.)
- [ ] Linked to business requirement (comment in CSV or mentioned in Steps)

---

## Example Workflow in Action

**User:** "I want to test Admin Edit Profile. What should I cover?"

**Claude (using this skill):**
1. Researches business-flows.md § 8.2.3 (Edit Profile Information)
2. Drafts 8 test cases:
   - 1 Positive: Edit basic info (HIGH)
   - 1 Positive: Edit with optional fields (HIGH)
   - 2 Negative: Validation (email invalid, required field empty) (MEDIUM)
   - 1 Negative: Cancel without saving (MEDIUM)
   - 1 Edge: Max field lengths (LOW)
   - 1 Edge: Edit then reload (verifying persistence) (LOW)
3. Saves to `Testcase/admin-edit-profile/admin-edit-profile.csv`
4. Prompts: "Review CSV? Any adjustments before I implement `.spec.ts`?"

**User:** Reviews, approves or requests changes.

**Claude:** Once approved, implements 6 `.spec.ts` tests matching the CSV exactly.

---

## Reference: CSV Template

Use this as a template:

```csv
Test Case ID,Feature,Priority,Category,Test Title,Pre-conditions,Steps,Expected Result
E2E_FEATURE_001,Feature Name,[PRIORITY],Positive,Test happy path,"Precondition 1, Precondition 2","1. Step 1
2. Step 2
3. Step 3","Expected result 1
Expected result 2"
```

Save as `Testcase/{feature-kebab-case}/{feature-kebab-case}.csv`

---

**Ready to draft test cases?** Call this skill and specify the feature you want to test.
