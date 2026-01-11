---
description: Initialize a new Node.js + React project with Bun and Terraform AWS infrastructure. Creates monorepo structure with shared types.
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep, TodoWrite, AskUserQuestion]
argument-hint: "<project description>"
---

# Init Fullstack TypeScript Project Command

Initialize a new fullstack project by copying the template and customizing it.

## Step 1: Get Project Details

If `$ARGUMENTS` is empty, use `AskUserQuestion` to get:
1. Project name (lowercase, kebab-case)
2. Brief project description

If `$ARGUMENTS` is provided, extract project name and description from it.

## Step 2: Copy Template

Copy the template directory to the target location:

```bash
cp -r {this-repo}/templates/fullstack-ts {target-directory}/{project-name}
```

## Step 3: Replace Placeholders

Replace all occurrences of placeholders in the copied files:
- `{project-name}` → actual project name (e.g., `my-app`)
- `{project_db_name}` → project name with hyphens replaced by underscores (e.g., `my_app`) - used for PostgreSQL CDC identifiers
- `{project-description}` → actual project description

Use `find` and `sed` to replace these placeholders in all `.ts`, `.tsx`, `.json`, `.html`, `.yaml`, and `.tfvars` files.

## Step 4: Set Up Environment Files

Copy the example environment files:

```bash
cd {project-name}
cp api/.env.example api/.env
cp frontend/.env.example frontend/.env
```

Adjust credentials in `api/.env` if using an existing PostgreSQL container with different credentials.

## Step 5: Install Dependencies

```bash
cd {project-name}
bun install
bun run --cwd shared build
```

## Step 6: Initialize Git & Create GitHub Repository

```bash
cd {project-name}
git init
git add .
git commit -m "Initial project setup"
```

IMPORTANT: Do NOT include "Co-Authored-By: Claude" in the commit message. The user should be the sole author.

Create GitHub repository using gh CLI:

```bash
gh repo create {project-name} --private --source=. --push
```

## Step 7: Deploy AWS Infrastructure with Terraform

Use Terraform to create S3 bucket and CloudFront distribution:

```bash
cd infrastructure
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with project name and environment=prod
terraform init
terraform apply -auto-approve
```

Capture the outputs for GitHub secrets:

```bash
S3_BUCKET=$(terraform output -raw s3_bucket_name)
CLOUDFRONT_ID=$(terraform output -raw cloudfront_distribution_id)
```

## Step 8: Create GitHub CI Pipeline with Path Filters

Create `.github/workflows/deploy.yml` with path filtering using `dorny/paths-filter@v2`.

**IMPORTANT**: Use path filters so pipelines only run when relevant files change:
- API pipeline runs only when `api/**` or `shared/**` changes
- Frontend pipeline runs only when `frontend/**` or `shared/**` changes

Example structure:

```yaml
jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      api: ${{ steps.filter.outputs.api }}
      frontend: ${{ steps.filter.outputs.frontend }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            api:
              - 'api/**'
              - 'shared/**'
            frontend:
              - 'frontend/**'
              - 'shared/**'

  test-api:
    needs: setup
    if: needs.setup.outputs.api == 'true'
    # ... test steps

  deploy-api:
    needs: [setup, test-api]
    if: github.ref == 'refs/heads/main' && needs.setup.outputs.api == 'true'
    # ... deploy steps
```

Reference the cliniki project workflow for a complete example.

## Step 9: Set Up GitHub Secrets with gh CLI

Set all required secrets using the gh CLI:

```bash
# Docker Hub credentials (for API deployment)
gh secret set DOCKERHUB_USERNAME --body "<your-dockerhub-username>"
gh secret set DOCKERHUB_TOKEN --body "<docker-hub-token>"

# Kubernetes config (for API deployment)
gh secret set KUBECONFIG_FILE < ~/.kube/config

# AWS credentials (for frontend deployment)
gh secret set AWS_ACCESS_KEY_ID --body "<aws-access-key>"
gh secret set AWS_SECRET_ACCESS_KEY --body "<aws-secret-key>"

# Terraform outputs (for frontend deployment)
gh secret set S3_BUCKET --body "$S3_BUCKET"
gh secret set CLOUDFRONT_DISTRIBUTION_ID --body "$CLOUDFRONT_ID"
```

## Step 10: Add Ingress Configuration

Add ingress configuration for the new project subdomain in your infrastructure:

Edit your main infrastructure ingress configuration file (e.g., `ingress.yaml`) and add:

```yaml
---
# {project-name} ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {project-name}-ingress
  annotations:
    nginx.ingress.kubernetes.io/use-regex: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/proxy-body-size: 50m
    nginx.ingress.kubernetes.io/proxy-read-timeout: 3600s
    nginx.ingress.kubernetes.io/proxy-send-timeout: 3600s
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  ingressClassName: nginx
  rules:
    - host: {project-name}.{your-domain}
      http:
        paths:
          - path: /?(.*)
            pathType: Prefix
            backend:
              service:
                name: {project-name}-api
                port:
                  number: 3000
  tls:
    - hosts:
        - {project-name}.{your-domain}
      secretName: {project-name}-cert
```

## Step 11: Copy and Customize Claude Code Commands

Copy the Claude Code commands from the template to the new project and customize them:

```bash
cp -r {this-repo}/templates/fullstack-ts/.claude {project-name}/.claude
```

Then update each command file in `{project-name}/.claude/commands/` to be project-specific:

1. **verify.md** - Update with exact bun commands and project paths
2. **run-apps.md** - Update container names, ports, health check URLs
3. **code-review.md** - Update project-specific patterns and directory structure
4. **implement-task.md** - Update with project paths and conventions
5. **test-ui.md** - Update with project-specific UI flows
6. **setup-data-sources.md** - Update database credentials and connection strings

Also update the frontend-design skill in `.claude/skills/frontend-design/SKILL.md` with:
- Actual project stack (React, Tailwind, etc.)
- Actual directory structure
- Actual component patterns from the codebase

**IMPORTANT**: Remove the `init-fullstack-ts-project.md` command from the new project - it's only needed in the template repo.

## Step 12: Start Local Development Environment

### PostgreSQL Container

First, check if a PostgreSQL container is already running on port 5432:

```bash
docker ps --format "{{.Names}} {{.Ports}}" | grep 5432
```

- **If a container is running on 5432**: Use the existing container. Note which database to use (may need to create a new database for this project).
- **If no container on 5432**: Start a new PostgreSQL container:

```bash
docker run -d --name {project-name}-db \
  -e POSTGRES_USER=dev \
  -e POSTGRES_PASSWORD=dev \
  -e POSTGRES_DB={project_db_name} \
  -p 5432:5432 \
  postgres:17
```

### Start Apps

Use `/run-apps` command to:
- Start API and Frontend apps in background
- Run health checks on all services

Migrations run automatically on API startup.

## Step 13: Verify Setup

Use `/verify` command to ensure build, lint, and tests all pass.

## Step 14: Report Results

Report what was created:

```
## Project Initialized Successfully

**{project-name}** is ready:

### Stack
- Runtime: Bun
- API: Node.js + Express + TypeScript + Typia
- Frontend: React + Vite + Tailwind CSS
- Shared: TypeScript types package (ESM + CommonJS)
- Infrastructure: Terraform (S3 + CloudFront)
- Kubernetes: API deployment manifest
- CI/CD: GitHub Actions with path filters

### URLs
- Frontend: http://localhost:5174
- API: http://localhost:3000
- Production API: https://{project-name}.{your-domain}
- Production Frontend: (CloudFront URL from terraform output)

### GitHub Secrets Configured
- DOCKERHUB_USERNAME
- DOCKERHUB_TOKEN
- KUBECONFIG_FILE
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY
- S3_BUCKET
- CLOUDFRONT_DISTRIBUTION_ID

### Ingress Added
- {project-name}.{your-domain} → {project-name}-api:3000

### Claude Code Commands Customized
- /verify - Build, lint, test
- /run-apps - Start dev environment
- /code-review - Review code changes
- /implement-task - Implement features
- /test-ui - Browser testing
- /setup-data-sources - MCP configuration

### Next Steps
1. Apply the ingress: `kubectl apply -f ingress.yaml`
2. Push to main to trigger CI/CD pipeline
3. Deploy API: `kubectl apply -f k8s/api.yaml`
```

## IMPORTANT: Typia for Runtime Validation

This project uses Typia for runtime type validation. **This is critical for both request validation AND integration tests.**

### Request Validation (Middleware)

Always use `typia.createValidateEquals` (NOT `createValidate`) for incoming requests:

```typescript
import typia from 'typia';
import { validateBody } from './lib/validate';
import { CreateUserRequest } from '@{project-name}/shared';

router.post('/users',
  validateBody(typia.createValidateEquals<CreateUserRequest>()),
  async (req, res) => {
    // req.body is guaranteed to match CreateUserRequest exactly
    // Extra properties are REJECTED (prevents mass assignment attacks)
  }
);
```

### Integration Tests (CRITICAL!)

Use `typia.validateEquals` with `testType()` wrapper to validate API responses:

```typescript
import typia from 'typia';
import { testType, Serialize } from '../utils/test-utils';
import { User } from '@{project-name}/shared';

it('should return user data', async () => {
  const resp = await request(app)
    .get('/users/me')
    .set('Authorization', `Bearer ${token}`)
    .expect(200);

  // CRITICAL: Validate response matches User type EXACTLY
  testType(() => typia.validateEquals<Serialize<User>>(resp.body));

  expect(resp.body.email).to.equal('test@example.com');
});
```

Why `validateEquals` over `validate`:
- `validateEquals`: STRICT - rejects extra properties (secure!)
- `validate`: LOOSE - allows extra properties (dangerous!)
