---
description: Set up Playwright and Postgres MCP data sources for browser automation and database access.
allowed-tools: [Bash, Read, Write, Edit, Glob, AskUserQuestion]
---

# Setup Data Sources

Configure MCP (Model Context Protocol) servers for Playwright browser automation and Postgres database access.

## Step 1: Check Current Configuration

Check if MCP settings already exist:

```bash
# Check for existing claude settings
cat .claude/settings.json 2>/dev/null || echo "No .claude/settings.json found"

# Check for project-level settings
cat claude.json 2>/dev/null || echo "No claude.json found"
```

## Step 2: Detect Database Configuration

Look for database connection info in the project:

```bash
# Check common locations
cat .env 2>/dev/null | grep -i "database\|postgres\|db_" || true
cat .env.local 2>/dev/null | grep -i "database\|postgres\|db_" || true
cat .env.example 2>/dev/null | grep -i "database\|postgres\|db_" || true

# Check docker-compose
cat docker-compose.yml 2>/dev/null | grep -A 10 "postgres" || true
cat docker-compose.yaml 2>/dev/null | grep -A 10 "postgres" || true

# Check for Django settings
grep -r "DATABASES" --include="settings.py" 2>/dev/null | head -5 || true

# Check for SQLAlchemy/FastAPI
grep -r "DATABASE_URL\|postgresql" --include="*.py" 2>/dev/null | head -5 || true
```

## Step 3: Gather Configuration

Ask user for any missing information:

### Postgres Configuration

If database config not found, ask user:

```
What is your Postgres connection info?

Options:
1. Local Docker (localhost:5432)
2. Local native install
3. Remote database
4. I'll provide connection string
```

**Required info:**
- Host (default: localhost)
- Port (default: 5432)
- Database name
- Username
- Password

### Playwright Configuration

Playwright MCP typically needs:
- Browser preference (chromium, firefox, webkit)
- Headless mode (true/false for debugging)

## Step 4: Create MCP Configuration

Create or update `.claude/settings.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-playwright"],
      "env": {
        "PLAYWRIGHT_BROWSER": "chromium",
        "PLAYWRIGHT_HEADLESS": "true"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "POSTGRES_HOST": "localhost",
        "POSTGRES_PORT": "5432",
        "POSTGRES_DB": "your_database",
        "POSTGRES_USER": "your_user",
        "POSTGRES_PASSWORD": "your_password"
      }
    }
  }
}
```

**Alternative: Connection string format for Postgres:**

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:password@localhost:5432/database"
      }
    }
  }
}
```

## Step 5: Create Settings File

Write the configuration:

```bash
mkdir -p .claude
```

Then write the settings.json file with detected/provided values.

## Step 6: Verify Installation

Check that MCP servers can be started:

```bash
# Test Playwright MCP
npx -y @anthropic-ai/mcp-server-playwright --help 2>/dev/null && echo "Playwright MCP: OK" || echo "Playwright MCP: Install needed"

# Test Postgres MCP
npx -y @anthropic-ai/mcp-server-postgres --help 2>/dev/null && echo "Postgres MCP: OK" || echo "Postgres MCP: Install needed"
```

## Step 7: Add to .gitignore

Ensure settings with credentials aren't committed:

```bash
# Check if .gitignore exists and has .claude/settings.json
grep -q ".claude/settings.json" .gitignore 2>/dev/null || echo ".claude/settings.json" >> .gitignore
```

Or create `.claude/settings.local.json` pattern:
```bash
echo ".claude/settings.local.json" >> .gitignore
```

## Step 8: Report Configuration

```markdown
## MCP Data Sources Configured

### Playwright
- Status: Configured
- Browser: chromium
- Headless: true
- Usage: `browser_navigate`, `browser_click`, `browser_screenshot`, etc.

### Postgres
- Status: Configured
- Host: localhost
- Port: 5432
- Database: [database_name]
- Usage: `postgres_query`, `postgres_execute`, etc.

### Files Created/Modified
- `.claude/settings.json` - MCP server configuration
- `.gitignore` - Added settings.json to prevent credential leak

### Next Steps
1. Restart Claude Code to load new MCP servers
2. Test with: "Take a screenshot of google.com"
3. Test with: "Show me the first 5 rows from users table"

### Security Notes
- Database credentials are stored in `.claude/settings.json`
- This file is gitignored to prevent credential leaks
- For team use, consider using environment variables
```

## Configuration Templates

### Development (Local Docker)

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-playwright"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://postgres:postgres@localhost:5432/app_dev"
      }
    }
  }
}
```

### With Environment Variables

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-postgres"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

### Headless Disabled (for debugging)

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-playwright"],
      "env": {
        "PLAYWRIGHT_HEADLESS": "false"
      }
    }
  }
}
```

## Troubleshooting

**MCP server not connecting:**
- Restart Claude Code after adding configuration
- Check that npx can access the packages
- Verify network access for remote databases

**Postgres connection failed:**
- Verify database is running: `pg_isready -h localhost -p 5432`
- Check credentials are correct
- Ensure database allows connections from localhost

**Playwright browser issues:**
- Install browsers: `npx playwright install chromium`
- Check system dependencies for headless browsers
