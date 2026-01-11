---
description: Review uncommitted and unpushed code for clean code principles, then fix issues if requested.
allowed-tools: [Bash, Read, Grep, Glob]
---

# Code Review Command

Review uncommitted and committed-but-not-pushed code for clean code principles, architecture patterns, and best practices.

**IMPORTANT**: This command runs in TWO phases:
1. **Review Phase** - Analyze code and generate review report (read-only)
2. **Fix Phase** - Create implementation plan to fix issues (after user says "fix")

## Phase 1: Code Review

### Step 1: Identify Changed Files

Get all changed files:

```bash
# Uncommitted changes (staged and unstaged)
git diff --name-only
git diff --cached --name-only

# Committed but not pushed (compare to default branch)
git log origin/main..HEAD --name-only --pretty=format:"" 2>/dev/null || \
git log origin/master..HEAD --name-only --pretty=format:"" 2>/dev/null
```

Get diff content for review:

```bash
# All uncommitted changes
git diff
git diff --cached

# Committed but not pushed
git diff origin/main..HEAD 2>/dev/null || git diff origin/master..HEAD 2>/dev/null
```

### Step 2: Detect Project Patterns

Read the codebase to understand project-specific patterns:

1. **Check for service/interface patterns** (DI, repository pattern)
2. **Check for existing abstractions** to ensure consistency
3. **Identify naming conventions** in use

### Step 3: Clean Code Review

Review all changed code for:

#### DRY (Don't Repeat Yourself)
- Look for duplicated code blocks (3+ lines repeated)
- Look for similar logic that could be extracted
- Check for copy-pasted code with minor variations

#### Single Responsibility
- Functions should do one thing
- Classes should have one reason to change
- Files shouldn't be too long (>500 lines is suspicious)

#### Naming
- Variables/functions should have descriptive names
- No single-letter variables (except loop counters)
- Consistent naming conventions (snake_case Python, camelCase TypeScript)
- Boolean variables should be `is_`, `has_`, `can_`, `should_`

#### Function Length & Complexity
- Functions over 50 lines should be flagged
- Deeply nested code (4+ levels) should be refactored
- Cyclomatic complexity concerns

#### Comments & Documentation
- Code should be self-documenting
- Comments should explain "why", not "what"
- No commented-out code
- No TODO comments without ticket/issue references

#### Error Handling
- No bare `except:` clauses (Python)
- No empty catch blocks (JS/TS)
- Errors should be logged with context
- No swallowing exceptions silently

#### Type Safety
- Python: Type hints on all function parameters and returns
- TypeScript: No `any` types without justification
- No type assertions without comments explaining why

### Step 4: Code Smells

Check for common code smells:

1. **Long Parameter Lists**: Functions with >4 parameters
2. **Magic Numbers**: Hardcoded values that should be constants
3. **God Classes**: Classes with too many methods/responsibilities
4. **Feature Envy**: Methods that use more from other classes than their own
5. **Dead Code**: Unused imports, variables, functions
6. **Inconsistent Abstraction**: Mixing high-level and low-level code

### Step 5: Security Review

Quick security check:

1. **No Secrets**: No hardcoded passwords, API keys, tokens
2. **SQL Injection**: Use parameterized queries, no string concatenation
3. **XSS**: Sanitize user input in frontend
4. **Logging**: Don't log sensitive data (passwords, tokens, PII)
5. **Input Validation**: Validate at system boundaries

### Step 6: Generate Review Report

Output the review report:

```markdown
# Code Review Report

**Date**: [current date]
**Files Changed**: X files
**Lines Changed**: +Y / -Z
**Issues Found**: N issues

## Critical Issues
- **[file:line]** Description of critical issue

## Warnings
- **[file:line]** Description of warning

## Suggestions
- **[file:line]** Suggestion for improvement

## Checklist

### Clean Code
- [ ] No DRY violations
- [ ] Functions are focused (single responsibility)
- [ ] Good naming conventions
- [ ] No overly long functions
- [ ] Proper error handling

### Security
- [ ] No hardcoded secrets
- [ ] No SQL injection risks
- [ ] Input properly validated
```

### Step 7: Prompt for Fix

After completing the review, output:

```
---
CODE REVIEW COMPLETE

I have completed the code review and documented all findings above.

To proceed with fixing the issues, respond with "fix" and I will:
1. Create a detailed implementation plan for each fix
2. Ask for your approval before making changes
3. Apply the fixes
---
```

Then STOP and wait for user response.

---

## Phase 2: Fix Implementation (After User Says "fix")

### Step 1: Prioritize Issues

Order issues by severity:
1. **Critical** - Security issues, broken functionality, missing required patterns
2. **Warnings** - DRY violations, type issues, architecture concerns
3. **Suggestions** - Naming improvements, minor refactoring

### Step 2: Create Implementation Plan

For each issue, document:
- **File**: Path to file
- **Line**: Line number(s)
- **Issue**: What's wrong
- **Fix**: Exact changes to make
- **Impact**: Other files affected

### Step 3: Present Plan

Structure the plan as:

```markdown
# Implementation Plan: Code Review Fixes

## Summary
- X critical issues to fix
- Y warnings to fix
- Z suggestions to implement

## Fixes

### 1. [Critical] Issue Title
**File**: `path/to/file.py:123`
**Issue**: Description
**Fix**:
- Step 1
- Step 2

**Code Change**:
\`\`\`python
# Before
old code

# After
new code
\`\`\`

### 2. [Warning] Issue Title
...
```

### Step 4: Apply Fixes

After user approves the plan:
1. Apply fixes one at a time
2. Verify each fix doesn't break anything
3. Report progress as you go

---

## Issue Severity Reference

- **Critical**: Must fix - security issues, broken functionality, violations of required patterns
- **Warning**: Should fix - DRY violations, long functions, missing types
- **Suggestion**: Nice to have - naming improvements, minor refactoring opportunities

## Example Patterns

### Good: Extract Duplicated Code
```python
# Before (DRY violation)
def create_user(data):
    if not data.get('email') or '@' not in data['email']:
        raise ValueError("Invalid email")

def update_user(data):
    if not data.get('email') or '@' not in data['email']:
        raise ValueError("Invalid email")

# After
def validate_email(email: str) -> None:
    if not email or '@' not in email:
        raise ValueError("Invalid email")

def create_user(data):
    validate_email(data.get('email', ''))

def update_user(data):
    validate_email(data.get('email', ''))
```

### Good: Proper Error Handling
```python
# Before
try:
    result = api.call()
except:
    pass

# After
try:
    result = api.call()
except ApiError as e:
    logger.error(f"API call failed: {e}", exc_info=True)
    raise
```
