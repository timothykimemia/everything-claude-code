# Resume Instructions for Claude Code Sessions

This guide explains how to resume a Claude Code session after it has ended, using the session recovery documentation to restore full context.

---

## Quick Resume (2 minutes)

### Step 1: Read Session Recovery
```bash
# Open the session recovery document
cat SESSION_RECOVERY.md
```

### Step 2: Check Git Status
```bash
# See what was being worked on
git status
git log --oneline -5
git diff HEAD
```

### Step 3: Provide Context to Claude

Copy and paste this template to Claude:

```
RESUMING PREVIOUS SESSION

Please review the following to restore context:

1. SESSION_RECOVERY.md contents:
[Paste relevant sections from SESSION_RECOVERY.md]

2. Current git status:
[Paste git status output]

3. Recent commits:
[Paste git log output]

4. What I was working on:
[Brief description of the task]

5. What needs to be done next:
[Next steps from SESSION_RECOVERY.md]

Please confirm you understand the context and can continue from where we left off.
```

---

## Detailed Resume (5-10 minutes)

For complex sessions requiring full context restoration:

### Step 1: Environment Check

```bash
# Verify you're in the correct directory
pwd

# Check git branch
git branch

# Check for uncommitted changes
git status

# Review recent work
git log --oneline -10

# Check for stashed changes
git stash list
```

### Step 2: Review Session Documentation

Read these files in order:

1. **SESSION_RECOVERY.md** - Full session state
2. **Git commit messages** - What was completed
3. **Modified files** - What changed

```bash
# List recently modified files
find . -type f -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \
  -o -name "*.php" -o -name "*.py" -o -name "*.dart" -o -name "*.java" \
  | xargs ls -lt | head -20
```

### Step 3: Test Current State

```bash
# For JavaScript/TypeScript projects
npm run test
npm run build

# For PHP/Laravel projects
php artisan test
composer validate

# For Python/Django projects
python manage.py test
python manage.py check

# For Flutter projects
flutter test
flutter analyze

# For React Native projects
npm test
react-native doctor

# For Java/Spring Boot projects
mvn test
mvn verify
```

### Step 4: Restore Claude Context

Provide Claude with comprehensive context:

```
COMPREHENSIVE SESSION RESTORATION

Project: [Project Name]
Branch: [Branch Name]
Last Session Date: [Date]

## Session Summary
[Copy "Quick Context Summary" from SESSION_RECOVERY.md]

## Active Tasks
[Copy "Active Tasks" section]

## Recent Changes
[List files created/modified from SESSION_RECOVERY.md]

## Architecture Decisions
[Copy "Architecture Decisions" section]

## Outstanding Issues
[Copy "Unresolved Issues" section]

## Test Status
[Copy "Testing Status" section]

## Next Steps
[Copy immediate next steps]

## Additional Context
[Any other relevant information]

Ready to continue? Please confirm context understanding.
```

---

## Language-Specific Resume

### JavaScript/TypeScript Projects

```bash
# Check dependencies
npm list --depth=0
npm outdated

# Check for type errors
npx tsc --noEmit

# Run linter
npm run lint

# Check build
npm run build
```

### PHP/Laravel Projects

```bash
# Check Laravel version
php artisan --version

# Check migrations
php artisan migrate:status

# Check queue status
php artisan queue:failed

# Check logs
tail -n 50 storage/logs/laravel-$(date +%Y-%m-%d).log

# Run tests
php artisan test --parallel
```

### Python/Django Projects

```bash
# Check Python environment
python --version
pip list

# Check Django version
python manage.py --version

# Check migrations
python manage.py showmigrations

# Check for issues
python manage.py check --deploy

# Run tests
pytest -v
```

### Flutter/Dart Projects

```bash
# Check Flutter version
flutter --version

# Check dependencies
flutter pub outdated

# Analyze code
flutter analyze

# Run tests
flutter test

# Check device connectivity
flutter devices
```

### React Native Projects

```bash
# Check React Native version
npx react-native --version

# Check dependencies
npm list react-native

# Run metro bundler check
npx react-native doctor

# Run tests
npm test

# Check for iOS/Android setup
npx react-native info
```

### Java/Spring Boot Projects

```bash
# Check Java version
java -version
mvn --version

# Check dependencies
mvn dependency:tree

# Verify project
mvn verify

# Run tests
mvn test

# Check Spring Boot version
mvn dependency:list | grep spring-boot
```

---

## Common Resume Scenarios

### Scenario 1: Mid-Feature Development

**Context Needed**:
- Feature requirements
- Files already modified
- What's implemented vs. what's pending
- Test status

**Resume Steps**:
1. Read feature description from SESSION_RECOVERY.md
2. Review modified files
3. Check test coverage
4. Continue with next implementation step

### Scenario 2: Debugging Session

**Context Needed**:
- Error description
- Attempted solutions
- Current hypothesis
- Debugging steps taken

**Resume Steps**:
1. Review error logs from SESSION_RECOVERY.md
2. Check if error still reproduces
3. Review attempted solutions
4. Continue debugging from last checkpoint

### Scenario 3: Refactoring

**Context Needed**:
- Refactoring goals
- Pattern being implemented
- Files refactored so far
- Remaining files

**Resume Steps**:
1. Review refactoring plan
2. Check which files completed
3. Verify tests still pass
4. Continue with remaining files

### Scenario 4: Test Writing

**Context Needed**:
- Coverage goals
- Test files created
- Functions tested vs. pending
- Current coverage percentage

**Resume Steps**:
1. Run coverage report
2. Review existing tests
3. Identify gaps
4. Continue writing tests

---

## Information to Provide Claude

### Essential Information

1. **Project Context**
   - Project name and purpose
   - Tech stack (languages, frameworks)
   - Current branch
   - Recent commits

2. **Task Context**
   - What was being worked on
   - Current status
   - Blockers or issues
   - Next steps

3. **Code Context**
   - Modified files
   - Key functions/classes involved
   - Related files

4. **Environment Context**
   - Development environment
   - Dependencies installed
   - Environment variables needed
   - Database state

### Optional Information

- API keys or credentials (never share actual values)
- External service status
- Team decisions or discussions
- Performance metrics
- User feedback

---

## Automation Scripts

### Generate Session Summary

```bash
#!/bin/bash
# save as: generate-session-summary.sh

echo "=== SESSION SUMMARY ==="
echo ""
echo "Branch: $(git branch --show-current)"
echo "Last Commit: $(git log -1 --oneline)"
echo ""
echo "Modified Files:"
git status --short
echo ""
echo "Recent Commits:"
git log --oneline -5
echo ""
echo "Uncommitted Changes:"
git diff --stat
echo ""
echo "=== END SUMMARY ==="
```

### Check Project Health

```bash
#!/bin/bash
# save as: check-project-health.sh

# Detect project type and run appropriate checks
if [ -f "package.json" ]; then
    echo "JavaScript/TypeScript project detected"
    npm test 2>&1 | tail -10
    npm run build 2>&1 | tail -5
elif [ -f "composer.json" ]; then
    echo "PHP/Laravel project detected"
    php artisan test 2>&1 | tail -10
elif [ -f "manage.py" ]; then
    echo "Python/Django project detected"
    python manage.py test 2>&1 | tail -10
elif [ -f "pubspec.yaml" ]; then
    echo "Flutter project detected"
    flutter test 2>&1 | tail -10
elif [ -f "pom.xml" ]; then
    echo "Java/Maven project detected"
    mvn test 2>&1 | tail -10
fi
```

---

## Best Practices

### During Active Session

1. **Update SESSION_RECOVERY.md regularly**
   - After completing each major task
   - Before taking breaks
   - Before ending session

2. **Commit frequently**
   - Small, logical commits
   - Descriptive commit messages
   - Push to remote regularly

3. **Document decisions**
   - Why certain approaches chosen
   - Alternatives considered
   - Trade-offs made

### When Ending Session

1. **Complete current thought**
   - Finish current function/method
   - Ensure code compiles/runs
   - Commit work in progress

2. **Update documentation**
   - Complete SESSION_RECOVERY.md
   - Add TODO comments in code
   - Update project README if needed

3. **Create checkpoint**
   - Commit with clear message
   - Update session checkpoint in SESSION_RECOVERY.md
   - Note next steps

### When Resuming Session

1. **Don't assume Claude remembers**
   - Always provide full context
   - Share relevant documentation
   - Explain current state

2. **Verify understanding**
   - Ask Claude to summarize understanding
   - Confirm next steps
   - Clarify any ambiguities

3. **Start with verification**
   - Run tests
   - Check builds
   - Verify environment

---

## Troubleshooting Resume Issues

### Claude Doesn't Understand Context

**Solution**: Provide more specific information
```
I need you to understand this specific context:

1. We were implementing [feature]
2. The code is in [files]
3. The approach we decided on was [approach]
4. Current status: [status]
5. Next step: [step]

Can you confirm you understand this context?
```

### Missing Critical Information

**Solution**: Reconstruct from git history
```bash
# See what files changed
git diff HEAD~5 --stat

# See detailed changes
git diff HEAD~5

# See commit messages
git log --oneline -10

# See specific file history
git log -p -- path/to/file
```

### Environment Doesn't Match

**Solution**: Restore environment state
```bash
# Reinstall dependencies
npm install  # or pip install -r requirements.txt, composer install, etc.

# Reset database
# (Project-specific commands)

# Clear caches
# (Project-specific commands)
```

---

## Templates

### Minimal Resume Template

```
Resuming work on [PROJECT].

Last working on: [TASK]
Current status: [STATUS]
Next step: [NEXT_STEP]

Modified files:
- file1.ext
- file2.ext

Please continue from here.
```

### Detailed Resume Template

```
RESUMING SESSION - [PROJECT NAME]

CONTEXT:
Branch: [branch]
Last commit: [commit hash]
Session date: [date]

TASK SUMMARY:
[2-3 sentences describing what was being worked on]

PROGRESS:
✅ Completed:
   - Item 1
   - Item 2

🔄 In Progress:
   - Item 3 (50% complete)

⏳ Pending:
   - Item 4
   - Item 5

CODE CHANGES:
Created:
- file1.ext - Description
- file2.ext - Description

Modified:
- file3.ext - Changes made
- file4.ext - Changes made

DECISIONS MADE:
1. Decision 1 - Rationale
2. Decision 2 - Rationale

CURRENT STATE:
- Tests: X passing, Y failing
- Build: Success/Failure
- Issues: [Any blockers]

NEXT STEPS:
1. Step 1
2. Step 2
3. Step 3

Ready to continue?
```

---

## Integration with Your Workflow

### Update Your CLAUDE.md

Add this to your project or user CLAUDE.md:

```markdown
## Session Recovery Protocol

When ending a session:
1. Update SESSION_RECOVERY.md with the current state
2. Commit work with a descriptive message
3. Note next steps clearly

When starting a session:
1. Read SESSION_RECOVERY.md
2. Check git status and recent commits
3. Provide a context summary to Claude
4. Verify the environment before continuing
```

---

**Remember**: The more context you provide, the better Claude can resume from exactly where you left off. Treat each new session as if Claude is a new team member joining your project - provide all necessary context for productive collaboration.
