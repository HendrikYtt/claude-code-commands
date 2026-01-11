---
description: Run build, lint, and tests across the project. Auto-detects package manager, frameworks, and tools.
allowed-tools: [Bash, Read, Glob, Grep, Edit]
---

# Verify Command

Run verification checks (build, lint, test) with auto-fix, iterating until all checks pass.

## Step 1: Detect Project Stack

First, detect the project configuration by checking for these files:

### Package Manager Detection
```bash
# Check which lockfile exists
ls -la package-lock.json yarn.lock pnpm-lock.yaml bun.lockb 2>/dev/null
```

**Detection rules:**
- `yarn.lock` -> Use `yarn`
- `pnpm-lock.yaml` -> Use `pnpm`
- `package-lock.json` -> Use `npm`
- `bun.lockb` -> Use `bun`

### Project Structure Detection
```bash
# Check for common project files
ls -la package.json pyproject.toml requirements.txt manage.py Cargo.toml go.mod 2>/dev/null

# Check for monorepo structure
ls -la apps/ packages/ frontend/ backend/ api/ client/ server/ 2>/dev/null
```

### Framework Detection

**Read package.json** (if exists) to detect:
- `react` -> React frontend
- `vue` -> Vue frontend
- `svelte` -> Svelte frontend
- `next` -> Next.js
- `express` -> Express backend
- `nestjs` -> NestJS backend
- `jest` -> Jest test runner
- `vitest` -> Vitest test runner
- `eslint` -> ESLint linter
- `biome` -> Biome linter

**Read pyproject.toml or requirements.txt** (if exists) to detect:
- `django` -> Django backend
- `fastapi` -> FastAPI backend
- `flask` -> Flask backend
- `pytest` -> Pytest test runner
- `ruff` -> Ruff linter

### Report Detection

Tell the user what was detected:

```
## Detected Stack

**Package Manager:** [yarn/pnpm/npm/bun]
**Project Type:** [monorepo/single]

**Frontend:**
- Framework: [React/Vue/Svelte/Next.js/None]
- Location: [frontend/ or ./]
- Build: [yarn build / npm run build / etc.]
- Lint: [yarn lint / eslint / biome]
- Test: [yarn test / jest / vitest]

**Backend:**
- Framework: [Django/FastAPI/Express/NestJS/None]
- Location: [backend/ or api/ or ./]
- Build: [yarn build / tsc / None]
- Lint: [ruff / eslint / pylint]
- Test: [pytest / jest / mocha]
```

## Step 2: Run Verification

Execute checks based on detected stack. Run in this order:

### Frontend Verification (if detected)

1. **Build** (skip if no build script):
   ```bash
   cd [frontend-dir] && [package-manager] build
   ```

2. **Lint with auto-fix**:
   ```bash
   # ESLint
   cd [frontend-dir] && [package-manager] lint --fix
   # OR Biome
   cd [frontend-dir] && [package-manager] biome check --write
   ```

3. **Tests** (if test script exists):
   ```bash
   cd [frontend-dir] && [package-manager] test
   ```

### Backend Verification (if detected)

**For Node.js backends:**
```bash
cd [backend-dir] && [package-manager] build
cd [backend-dir] && [package-manager] lint --fix
cd [backend-dir] && [package-manager] test
```

**For Python backends:**
```bash
cd [backend-dir] && ruff check --fix .
cd [backend-dir] && ruff format .
cd [backend-dir] && pytest
```

**For Go backends:**
```bash
cd [backend-dir] && go build ./...
cd [backend-dir] && golangci-lint run --fix
cd [backend-dir] && go test ./...
```

## Step 3: Fix Errors Iteratively

If any command fails:

1. **Read and understand the error**
2. **Fix the error** in the source file
3. **Re-run the failed command**
4. **Repeat** until no errors remain

### Common Error Fixes

**TypeScript/ESLint errors:**
- Unused variables: Remove or prefix with `_`
- Missing types: Add proper type annotations
- Import errors: Fix import paths

**Ruff/Python errors:**
- Unused imports: Remove them
- Line too long: Break into multiple lines
- Missing docstrings: Add them (if required by config)

**Test failures:**
- Read the test file to understand expected behavior
- Fix the implementation or update the test

## Step 4: Final Verification

Run all checks one more time to confirm everything passes:

```bash
# Final check for frontend
cd [frontend-dir] && [package-manager] build && [package-manager] lint && [package-manager] test

# Final check for backend
cd [backend-dir] && [backend-lint] && [backend-test]
```

## Step 5: Report Results

```
## Verification Complete

### Frontend
- Build: [PASS/FAIL]
- Lint: [PASS/FAIL] (X issues fixed)
- Tests: [PASS/FAIL] (X passed, Y failed)

### Backend
- Build: [PASS/FAIL]
- Lint: [PASS/FAIL] (X issues fixed)
- Tests: [PASS/FAIL] (X passed, Y failed)

### Summary
[All checks passed / X issues remaining]
```

## Error Handling Strategy

- Fix one error at a time
- For TypeScript errors: check types, imports, and function signatures
- For ESLint errors: follow the suggested fix or rule documentation
- For Python/Ruff errors: follow PEP8 style, check imports
- For test failures: understand the expected vs actual behavior
- If stuck on an error, explain it to the user and ask for guidance

## Success Criteria

All of the following must pass with no errors:
- Build exits with code 0
- Lint shows no remaining errors
- Tests all pass
