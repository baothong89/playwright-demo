---
name: qa-testing-standards
description: QA testing conventions — Priority levels, naming, patterns, selectors
metadata:
  type: project
---

# QA Testing Standards

Aligned with existing API testing project (see `../API testing/Must read/QA_Agent_Instruction_v4.md`).

## Priority Levels (from API testing project)

| Priority | When to Use | Examples |
|----------|-------------|----------|
| **HIGH** | Critical workflows, core features, happy path | Login, create employee, approve time-off, list data |
| **MEDIUM** | Important secondary flows, required but less frequent | Edit profile, filter table, export PDF |
| **LOW** | Edge cases, nice-to-have, validation errors | Invalid email format, boundary values, optional fields |

**Test tag format:**
```typescript
test('[HIGH] Admin can create new employee', async ({ adminPage }) => { ... });
test('[MEDIUM] Sort employees by name', async ({ adminPage }) => { ... });
test('[LOW] Reject invalid email on profile edit', async ({ adminPage }) => { ... });
```

## Test Data Management (DEV is Shared)

**Problem:** DEV is a real shared environment for QA. Tests must not pollute it.

**Rules:**
1. **Unique Identifiers** — Use timestamp or UUID suffix:
   ```typescript
   const email = `test-e2e-${Date.now()}@openwt.vn`;
   ```

2. **Cleanup After Test** — Delete created data in `afterEach()`:
   ```typescript
   test.afterEach(async ({ page }) => {
     // Call DELETE API to remove test employee
     await page.request.delete(`/v1/admin/users/${testEmployeeId}`);
   });
   ```

3. **Reuse Test Fixtures** — Don't recreate the same employee 100 times:
   ```typescript
   // Good: Create once per describe block
   let testEmployeeId: number;
   test.beforeAll(async () => {
     testEmployeeId = await createTestEmployee();
   });
   
   // Bad: Create in every test
   test('scenario 1', async () => {
     const id = await createTestEmployee();
   });
   ```

## Naming Conventions

| What | Pattern | Example |
|------|---------|---------|
| **Test file** | `{feature}-{scenario}.spec.ts` | `create-employee.spec.ts`, `edit-profile.spec.ts` |
| **Test suite** | `[FEATURE] Description` | `[ADMIN] Profile Management` |
| **Test name** | `[PRIORITY] Specific action` | `[HIGH] Admin can create new employee` |
| **Page class** | `{PageName}Page` | `ProfileListPage`, `EmployeeCVPage` |
| **Page method** | Verb-based, async | `async clickCreate()`, `async fillEmail()`, `async isLoaded()` |
| **Variable** | camelCase, descriptive | `testEmployeeId`, `profileUpdateForm`, `errorMessage` |

## Selector Strategy

No `data-testid` attributes exist yet in FE. Preference order:

1. **Semantic Locators** (most robust, future-proof):
   ```typescript
   page.getByRole('button', { name: 'Save' })
   page.getByLabel('First Name')
   page.getByText('Employee Profile')
   ```

2. **`data-testid`** (when FE adds them incrementally):
   ```typescript
   page.locator('[data-testid="profile-save-btn"]')
   ```

3. **CSS/XPath** (fallback, less robust):
   ```typescript
   page.locator('button[type="submit"]:has-text("Save")')
   page.locator('input#firstName')
   ```

**Never rely on:** Unstyled class names (`.ng-untouched`), dynamic IDs, text-only on dynamic content.

## Test Structure (AAA Pattern)

```typescript
test('[HIGH] Admin can edit employee profile', async ({ adminPage }) => {
  // Arrange — setup page, get test data
  const profileEdit = new ProfileEditPage(adminPage);
  await profileEdit.goto(testEmployeeId);
  
  // Act — perform user action
  await profileEdit.fillFirstName('John');
  await profileEdit.fillLastName('Doe');
  await profileEdit.clickSave();
  
  // Assert — verify results
  await expect(adminPage).toHaveURL(/\/admin\/profile\/\d+/); // Stayed on detail page
  const successMsg = adminPage.locator('[role="alert"]:has-text("Success")');
  await expect(successMsg).toBeVisible();
  
  // Cleanup — delete test data
  // (if not using afterEach hook)
});
```

## Page Object Patterns

**Good:**
```typescript
export class ProfileListPage {
  readonly page: Page;
  readonly createButton: Locator;  // Store as property
  
  constructor(page: Page) {
    this.page = page;
    this.createButton = page.locator('button:has-text("Create")');
  }
  
  async goto() {
    await this.page.goto('/admin/profile', { waitUntil: 'networkidle' });
  }
  
  async clickCreate() {
    await this.createButton.click();
    await this.page.waitForURL(/profile.*create/);
  }
}
```

**Avoid:**
```typescript
// Bad: querying in method (repeated queries, slower)
async clickCreate() {
  await this.page.locator('button:has-text("Create")').click();
}

// Bad: returning raw locators (test has to manage waits)
getCreateButton() {
  return this.page.locator('button:has-text("Create")');
}
```

## Comment Guidelines

- **No comments on WHAT code does** — good naming already says that
- **Comment WHY** when non-obvious:
  ```typescript
  // Keycloak may show a consent/approval page on first login
  const approveBtn = page.locator('button:has-text("Approve")').first();
  if (await approveBtn.isVisible({ timeout: 2000 }).catch(() => false)) {
    await approveBtn.click();
  }
  ```
- **No step-by-step comments**:
  ```typescript
  // Bad:
  // Click the button
  await button.click();
  // Wait for redirect
  await page.waitForURL(/new-page/);
  
  // Good:
  await button.click();
  await page.waitForURL(/new-page/);
  ```

## CI/CD Alignment

- Tests run **nightly** in GitHub Actions (2 AM UTC)
- **Manual trigger** available via workflow_dispatch
- **Artifacts:** Playwright HTML report + JSON results
- **Secrets:** ADMIN_PASSWORD, EMPLOYEE_PASSWORD (real test account credentials)
- **Environment:** Always DEV (`https://employee-dev.owt.vn`), never production

**Local runs** use same `.env` file (test credentials), no need to recreate auth sessions once created.
