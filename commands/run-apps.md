---
description: Start all project applications and containers. Auto-detects project structure, starts Docker containers, runs apps in background, and verifies health.
allowed-tools: [Bash, Read, Glob, Grep, TaskOutput]
---

# Run Apps Command

Start all project applications and required containers for development.

## Step 1: Detect Project Structure

### Check Package Manager
```bash
ls -la yarn.lock pnpm-lock.yaml package-lock.json bun.lockb 2>/dev/null
```

**Detection rules:**
- `yarn.lock` -> Use `yarn`
- `pnpm-lock.yaml` -> Use `pnpm`
- `package-lock.json` -> Use `npm`
- `bun.lockb` -> Use `bun`

### Check for Monorepo
```bash
ls -la apps/ packages/ frontend/ backend/ api/ client/ server/ services/ 2>/dev/null
```

### Read package.json Scripts
```bash
cat package.json | grep -A 30 '"scripts"'
```

Look for:
- `start`, `dev`, `serve` scripts
- `start:api`, `start:frontend`, `start:backend` scripts
- Workspace scripts

## Step 2: Detect Required Containers

Check for Docker-related files and configuration:

```bash
# Check for docker-compose
ls -la docker-compose.yml docker-compose.yaml compose.yml compose.yaml 2>/dev/null

# Check for container references in config files
grep -r "POSTGRES\|REDIS\|MONGO\|MYSQL" .env* package.json 2>/dev/null | head -20
```

Common containers to detect:
- **PostgreSQL**: `POSTGRES_` env vars, `pg` in dependencies
- **Redis**: `REDIS_` env vars, `redis` in dependencies
- **MongoDB**: `MONGO_` env vars, `mongodb` in dependencies
- **MySQL**: `MYSQL_` env vars, `mysql` in dependencies

## Step 3: Start Containers

Start required containers using Docker. Try `docker start` first (for existing containers), fall back to `docker run` (for new containers):

### PostgreSQL
```bash
docker start {project-name}-db 2>/dev/null || \
docker run -d --name {project-name}-db \
  -e POSTGRES_USER=dev \
  -e POSTGRES_PASSWORD=dev \
  -e POSTGRES_DB={project-name} \
  -p 5432:5432 \
  postgres:17
```

### Redis (if needed)
```bash
docker start {project-name}-redis 2>/dev/null || \
docker run -d --name {project-name}-redis \
  -p 6379:6379 \
  redis:7-alpine
```

### Using docker-compose (if exists)
```bash
docker-compose up -d
```

## Step 4: Wait for Containers to be Healthy

```bash
# Wait for PostgreSQL
echo "Waiting for PostgreSQL..."
for i in {1..30}; do
  docker exec {project-name}-db pg_isready -U dev && break
  sleep 1
done

# Wait for Redis
echo "Waiting for Redis..."
for i in {1..10}; do
  docker exec {project-name}-redis redis-cli ping | grep -q PONG && break
  sleep 1
done
```

## Step 5: Kill Existing App Processes

```bash
# Kill by common ports
lsof -ti:3000 | xargs kill -9 2>/dev/null || true
lsof -ti:5173 | xargs kill -9 2>/dev/null || true
lsof -ti:5174 | xargs kill -9 2>/dev/null || true
lsof -ti:8000 | xargs kill -9 2>/dev/null || true

# Kill by process patterns
pkill -f "bun.*dev" 2>/dev/null || true
pkill -f "bun.*start" 2>/dev/null || true
pkill -f "node.*dev" 2>/dev/null || true
pkill -f "vite" 2>/dev/null || true
```

## Step 6: Start Applications

Start apps using `run_in_background: true` so they run in Claude's background.

### Pattern: Bun Monorepo
```bash
# Start all apps (uses workspaces)
bun run start
```

### Pattern: Separate Start Commands
```bash
# Start API
cd api && bun run dev

# Start Frontend
cd frontend && bun run dev
```

### Pattern: Yarn/NPM Workspaces
```bash
yarn dev
# or
npm run dev
```

## Step 7: Health Checks

Wait for apps to start, then verify everything is healthy:

### Container Health
```bash
# PostgreSQL
docker exec {project-name}-db pg_isready -U dev && echo "PostgreSQL: Healthy" || echo "PostgreSQL: Unhealthy"

# Redis (if used)
docker exec {project-name}-redis redis-cli ping && echo "Redis: Healthy" || echo "Redis: Unhealthy"
```

### App Health
```bash
# Wait for apps to initialize
sleep 5

# API health check
curl -sf http://localhost:3000/ping/liveness && echo "API: Healthy" || echo "API: Unhealthy"
curl -sf http://localhost:3000/ping/readiness && echo "API DB: Connected" || echo "API DB: Not connected"

# Frontend health check
curl -sf -o /dev/null http://localhost:5174 && echo "Frontend: Healthy" || echo "Frontend: Unhealthy"
```

If any health check fails, wait a few more seconds and retry. Report which services failed if they don't recover.

## Step 8: Report Status

```markdown
## Apps Running

### Containers
| Container | Port | Status |
|-----------|------|--------|
| PostgreSQL | 5432 | Healthy |
| Redis | 6379 | Healthy |

### Applications
| App | Port | Status | Logs |
|-----|------|--------|------|
| API | 3000 | Healthy | task_id_xxx |
| Frontend | 5174 | Healthy | task_id_yyy |

### URLs
- Frontend: http://localhost:5174
- API: http://localhost:3000
- API Health: http://localhost:3000/ping/liveness

### Commands

# View app logs
Use TaskOutput tool with the task IDs above

# Stop apps
pkill -f "bun run"

# Stop containers
docker stop {project-name}-db {project-name}-redis

# Restart everything
Run /run-apps again
```

## Troubleshooting

**Container won't start:**
```bash
# Check container logs
docker logs {project-name}-db --tail 50

# Check if port is in use by something else
lsof -i :5432
```

**App won't start:**
```bash
# Check logs via TaskOutput with the task ID
# Common issues:
# - Port in use: kill process on that port
# - Missing dependencies: run bun install
# - Database not ready: wait and retry
```

**Health check fails:**
```bash
# Check if process is running
ps aux | grep -E "bun|node" | grep -v grep

# Retry health check
curl -v http://localhost:3000/ping/liveness
```
