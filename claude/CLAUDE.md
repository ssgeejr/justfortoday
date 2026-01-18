# Global Claude Code Configuration

This file applies to ALL projects on this system. These rules are non-negotiable and override any project-specific instructions that conflict with them.

## Critical Instructions

### 🎯 ASK QUESTIONS - NEVER ASSUME
**MOST IMPORTANT RULE**: When you are unsure about:
- Implementation approach
- Architecture decisions
- File placement or directory structure
- Technology choices
- Security implications
- Breaking changes
- Interpretation of requirements
- Where data should be stored vs where code should live

**YOU MUST ASK QUESTIONS BEFORE PROCEEDING.**

Never assume. Never guess. Better to ask 10 clarifying questions than to build the wrong thing once.

## Security Rules - ABSOLUTE

### 🔒 NEVER Access These Files
Claude Code must NEVER read, display, or reference the contents of:
- `.env` files (environment variables)
- `.env.*` files (any environment variant)
- `secrets.json`, `secrets.yaml`, or any file with "secret" in the name
- `config/credentials.*`
- `.aws/credentials`
- `.ssh/*` (SSH private keys)
- `*.pem`, `*.key`, `*.p12` (certificate/key files)
- `.npmrc`, `.pypirc` (package manager auth)
- Database connection strings with embedded passwords
- API keys or tokens in any form

### 🛡️ Security Best Practices
- **Always use environment variables** for credentials (via .env files that are gitignored)
- **Never hardcode** passwords, API keys, or tokens in source code
- **Use PreparedStatements** for SQL (prevent injection attacks)
- **Validate all user input** server-side (never trust client)
- **Log security-relevant actions** but never log credentials or sensitive data
- **Use HTTPS** for production deployments
- **Sanitize error messages** shown to users (no stack traces in production)

## Directory Structure & Data Separation

### CRITICAL: Separate Code from Runtime Data

**Code/Source Repository Location:**
```
/opt/apps/<project-name>/
├── src/                    # Source code
├── docs/                   # Documentation
├── tests/                  # Test files
├── config/                 # Configuration templates (no secrets)
├── scripts/                # Build/deployment scripts
├── README.md
├── CLAUDE.md
├── .gitignore
└── (build configs: pom.xml, package.json, etc.)
```

**Runtime Data Location:**
```
/opt/dta/<project-name>/
├── logs/                   # All application logs
│   ├── app/               # Application logs
│   ├── server/            # Server logs (tomcat, nginx, etc.)
│   └── database/          # Database logs
├── data/                   # Persistent data storage
│   ├── database/          # Database files (MySQL, PostgreSQL, etc.)
│   ├── uploads/           # User uploaded files
│   └── cache/             # Cache files
├── backups/               # Database and file backups
├── tmp/                   # Temporary runtime files
└── secrets/               # Runtime secrets (NOT in git)
    └── .env               # Environment variables
```

### WHY This Separation Matters

**NEVER put runtime data in the git repository:**
- ❌ Database files (.sqlite, mysql data directories)
- ❌ Log files (*.log)
- ❌ User uploads (images, documents)
- ❌ Cache files
- ❌ Temporary files
- ❌ Any file that changes during runtime

**Reasons:**
1. **Repository bloat**: Runtime data makes repos huge and slow
2. **Security**: Logs/data might contain sensitive information
3. **Conflicts**: Multiple developers/environments create merge conflicts
4. **Cleanliness**: Source code repos should only contain source code
5. **Portability**: Code should work anywhere without carrying data

### Docker Volume Mounts

When using Docker, ALWAYS mount runtime data outside the repository:

```yaml
# ✅ CORRECT - Data persists outside repository
services:
  mysql:
    volumes:
      - /opt/dta/project-name/data/mysql:/var/lib/mysql
      - /opt/dta/project-name/logs/mysql:/var/log/mysql

  app:
    volumes:
      - /opt/dta/project-name/logs/app:/var/log/app
      - /opt/dta/project-name/uploads:/app/uploads

# ❌ WRONG - Data inside repository
services:
  mysql:
    volumes:
      - ./mysql-data:/var/lib/mysql  # DON'T DO THIS
      - ./logs:/var/log              # DON'T DO THIS
```

## Git & Version Control

### Never Commit These
- `.env` files and any secrets
- `node_modules/`, `target/`, `build/`, `dist/`, `__pycache__/` (build artifacts)
- IDE-specific folders (`.idea/`, `.vscode/`, `.vs/`)
- OS files (`.DS_Store`, `Thumbs.db`, `desktop.ini`)
- Temporary files (`*.tmp`, `*.temp`, `*.swp`, `*~`)
- Log files (`*.log`, `logs/`)
- Database files (`*.sqlite`, `*.db`, `mysql-data/`, `postgres-data/`)
- Cache directories (`cache/`, `.cache/`)
- User uploads (`uploads/`, `user-content/`)
- Compiled binaries (unless intentional)
- Large files (>50MB) - use Git LFS if needed

### Always Create .gitignore

Every project MUST have a `.gitignore` that excludes:
```gitignore
# Environment variables and secrets
.env
.env.*
secrets/

# Build artifacts
target/
build/
dist/
*.class
*.jar
*.war

# Dependencies
node_modules/
vendor/

# IDE
.idea/
.vscode/
*.iml

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Temporary
*.tmp
*.temp
*.swp

# Runtime data (should be in /opt/dta/)
data/
uploads/
cache/
```

### Git Commit Standards
- **Commit messages**: Use conventional commits format
  - `feat:` for new features
  - `fix:` for bug fixes
  - `docs:` for documentation
  - `refactor:` for code refactoring
  - `test:` for test changes
  - `chore:` for maintenance tasks
- **Atomic commits**: Each commit should represent one logical change
- **Test before commit**: Run tests and ensure they pass
- **No WIP commits**: Commits should be complete and functional

### Example Good Commit Messages
```
feat: add badge calculation logic for sobriety milestones
fix: prevent SQL injection in user input validation
docs: update README with Docker deployment instructions
refactor: extract database connection logic into separate class
test: add unit tests for date calculation edge cases
```

## Code Quality Standards

### General Principles
- **DRY (Don't Repeat Yourself)**: Extract repeated logic into functions/methods
- **KISS (Keep It Simple, Stupid)**: Prefer simple solutions over complex ones
- **YAGNI (You Aren't Gonna Need It)**: Don't build features before they're needed
- **Separation of Concerns**: Keep different responsibilities in different modules
- **Single Responsibility**: Each function/class should do one thing well

### Naming Conventions
- **Variables/Functions**: camelCase (`getUserData`, `isValid`, `calculateTotal`)
- **Classes**: PascalCase (`UserController`, `DataModel`, `DatabaseConnection`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_RETRIES`, `API_TIMEOUT`, `DB_HOST`)
- **Files**: Match the language convention (Java: PascalCase, Python: snake_case)
- **Be descriptive**: `getUserById` not `get`, `isEmailValid` not `check`
- **Avoid abbreviations**: `customer` not `cust`, `message` not `msg` (unless very common)

### Error Handling
- **Always handle errors explicitly** - no silent failures
- **Log errors with context** - include relevant data for debugging (but not secrets!)
- **User-facing errors** should be friendly and actionable
- **Never expose stack traces** to end users in production
- **Use appropriate exception types** - create custom exceptions when needed
- **Clean up resources** - use try-with-resources or finally blocks

### Documentation
- **Public APIs**: Must have clear documentation (JavaDoc, docstrings, etc.)
- **Complex logic**: Requires inline comments explaining WHY, not WHAT
- **README.md**: Every project must have:
  - What the project does
  - How to set it up
  - How to run it
  - How to run tests
  - How to deploy it
- **Update docs** when changing functionality
- **Document assumptions and constraints**

## Testing Philosophy

### Test Coverage Requirements
- **Critical paths**: MUST have tests (authentication, payments, data loss scenarios)
- **Edge cases**: Test boundary conditions and error states
- **Regression**: When fixing bugs, add tests to prevent recurrence
- **Happy path**: Test the normal, expected flow
- **Sad path**: Test error conditions and validation failures

### Test Organization
- **Unit tests**: Test individual functions/methods in isolation
- **Integration tests**: Test components working together
- **E2E tests**: Test complete user workflows from UI to database
- **Keep tests fast**: Slow tests don't get run often
- **Independent tests**: Each test should be able to run alone
- **Clean up after tests**: Reset state, delete test data

### Test-Driven Development (When Appropriate)
1. Write a failing test that defines desired behavior
2. Write minimum code to make the test pass
3. Refactor while keeping tests green
4. Repeat

## Development Workflow

### Before Starting Work
1. Ensure you understand the requirements fully
2. **Ask clarifying questions** if anything is unclear
3. Check for existing solutions or patterns in the codebase
4. Plan your approach before writing code
5. Consider edge cases and error scenarios

### During Development
1. Write tests first (TDD) when appropriate
2. Commit frequently with clear messages
3. Run tests after each significant change
4. Keep functions/methods small and focused (< 50 lines ideally)
5. Review your own code before committing

### Before Completing Work
1. Run full test suite
2. Check for console errors/warnings
3. Verify code follows project standards
4. Update documentation if needed
5. Clean up debug code, console.logs, and commented code
6. Verify .gitignore is properly configured

## Language-Specific Standards

### Java
- Use Java 11+ features (prefer modern APIs)
- Follow Oracle's Java Code Conventions
- Use try-with-resources for AutoCloseable objects
- Prefer composition over inheritance
- Use meaningful exception types
- Always use PreparedStatement for SQL (never string concatenation)
- Package naming: com.company.project.module

### Python
- Follow PEP 8 style guide strictly
- Use type hints for function signatures
- Prefer list comprehensions for simple transformations
- Use context managers (`with` statement) for resources
- Virtual environments for ALL projects (venv or virtualenv)
- Use f-strings for formatting (not % or .format())

### JavaScript/TypeScript
- Use ES6+ features (const/let, arrow functions, destructuring)
- Prefer async/await over raw promises
- Use strict equality (`===`) not loose (`==`)
- TypeScript: Enable strict mode in tsconfig.json
- Avoid `any` type in TypeScript - use unknown or proper types
- Use semicolons consistently

### SQL
- Use uppercase for SQL keywords (`SELECT`, `FROM`, `WHERE`)
- Use meaningful table and column names (snake_case)
- Always use indexes on foreign keys
- Use transactions for multi-step operations
- **ALWAYS use prepared statements** - never concatenate user input
- Document complex queries with comments

## Docker & Containers

### Container Best Practices
- Use official base images when possible
- Keep images small (use Alpine variants when appropriate)
- Don't run containers as root (create non-root user)
- Use `.dockerignore` to exclude unnecessary files
- Tag images with versions, not just `latest`
- Multi-stage builds for smaller production images
- Pin base image versions (don't use `latest`)

### Docker Compose Best Practices
- Use health checks for dependent services
- Define resource limits (memory, CPU)
- Use environment variables for configuration (via .env file)
- **Volume mount runtime data to /opt/dta/** (NEVER in the repository)
- Network isolation between services
- Use depends_on with condition: service_healthy
- Separate development and production compose files

### Volume Mounting Rules
```yaml
# ✅ CORRECT - Explicit paths outside repository
volumes:
  - /opt/dta/project/logs/app:/var/log/app
  - /opt/dta/project/data/mysql:/var/lib/mysql

# ❌ WRONG - Relative paths inside repository
volumes:
  - ./logs:/var/log
  - ./data:/var/lib/mysql
```

## Performance Considerations

### General Guidelines
- **Measure before optimizing** - don't guess at bottlenecks
- **Cache expensive operations** when appropriate (but not prematurely)
- **Use pagination** for large datasets (never load entire tables)
- **Lazy load** when possible (images, modules, data)
- **Database indexes** on frequently queried fields (especially foreign keys)
- **Connection pooling** for databases
- **Avoid N+1 queries** - use joins or batch loading

### Red Flags
- N+1 query problems (query in a loop)
- Loading entire datasets into memory
- Synchronous operations blocking the main thread
- Missing database indexes on foreign keys or WHERE clause columns
- Unbounded loops or recursion
- No pagination on list endpoints

## When in Doubt

If you're unsure about:
- **Security implications**: Ask before proceeding, err on the side of caution
- **Architecture decisions**: Discuss the approach first, present options
- **Breaking changes**: Get confirmation before implementing
- **Unclear requirements**: Seek clarification immediately - never guess
- **Trade-offs**: Present options with pros/cons and ask for decision
- **File/directory placement**: Ask where things should go
- **Data vs code separation**: When in doubt, put runtime data in /opt/dta/

### Communication Style
- Be clear and concise
- Ask one question at a time when seeking clarification
- Explain your reasoning for technical decisions
- Flag risks and trade-offs proactively
- Admit when you don't know something
- Present options rather than making unilateral decisions
- Use examples to clarify ambiguous requirements

## Editor Constraints (Project-Specific)

Some projects may specify editor constraints (e.g., "use vi only"). Always follow project-specific CLAUDE.md for such requirements. The global file doesn't enforce specific editors.

## System Information

- **OS**: Ubuntu 24.04 on WSL2 (Windows 11 25H2)
- **User**: aiadmin
- **Host**: GRAVITYDRIVE
- **Primary Use**: Learning & open source development
- **Focus**: Building tools that give back to communities
- **Philosophy**: Code with purpose, build with care

## Environment Setup

### Standard Directory Structure
```bash
# Code repositories (tracked in git)
/opt/apps/
  └── project-name/

# Runtime data (NOT in git)
/opt/dta/
  └── project-name/
      ├── logs/
      ├── data/
      ├── backups/
      ├── tmp/
      └── secrets/
```

### Creating New Project Directories
```bash
# Create project structure
sudo mkdir -p /opt/apps/project-name
sudo mkdir -p /opt/dta/project-name/{logs,data,backups,tmp,secrets}
sudo chown -R aiadmin:aiadmin /opt/apps/project-name /opt/dta/project-name
```

## Tool Versions (Update as needed)

Current standard versions on this system:
- **Java**: OpenJDK 17.0.17
- **Maven**: Apache Maven 3.8.7
- **Git**: 2.43.0
- **Docker**: 29.1.3 (via Docker Desktop with WSL integration)
- **Python**: (add when used)
- **Node.js**: (add when used)

---

## Remember

These rules exist to:
1. **Prevent security vulnerabilities** (exposed secrets, SQL injection)
2. **Maintain code quality** (readable, maintainable, testable)
3. **Ensure proper architecture** (separation of concerns, data vs code)
4. **Enable collaboration** (clear standards, good documentation)
5. **Protect user data** (proper handling, logging, validation)

**When project-specific rules conflict with these global rules, the global rules win** - unless explicitly discussed and approved for good reason.

**Always prioritize security and user safety over convenience or speed.**

**When in doubt, ASK. Never assume.**

