---
description: Implement a feature or fix from a casual description. Plans, implements, then runs verify, code-review, and test-ui.
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep, Task, TodoWrite, AskUserQuestion, EnterPlanMode]
argument-hint: "<task description in casual human form>"
---

# Implement Task

**Input:** $ARGUMENTS

## Phase 1: Understand & Refine the Task

First, analyze the casual task description and create a refined version.

### Parse the Input

1. **Identify task type**: New feature, enhancement, or bug fix?
2. **Clarify scope**: What parts of the codebase will be affected?
3. **Extract requirements**: What specifically needs to be done?

### Explore the Codebase

Before refining, explore relevant parts of the codebase:

```bash
# Find relevant files
find . -type f -name "*.ts" -o -name "*.tsx" -o -name "*.py" | grep -i "[keyword]" | head -20
```

Read existing code to understand:
- Current patterns and conventions
- Related functionality
- Where new code should go

### Present Refined Understanding

```markdown
## Task Understanding

**Type:** [Feature / Enhancement / Bug Fix]
**Summary:** [1-2 sentence refined description]
**Affected areas:** [frontend, backend, shared, database]

**Key Requirements:**
- [requirement 1]
- [requirement 2]
- [requirement 3]

**Questions/Assumptions:**
- [any clarifications needed]
```

## Phase 2: Clarify Requirements

If the task description is ambiguous, use `AskUserQuestion` to clarify:

**Common clarifications:**
- Which user roles should have access?
- What specific fields/data are needed?
- Any specific validation rules?
- How should errors be handled/displayed?
- Any UI/UX preferences?
- Priority/severity for bug fixes?

**Do NOT proceed with implementation if key requirements are unclear.**

## Phase 3: Plan the Implementation

Use `EnterPlanMode` to design the implementation approach.

In plan mode:
1. Explore the codebase to understand existing patterns
2. Identify files that need to be created or modified
3. Design the implementation approach
4. Consider edge cases and error handling
5. Write a detailed plan for user approval

### Plan Should Include

```markdown
## Implementation Plan

### Files to Create
- `path/to/new/file.ts` - Description of purpose

### Files to Modify
- `path/to/existing/file.ts` - What changes needed

### Database Changes (if any)
- Migration: Add column X to table Y

### API Changes (if any)
- New endpoint: POST /api/resource
- Modified endpoint: GET /api/resource/:id

### Frontend Changes (if any)
- New page/component: ResourceForm
- Modified component: ResourceList

### Steps
1. [First step]
2. [Second step]
3. [etc.]
```

## Phase 4: Implement

After plan is approved, implement the changes.

### Implementation Guidelines

**General:**
- Follow existing code patterns and conventions
- Keep changes minimal and focused
- Don't add unnecessary features or refactoring
- Handle errors appropriately
- Add types/type hints

**Frontend:**
- Reuse existing components
- Follow existing styling patterns
- Update translations if i18n is used
- Add proper loading/error states

**Backend:**
- Follow existing service/controller patterns
- Add proper validation
- Handle errors with appropriate status codes
- Add logging where helpful

**Database:**
- Create migrations for schema changes
- Consider data migration if needed

### Track Progress

Use `TodoWrite` to track implementation progress:

```
- Create database migration - in_progress
- Add API endpoint - pending
- Create frontend form - pending
- Add form validation - pending
- Write tests - pending
```

## Phase 5: Quality Assurance

After implementation, run quality checks in sequence.

### Run /verify

Execute the verify command to run build, lint, and tests:

```
/verify
```

Fix any errors until all checks pass.

### Run /code-review (for significant changes)

For changes that:
- Touch 5+ files
- Add new features with multiple components
- Modify authentication/authorization
- Handle sensitive data

Run code review:

```
/code-review
```

If issues found, fix them and re-run verify.

### Run /test-ui (if UI changed)

For changes that affect the user interface:

```
/test-ui
```

If visual issues or bugs found, fix them and re-run all checks.

## Phase 6: Summary

After all quality checks pass, provide a summary:

```markdown
## Implementation Complete

### Changes Made
- [file1.ts] - Added new component
- [file2.ts] - Modified API endpoint
- [etc.]

### New Features/Fixes
- [Description of what was implemented]

### Quality Checks
- Build: PASS
- Lint: PASS
- Tests: PASS (X tests)
- Code Review: PASS (no issues)
- UI Tests: PASS (desktop + mobile)

### Next Steps (if any)
- [Any follow-up tasks]
- [Documentation to update]
```

## Error Recovery

If any phase fails:

1. **Clarification fails:** Ask more specific questions
2. **Plan rejected:** Revise plan based on feedback
3. **Build fails:** Fix syntax/type errors
4. **Tests fail:** Fix failing tests or implementation
5. **Code review issues:** Fix issues and re-run verify
6. **UI test issues:** Fix visual/functional bugs

**Always re-run the full quality pipeline after fixes.**

## Task Types Reference

### New Feature
1. Full planning phase
2. Create new files
3. Full quality pipeline

### Enhancement
1. Lighter planning
2. Modify existing files
3. Full quality pipeline

### Bug Fix
1. Identify root cause first
2. Minimal changes to fix
3. Add regression test
4. Full quality pipeline

### Refactoring
1. Plan the refactoring scope
2. Make changes incrementally
3. Run tests after each change
4. Full quality pipeline
