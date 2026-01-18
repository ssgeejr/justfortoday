# JustForToday - Project Overview & Mission

## What Is This Project?

**JustForToday** is a sobriety tracking web application built to support individuals in recovery by:
- Tracking days of sobriety from a user-defined start date
- Calculating money saved based on average daily drinking costs
- Awarding milestone badges (1 day, 1 week, 1 month, 3 months, 6 months, 1 year, etc.)
- Providing encouragement and visual progress indicators

This application serves two purposes:
1. **Community Impact (49%)**: Give back to the sober community with a free, helpful tool
2. **Learning Opportunity (51%)**: Hands-on experience with autonomous AI development using Claude Code

## Why This Matters

Recovery is a journey measured one day at a time. This tool provides:
- **Concrete Progress**: Visual representation of days sober
- **Financial Motivation**: Real dollars saved adds tangible value
- **Milestone Celebration**: Badges recognize achievement at key intervals
- **Hope**: Seeing progress reinforces the commitment to sobriety

The name "JustForToday" reflects the recovery philosophy: focus on staying sober today, not forever. One day at a time.

## Technical Architecture

### Technology Stack (Specified Versions)

**Development Environment:**
- **OS**: Ubuntu 24.04 on WSL2 (Windows 11 25H2)
- **Java**: OpenJDK 17.0.17
- **Maven**: Apache Maven 3.8.7
- **Git**: 2.43.0
- **Docker**: 29.1.3 (via Docker Desktop with WSL integration)

**Application Stack:**
- **Backend**: Java 17 Servlets (NO frameworks - vanilla servlets only)
- **Frontend**: JSP pages with vanilla CSS (NO frameworks)
- **Server**: Apache Tomcat 10
- **Database**: MySQL 8.0
- **Testing**: Selenium WebDriver + Firefox + JUnit
- **Deployment**: Docker Compose multi-container setup

### System Architecture

```
┌─────────────────────────────────────────────┐
│         User Browser (Firefox)              │
│         http://localhost:8080               │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│      Tomcat 10 Container                    │
│      - Java Servlets (Controllers)          │
│      - JSP Views (Presentation)             │
│      - JDBC DAO Layer (Data Access)         │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│      MySQL 8.0 Container                    │
│      - sobriety_sessions table              │
│      - badges table                         │
│      - Persistent volume storage            │
└─────────────────────────────────────────────┘
```

## Directory Structure & Organization

### Critical Directory Rules

**Software & Source Code Location:**
```
/opt/apps/justfortoday/
├── src/
│   ├── main/
│   │   ├── java/com/justfortoday/
│   │   └── webapp/
│   └── test/
├── pom.xml
├── Dockerfile
├── docker-compose.yaml
├── README.md
├── CLAUDE.md (project-specific instructions)
└── .gitignore
```

**Data & Runtime Artifacts Location:**
```
/opt/dta/justfortoday/
├── mysql-data/          # MySQL persistent volume
├── logs/
│   ├── tomcat/          # Tomcat access & error logs
│   ├── mysql/           # MySQL logs
│   └── application/     # Application-specific logs
├── backups/             # Database backups
└── tmp/                 # Temporary runtime files
```

**CRITICAL SEPARATION RULE:**
- **ONLY** source code, documentation, and configuration files go in `/opt/apps/justfortoday/`
- **ALL** runtime data (logs, database files, temporary files) go in `/opt/dta/justfortoday/`
- **NEVER** mix the two - this keeps the source repository clean and separates concerns

### Git Repository

- **Remote**: To be created at https://github.com/ssgeejr/justfortoday
- **Local**: `/opt/apps/justfortoday/`
- **Branch Strategy**: main branch for stable code
- **Commit Philosophy**: Atomic, meaningful commits following conventional commits format

## Development Workflow Philosophy

### The "Ask First" Principle

**CRITICAL INSTRUCTION**: When you (Claude Code) are unsure about:
- Implementation approach
- Architecture decisions
- File placement
- Technology choices
- Security implications
- Breaking changes
- Interpretation of requirements

**YOU MUST ASK QUESTIONS BEFORE PROCEEDING.**

Never assume. Never guess. Better to ask 10 clarifying questions than to build the wrong thing once.

### Test-Driven Development Approach

1. **Understand** the user story/requirement fully
2. **Write tests** that define expected behavior
3. **Watch tests fail** (red)
4. **Implement** the minimum code to pass tests (green)
5. **Refactor** while keeping tests green
6. **Commit** with clear messages

### Iterative Build-Test-Fix Cycle

```
┌─────────────────────────────────────────────┐
│  1. Write/Modify Code                       │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  2. Build: mvn clean package                │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  3. Deploy: docker-compose up --build       │
└──────────────────┬──────────────────────────┘
                   ▼
┌─────────────────────────────────────────────┐
│  4. Test: mvn test (Selenium + JUnit)       │
└──────────────────┬──────────────────────────┘
                   ▼
           ┌───────┴────────┐
           │                │
        PASS ✅          FAIL ❌
           │                │
           │                ▼
           │    ┌─────────────────────────┐
           │    │  5. Read Error Logs     │
           │    │  6. Analyze Failure     │
           │    │  7. Fix Issue           │
           │    └──────────┬──────────────┘
           │               │
           │               ▼
           └───────────> REPEAT
```

## Editor Constraint

**CRITICAL RULE**: Use **vi/vim ONLY** for all file editing.

Why? This constraint:
- Forces familiarity with a universal Unix editor
- Ensures consistency in editing approach
- Simulates real production server environments
- Builds fundamental Linux skills

**No exceptions**: nano, emacs, VS Code remote, or any other editor is forbidden for this project.

## What We're Building: User Stories

### Core User Flow

**Story 1: First-Time User Entry**
```
AS A person in recovery
I WANT TO enter my sobriety start date and daily drinking cost
SO THAT I can see my progress and savings
```

**Acceptance Criteria:**
- User sees a clean, simple form on homepage
- Form has two inputs: date picker (start date) and number input (daily cost)
- Submit button clearly labeled "Calculate My Progress"
- Form validates inputs (date not in future, cost >= 0)
- Helpful error messages if validation fails

**Story 2: Results Display**
```
AS A user who submitted their information
I WANT TO see my sobriety statistics and earned badges
SO THAT I feel encouraged and motivated
```

**Acceptance Criteria:**
- Results page shows:
  - "You've been sober for X days" (large, prominent)
  - "You've saved $X.XX during your journey" (calculated and formatted)
  - Visual badge display with icons and names
  - Encouraging message
- All calculations are accurate
- UI is clean and uplifting (not clinical)

**Story 3: Data Persistence**
```
AS A returning user
I WANT MY data to be saved
SO THAT I can track my progress over time
```

**Acceptance Criteria:**
- Data persists in MySQL database
- User can retrieve their session
- Database survives container restarts (volumes configured)
- No data loss scenarios

### Badge System Specifications

Badges are awarded automatically based on days sober:

| Days | Badge Name | Emoji | Description |
|------|------------|-------|-------------|
| 1 | First Step | 🌟 | You took the first step |
| 7 | One Week Strong | 📅 | Seven days of strength |
| 30 | 30 Day Warrior | 🗓️ | One month milestone |
| 90 | 90 Days of Strength | 💪 | Three months strong |
| 180 | Half Year Hero | 🎯 | Six months of sobriety |
| 365 | One Year Champion | 🏆 | A full year sober |
| 730 | Two Year Legend | 🌈 | Two years of victory |

**Badge Logic:**
- User earns ALL badges where `days_sober >= days_required`
- Display badges in chronological order (oldest first)
- Highlight the most recent badge earned
- Show next badge goal and days remaining

## Technical Requirements & Constraints

### Must Use
- Java Servlets extending `HttpServlet`
- JSP for all views (no template engines)
- JDBC with `PreparedStatement` for all database operations
- Maven for dependency management and build lifecycle
- Docker Compose for orchestration
- Git for version control

### Must NOT Use
- Spring Boot, Spring MVC, or any Java frameworks
- Node.js, Express, or JavaScript backend frameworks
- Hibernate, JPA, or any ORM frameworks
- Thymeleaf, Freemarker, or template engines
- Bootstrap, Tailwind, or CSS frameworks
- React, Vue, Angular, or frontend frameworks

### Security Requirements

**SQL Injection Prevention:**
```java
// ❌ NEVER DO THIS
String sql = "SELECT * FROM users WHERE id = " + userId;

// ✅ ALWAYS DO THIS
String sql = "SELECT * FROM users WHERE id = ?";
PreparedStatement stmt = conn.prepareStatement(sql);
stmt.setInt(1, userId);
```

**Environment Variables for Credentials:**
```yaml
# docker-compose.yaml
services:
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}  # From .env file
```

**Input Validation:**
- Validate on server-side (never trust client)
- Check date ranges (start date not in future)
- Validate cost is positive number
- Sanitize all user inputs

### Database Schema

**Table: sobriety_sessions**
```sql
CREATE TABLE sobriety_sessions (
    id INT PRIMARY KEY AUTO_INCREMENT,
    start_date DATE NOT NULL,
    daily_cost DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_start_date (start_date)
);
```

**Table: badges**
```sql
CREATE TABLE badges (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description VARCHAR(255),
    days_required INT NOT NULL UNIQUE,
    emoji VARCHAR(10),
    INDEX idx_days_required (days_required)
);
```

**Seed Data:**
```sql
INSERT INTO badges (name, description, days_required, emoji) VALUES
('First Step', 'You took the first step', 1, '🌟'),
('One Week Strong', 'Seven days of strength', 7, '📅'),
('30 Day Warrior', 'One month milestone', 30, '🗓️'),
('90 Days of Strength', 'Three months strong', 90, '💪'),
('Half Year Hero', 'Six months of sobriety', 180, '🎯'),
('One Year Champion', 'A full year sober', 365, '🏆'),
('Two Year Legend', 'Two years of victory', 730, '🌈');
```

## Docker Configuration

### docker-compose.yaml Key Points

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: justfortoday
    volumes:
      - /opt/dta/justfortoday/mysql-data:/var/lib/mysql
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 10
    networks:
      - justfortoday-net

  tomcat:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_HOST: mysql
      DB_PORT: 3306
      DB_NAME: justfortoday
      DB_USER: root
      DB_PASSWORD: ${DB_PASSWORD}
    volumes:
      - /opt/dta/justfortoday/logs/tomcat:/usr/local/tomcat/logs
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - justfortoday-net

networks:
  justfortoday-net:
    driver: bridge

volumes:
  mysql-data:
```

**Critical Points:**
- MySQL starts first with healthcheck
- Tomcat waits for MySQL to be healthy
- Logs mount to `/opt/dta/justfortoday/logs/`
- Database data persists in `/opt/dta/justfortoday/mysql-data/`
- `.env` file provides `DB_PASSWORD` (NEVER commit this file)

### Dockerfile for Tomcat

```dockerfile
FROM tomcat:10-jdk17

# Remove default webapps
RUN rm -rf /usr/local/tomcat/webapps/*

# Copy WAR as ROOT.war (deploys to root context)
COPY target/ROOT.war /usr/local/tomcat/webapps/ROOT.war

# Expose port
EXPOSE 8080

# Tomcat starts automatically
CMD ["catalina.sh", "run"]
```

## Build & Deployment Process

### Complete Build Pipeline

```bash
# 1. Navigate to project root
cd /opt/apps/justfortoday

# 2. Build WAR file
mvn clean package

# Expected output:
# [INFO] BUILD SUCCESS
# [INFO] Total time: 15.234 s
# [INFO] Final Memory: 45M/178M

# 3. Build Docker images
docker-compose build

# 4. Start containers
docker-compose up -d

# 5. Watch logs (verify startup)
docker-compose logs -f

# 6. Test application
curl http://localhost:8080

# 7. Run tests
mvn test

# 8. If tests fail, check logs
docker-compose logs tomcat
docker-compose logs mysql
```

### Development Iteration Loop

```bash
# Make code changes with vi
vi src/main/java/com/justfortoday/servlets/SobrietyServlet.java

# Rebuild and redeploy
mvn clean package && docker-compose up -d --build

# Run tests
mvn test

# Check logs if issues
docker-compose logs -f tomcat
```

## Testing Strategy

### Unit Tests
- Test calculation logic (days sober, money saved)
- Test badge qualification logic
- Test input validation
- Mock database interactions

### Integration Tests
- Test DAO layer with test database
- Verify servlet request/response handling
- Test database connection pooling

### End-to-End Tests (Selenium)
```java
@Test
public void testCompleteSobrietyFlow() {
    // 1. Navigate to homepage
    driver.get("http://localhost:8080");
    
    // 2. Enter sobriety date (30 days ago)
    WebElement dateInput = driver.findElement(By.id("startDate"));
    dateInput.sendKeys("2025-12-19");
    
    // 3. Enter daily cost
    WebElement costInput = driver.findElement(By.id("dailyCost"));
    costInput.sendKeys("15.00");
    
    // 4. Submit form
    driver.findElement(By.id("submitBtn")).click();
    
    // 5. Verify results page
    WebElement daysDisplay = driver.findElement(By.id("daysSober"));
    assertEquals("30", daysDisplay.getText());
    
    WebElement savingsDisplay = driver.findElement(By.id("moneySaved"));
    assertEquals("$450.00", savingsDisplay.getText());
    
    // 6. Verify badges earned (should have 1 day and 7 day badges)
    List<WebElement> badges = driver.findElements(By.className("badge"));
    assertEquals(2, badges.size());
}
```

## Success Criteria (Definition of Done)

The project is complete when ALL of these are true:

### Functional Requirements
- ✅ User can navigate to `http://localhost:8080`
- ✅ Homepage displays input form for date and cost
- ✅ Form submission calculates and displays results
- ✅ Days sober calculation is accurate
- ✅ Money saved calculation is accurate (days × daily_cost)
- ✅ Correct badges display based on days sober
- ✅ Badge icons/emojis display properly
- ✅ Data persists in MySQL database
- ✅ Refreshing results page shows same data

### Technical Requirements
- ✅ `mvn clean package` completes with BUILD SUCCESS
- ✅ `docker-compose up -d` starts both containers without errors
- ✅ `mvn test` runs all tests with 0 failures
- ✅ No errors in Tomcat logs during normal operation
- ✅ No errors in MySQL logs during normal operation
- ✅ Application accessible at root context (no /app/ path)
- ✅ Database healthcheck passes before Tomcat starts

### Code Quality
- ✅ All Java code follows naming conventions
- ✅ PreparedStatements used for all SQL queries
- ✅ No hardcoded credentials in source code
- ✅ Error handling present and appropriate
- ✅ Code is well-commented where necessary
- ✅ No unused imports or variables

### Documentation
- ✅ README.md has clear setup instructions
- ✅ Database schema is documented
- ✅ Environment variables are documented
- ✅ Build process is documented
- ✅ Testing process is documented

### Repository Hygiene
- ✅ `.gitignore` excludes `.env`, `target/`, logs
- ✅ Git history shows atomic commits with clear messages
- ✅ No sensitive data in commit history
- ✅ Repository pushed to GitHub

## Development Phases (Recommended Approach)

### Phase 1: Foundation (Database & Models)
1. Create database initialization script (`init.sql`)
2. Define Java model classes (`SobrietySession`, `Badge`)
3. Implement DAO layer with JDBC
4. Write unit tests for models and DAO
5. Verify database connectivity

### Phase 2: Business Logic
1. Implement calculation service (days sober, money saved)
2. Implement badge qualification logic
3. Write comprehensive unit tests
4. Verify all calculations with edge cases

### Phase 3: Web Layer
1. Create servlets (form submission, results display)
2. Create JSP views (homepage, results page)
3. Add basic CSS styling
4. Wire up servlet mappings in web.xml

### Phase 4: Integration
1. Connect servlets to business logic
2. Connect business logic to DAO
3. End-to-end flow testing
4. Fix integration issues

### Phase 5: Docker Deployment
1. Create Dockerfile for Tomcat
2. Create docker-compose.yaml
3. Configure environment variables
4. Set up volume mounts for logs and data
5. Test full deployment

### Phase 6: Testing & Polish
1. Write Selenium tests
2. Verify all acceptance criteria
3. Fix any failing tests
4. Polish UI/UX
5. Final documentation pass

### Phase 7: Delivery
1. Complete README
2. Verify all git commits are clean
3. Push to GitHub
4. Final verification of all success criteria

## Communication Protocol with Claude Code

### When Starting a Task
- State your understanding of the task
- Ask clarifying questions if anything is unclear
- Outline your approach before coding
- Wait for confirmation before proceeding

### During Implementation
- Provide status updates after major milestones
- Flag any issues or blockers immediately
- Ask questions when you encounter ambiguity
- Explain your reasoning for technical decisions

### When Completing a Task
- Run all tests and report results
- Document any known issues or limitations
- Suggest next steps
- Wait for confirmation before moving to next phase

### Example Communication Pattern
```
Claude: "I understand we're implementing the badge qualification logic. 
My approach will be:
1. Query badges table ordered by days_required ASC
2. Compare user's days_sober to each badge's requirement
3. Return list of earned badges

Before I start coding, does this approach align with your expectations?"

[Wait for response]

Claude: "Acknowledged. I'll implement this now and write tests."

[After implementation]

Claude: "Badge logic implemented. All unit tests passing (8/8). 
Ready to move to servlet integration, or would you like to review 
the badge logic first?"
```

## Important Notes & Reminders

### For Future Claude Sessions

When you (a new Claude instance) start working on this project:

1. **First, read this entire document** - understand the mission, constraints, and approach
2. **Check `/opt/apps/justfortoday/CLAUDE.md`** - it will be moved to `~/.claude/CLAUDE.md` before you start
3. **Verify directory structure** - `/opt/apps/` for code, `/opt/dta/` for data
4. **Remember the vi constraint** - use vi for all editing
5. **Ask questions** - if anything is unclear, ASK before assuming
6. **Follow TDD** - write tests, watch them fail, make them pass
7. **Check Docker logs** - when things break, logs are your first stop
8. **Commit frequently** - small, atomic commits with clear messages
9. **Think about the user** - someone in recovery, making this work matters

### Why This Project Exists

This isn't just a coding exercise. Real people struggling with addiction could benefit from this tool. Every line of code should be written with:
- **Respect** for the user's journey
- **Care** in the implementation
- **Pride** in the craftsmanship
- **Hope** that it helps someone stay sober one more day

### Final Checklist Before Starting Development

- [ ] WSL environment fully configured
- [ ] Java 17, Maven, Git, Docker all installed and verified
- [ ] Claude Code installed and authenticated
- [ ] SSH keys configured for GitHub
- [ ] Directories created: `/opt/apps/justfortoday/` and `/opt/dta/justfortoday/`
- [ ] Global `~/.claude/CLAUDE.md` in place
- [ ] Project `CLAUDE.md` reviewed and understood
- [ ] GitHub repository created (optional - can be done later)
- [ ] Ready to start Phase 1: Foundation

---

**Remember**: One day at a time. One test at a time. One commit at a time.

Just for today, we build something that matters.
