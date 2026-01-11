# Claude Code Commands

Reusable, auto-detecting Claude Code commands and skills that work across different project stacks.

---

## For Humans

### Features

- **Auto-detection**: Commands automatically detect your package manager, framework, linters, and test runners
- **Stack-agnostic**: Works with React, Vue, Svelte, Next.js, Django, FastAPI, Express, NestJS, and more
- **Customizable**: Copy to your project and fine-tune for your specific needs

### Installation

#### Copy specific commands

1. Create `.claude/commands/` in your project root
2. Copy the commands you need from this repo's `commands/` folder
3. Customize paths and patterns for your project

#### Copy everything

```bash
# From your project root
mkdir -p .claude/commands .claude/skills
cp -r path/to/claude-code-commands/commands/* .claude/commands/
cp -r path/to/claude-code-commands/skills/* .claude/skills/
```

### Commands

| Command                      | Description                                                                       |
|------------------------------|-----------------------------------------------------------------------------------|
| `/verify`                    | Run build, lint, and tests with auto-fix                                          |
| `/code-review`               | Review uncommitted code for clean code principles                                 |
| `/test-ui`                   | Systematic UI testing with Playwright MCP                                         |
| `/run-apps`                  | Start all apps in background                                                      |
| `/implement-task`            | Turn casual description into implementation                                       |
| `/setup-data-sources`        | Configure Playwright + Postgres MCP servers                                       |
| `/init-fullstack-ts-project` | Initialize a new fullstack TypeScript project (Bun + Express + React + Terraform) |

### Skills

| Skill | Description |
|-------|-------------|
| `frontend-design` | Create React/Vue pages and components following project patterns |

### Customization

#### Adding Project-Specific Paths

Edit the command files to add your project's specific paths. Look for sections marked with comments like:

```markdown
## Project-Specific Configuration
<!-- Add your project paths here -->
```

#### Adding Test Credentials

For `/test-ui`, add your test user credentials:

```markdown
## Test Credentials
| Role | Email | Password |
|------|-------|----------|
| Admin | admin@example.com | testpass123 |
| User | user@example.com | testpass123 |
```

#### Framework-Specific Overrides

The commands auto-detect your stack, but you can override detection by adding explicit configuration at the top of the command:

```markdown
## Override Detection
- Package manager: pnpm
- Frontend: React + Vite
- Backend: FastAPI
- Test runner: pytest
```

### Requirements

- Claude Code CLI
- For `/test-ui`: Playwright MCP server configured
- For `/setup-data-sources`: Target database accessible

### Contributing

1. Fork the repo
2. Add or improve commands
3. Test with different project types
4. Submit PR with examples

### License

MIT

---

## For LLMs

> This section is for Claude Code and other LLMs reading this repository to understand its structure and conventions.

### Repository Structure

```
commands/          # Command instructions — read these when executing slash commands
  verify.md        # /verify command
  code-review.md   # /code-review command
  init-fullstack-ts-project.md  # /init-fullstack-ts-project command
  ...
skills/            # Skill instructions — read these when skill is invoked
  frontend-design/
    SKILL.md       # Frontend design skill instructions
templates/         # Project templates used by init commands
  fullstack-ts/    # Fullstack TypeScript template (Bun + Express + React + Terraform)
```

### How Commands Work

Each command file in `commands/` contains step-by-step instructions for you to follow. The general pattern is:

1. **Detect** the project stack (package manager, frameworks, tools)
2. **Execute** the task using detected configuration
3. **Iterate** until success (fix errors, re-run)
4. **Report** results to the user

### Auto-Detection Reference

When detecting project configuration, use these rules:

#### Package Managers
| File                | Use  |
|---------------------|------|
| `yarn.lock`         | yarn |
| `pnpm-lock.yaml`    | pnpm |
| `package-lock.json` | npm  |
| `bun.lockb`         | bun  |

#### Frontend Frameworks
| Indicator                               | Framework    |
|-----------------------------------------|--------------|
| `next.config.*`                         | Next.js      |
| `vite.config.*` + react in package.json | React + Vite |
| `vue.config.*` or vite + vue            | Vue          |
| `svelte.config.*`                       | SvelteKit    |
| `angular.json`                          | Angular      |

#### Backend Frameworks
| Indicator                           | Framework |
|-------------------------------------|-----------|
| `manage.py`                         | Django    |
| `main.py` + fastapi in dependencies | FastAPI   |
| `app.py` or `wsgi.py` + flask       | Flask     |
| `package.json` + express            | Express   |
| `nest-cli.json`                     | NestJS    |

#### Linters
| Indicator                               | Linter |
|-----------------------------------------|--------|
| `.eslintrc.*` or eslint in package.json | ESLint |
| `biome.json`                            | Biome  |
| `ruff.toml` or ruff in pyproject.toml   | Ruff   |
| `.pylintrc`                             | Pylint |

#### Test Runners
| Indicator                                | Runner |
|------------------------------------------|--------|
| jest in package.json                     | Jest   |
| vitest in package.json                   | Vitest |
| `pytest.ini` or pytest in pyproject.toml | Pytest |
| mocha in package.json                    | Mocha  |

### General Principles

1. **Always detect before executing** — Never assume the stack; check lockfiles and config files first
2. **Follow existing patterns** — Read existing code in the project to match conventions
3. **Iterate on failures** — When a command fails, fix the issue and retry
4. **Report clearly** — Summarize what was detected and what actions were taken
5. **Ask when uncertain** — If detection is ambiguous or a decision is needed, ask the user
