---
description: Analyze recent commits, identify UI flows to test, and run automated browser tests using Playwright MCP.
allowed-tools: [Bash, Read, Glob, Grep, Task, TodoWrite, TaskOutput]
argument-hint: "[commits:N] [viewport:desktop|mobile|both]"
---

# UI Flow Testing

Analyze recent code changes and test affected UI flows using Playwright MCP browser automation.

**Arguments:** $ARGUMENTS
- `commits:N` - Number of commits to analyze (default: 5)
- `viewport:desktop|mobile|both` - Which viewport(s) to test (default: both)

Parse arguments like: "6" = 6 commits, "viewport:mobile" = mobile only

## Step 0: Start Dev Servers

First, detect and start the development servers.

### Detect Project Structure

```bash
# Check for common project structures
ls -la package.json frontend/ client/ apps/ 2>/dev/null
```

### Kill Existing Processes

```bash
# Kill common dev server ports
lsof -ti:3000 | xargs kill -9 2>/dev/null || true
lsof -ti:3001 | xargs kill -9 2>/dev/null || true
lsof -ti:5173 | xargs kill -9 2>/dev/null || true
lsof -ti:5174 | xargs kill -9 2>/dev/null || true
lsof -ti:8000 | xargs kill -9 2>/dev/null || true
lsof -ti:8080 | xargs kill -9 2>/dev/null || true
```

### Start Servers

Start servers in background using `run_in_background: true`:

**For monorepo with frontend/backend:**
```bash
# Backend
cd backend && yarn start
# Frontend
cd frontend && yarn dev
```

**For single package.json:**
```bash
yarn dev
# or
yarn start
```

Wait a few seconds, then verify servers are running:

```bash
curl -s http://localhost:3000 > /dev/null && echo "Port 3000 running" || echo "Port 3000 not running"
curl -s http://localhost:5173 > /dev/null && echo "Port 5173 running" || echo "Port 5173 not running"
```

Use `TaskOutput` to check server logs anytime during testing.

## Step 1: Analyze Changes

Get changed frontend files:

```bash
git diff HEAD~$COMMITS --name-only | grep -E '\.(tsx|ts|vue|svelte|jsx|js)$' | grep -v '\.test\.' | grep -v '\.spec\.'
```

**READ each changed file** to understand:
- What UI components/pages were modified
- What user interactions are involved (forms, buttons, navigation)
- What API calls are made
- What validation logic exists

For each changed page/component, identify:
1. **Entry points** - How does user get to this page?
2. **User actions** - What can user do?
3. **Expected outcomes** - What should happen?
4. **Error states** - What validation/error handling exists?

## Step 2: Create Test Plan

**Use TodoWrite to create a test checklist** that tracks each flow:

Example:
- Test Login Flow - Desktop - pending
- Test Login Flow - Mobile - pending
- Test Registration Form - Desktop - pending
- Test Registration Form - Mobile - pending

For each flow, document:
1. **Flow name**: e.g., "User Login"
2. **Steps**: Exact sequence of actions
3. **Test data**: What credentials/input to use
4. **Verification**: How to confirm it worked

## Step 3: Run Browser Tests

Use Playwright MCP tools to test each identified flow.

### Viewport Testing

**Desktop viewport** (default ~1280x720):
- No resize needed

**Mobile viewport** (390x844):
```
browser_resize: { width: 390, height: 844 }
```

**Testing order**:
1. Complete ALL flows in desktop viewport first
2. Resize to mobile and repeat ALL flows
3. Or if specific viewport requested, only test that one

### Visual Inspection

After each navigation, **take a screenshot** with `browser_take_screenshot` (fullPage: true).

**Check for:**

**Layout Issues:**
- Elements overlapping or cut off
- Inconsistent spacing
- Content not properly aligned

**Mobile-Specific:**
- Buttons too small to tap (min 44x44px)
- Horizontal scrolling
- Elements not stacking properly
- Text too small/large

**Responsive Design:**
- Desktop layouts on mobile (should stack)
- Proper image/icon scaling
- Form fields appropriate width

**Component Consistency:**
- Consistent button styles
- Consistent icon sizes
- Consistent spacing and borders

### Bug Discovery Protocol

**If you discover a bug during testing:**
1. STOP testing immediately
2. Fix the bug in the source code
3. Close the browser with `browser_close`
4. Restart testing from Step 0

### Handling Dialogs

Browser may show password manager dialogs or other popups.

**After login, press Escape** to dismiss:
```
browser_press_key: Escape
```

If dialog blocks interaction:
- Use `browser_handle_dialog` with `accept: true`
- Or take snapshot to find dismiss button

### Navigation Tips

- If dropdown menus are flaky, navigate directly via URL
- Remember auth pages redirect when logged in - logout first
- Always take snapshot after navigation to verify page loaded

### Common Test Patterns

**Login Flow:**
1. Navigate to login page
2. Fill credentials
3. Click submit
4. Verify redirect to dashboard
5. Press Escape to dismiss password dialog
6. Test logout

**Form Validation:**
1. Submit empty form - verify errors
2. Enter invalid data - verify validation
3. Enter valid data - verify success

**CRUD Operations:**
1. Create new item
2. Verify it appears in list
3. Edit the item
4. Verify changes saved
5. Delete the item
6. Verify removal

## Step 4: Test Credentials

Look for test credentials in the project:

```bash
# Common locations
grep -r "test.*password\|testuser\|test@" --include="*.ts" --include="*.js" --include="*.json" | head -20
```

Check files like:
- `test-data.ts`
- `seed.ts`
- `fixtures/`
- `.env.example`

**Document found credentials for testing.**

## Step 5: Database Verification (if applicable)

If the project has database access via MCP:

```sql
-- Verify data after actions
SELECT * FROM users ORDER BY id DESC LIMIT 5;

-- Check specific records
SELECT * FROM users WHERE email = 'test@example.com';
```

## Step 6: Cleanup

After testing:

1. **Reset changed data** if possible
2. **Close browser** with `browser_close`
3. **Optionally stop servers** or leave running

## Step 7: Report Results

```markdown
## Test Results

### Desktop Viewport (1280x720)

#### Passed
- [Flow name] - [what was tested]

#### Failed
- [Flow name] - [what failed, error message]

#### Visual Issues
- [Page/component] - [description]

### Mobile Viewport (390x844)

#### Passed
- [Flow name] - [what was tested]

#### Failed
- [Flow name] - [what failed]

#### Visual Issues
- [Page/component] - [description]

### Not Tested
- [Flow name] - [reason]

### Coverage Summary
- Viewports: Desktop [x] Mobile [x]
- Forms tested: [x]
- Navigation tested: [x]
- Error states tested: [x]
```

## Notes

- App must be running locally
- Playwright MCP must be configured
- Use `browser_console_messages` to check for JS errors
- Take screenshots liberally for documentation
- Fix bugs immediately when found, then restart testing
