# Session Recovery Documentation

This document tracks the current Claude Code session state, context, and progress to enable seamless recovery when starting a new session.

---

## Session Metadata

**Last Updated**: <!-- Auto-update this timestamp -->
**Session ID**: <!-- Unique identifier for this session -->
**Project**: everything-claude-code
**Branch**: <!-- Current git branch -->
**Claude Model**: Sonnet 4.5

---

## Current Working Context

### Active Tasks
<!-- List all tasks currently in progress -->

1. **Task Name**:
   - **Status**: In Progress / Completed / Blocked
   - **Description**:
   - **Files Modified**:
   - **Next Steps**:

2. **Task Name**:
   - **Status**:
   - **Description**:
   - **Files Modified**:
   - **Next Steps**:

### Pending Tasks
<!-- Tasks identified but not started -->

- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

---

## Code Changes Summary

### Files Created
<!-- List all new files created in this session -->

```
path/to/new/file1.ext - Description
path/to/new/file2.ext - Description
```

### Files Modified
<!-- List all modified files with summary of changes -->

```
path/to/modified/file1.ext
- Change 1 description
- Change 2 description

path/to/modified/file2.ext
- Change description
```

### Files Deleted
<!-- List any deleted files -->

```
path/to/deleted/file.ext - Reason for deletion
```

---

## Git Status

### Current Branch
```bash
# Branch name
main
```

### Uncommitted Changes
```bash
# Run: git status
# Output summary:
```

### Recent Commits
```bash
# Run: git log --oneline -5
# Output:
```

### Staged Changes
```bash
# Run: git diff --cached --stat
# Output:
```

---

## Architecture Decisions

### Decisions Made
<!-- Document important architectural decisions made during the session -->

1. **Decision**:
   - **Rationale**:
   - **Impact**:
   - **Files Affected**:

2. **Decision**:
   - **Rationale**:
   - **Impact**:
   - **Files Affected**:

---

## Problems Encountered

### Resolved Issues

1. **Issue**:
   - **Error**:
   - **Solution**:
   - **Files Modified**:

### Unresolved Issues

1. **Issue**:
   - **Error**:
   - **Attempted Solutions**:
   - **Current Status**:
   - **Next Steps**:

---

## Dependencies & Environment

### Installed Packages
<!-- Track newly installed dependencies -->

**JavaScript/TypeScript**:
```json
{
  "dependencies": {},
  "devDependencies": {}
}
```

**PHP/Laravel**:
```json
{
  "require": {},
  "require-dev": {}
}
```

**Python**:
```
package==version
```

**Flutter**:
```yaml
dependencies:
  package: ^version
```

**Java/Maven**:
```xml
<dependency>
  <groupId>group</groupId>
  <artifactId>artifact</artifactId>
  <version>version</version>
</dependency>
```

### Environment Variables
<!-- Track new environment variables needed -->

```bash
VARIABLE_NAME=value # Description
```

### Database Migrations
<!-- Track database changes -->

**Pending Migrations**:
```
migration_name - Description
```

**Applied Migrations**:
```
migration_name - Description
```

---

## Testing Status

### Tests Written
```
path/to/test/file.test.ext - Description
```

### Tests Passing/Failing
```
✅ Test Suite 1 (12/12 passing)
❌ Test Suite 2 (8/10 passing - 2 failing)
   - test_name_1: failure reason
   - test_name_2: failure reason
```

### Coverage
```
Overall Coverage: X%
Critical Paths: Y%
```

---

## Configuration Changes

### Updated Config Files

**File**: path/to/config.ext
```
Key changes:
- Change 1
- Change 2
```

---

## External Resources

### API Integrations
<!-- Track external APIs used or configured -->

- **API Name**: Purpose, authentication method, endpoint

### Documentation References
<!-- Important documentation consulted -->

- [Link to documentation](url) - Topic

---

## Performance Considerations

### Optimizations Made
1. Optimization description
   - Impact:
   - Metrics:

### Performance Issues Identified
1. Issue description
   - Current Performance:
   - Target Performance:
   - Proposed Solution:

---

## Security Considerations

### Security Measures Implemented
1. Measure description
   - Files affected:
   - Vulnerabilities addressed:

### Security Issues Identified
1. Issue description
   - Severity: Critical/High/Medium/Low
   - Status: Addressed/Pending
   - Remediation plan:

---

## Communication Log

### Key Decisions Communicated
<!-- Important discussions with user -->

1. **Topic**:
   - **User Request**:
   - **Claude Response**:
   - **Outcome**:

---

## Session Checkpoints

### Checkpoint 1: [Timestamp]
**State**: Description of project state
**Achievements**: What was completed
**Next**: What should happen next

### Checkpoint 2: [Timestamp]
**State**: Description of project state
**Achievements**: What was completed
**Next**: What should happen next

---

## Resume Instructions

**To resume this session in a new Claude Code instance**:

1. Read this file (`SESSION_RECOVERY.md`)
2. Read `RESUME_INSTRUCTIONS.md` for detailed restoration steps
3. Check git status: `git status`
4. Review uncommitted changes: `git diff`
5. Review recent commits: `git log --oneline -10`
6. Continue with "Next Steps" from the active task above

---

## Quick Context Summary

<!-- 2-3 sentence summary of what was being worked on -->

**Summary**:


**Current Focus**:


**Blockers**:


**Immediate Next Steps**:
1.
2.
3.

---

**Note**: Update this file at the end of each significant task or before ending the session.
