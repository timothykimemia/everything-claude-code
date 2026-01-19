# Token Optimization Guide for $20 Subscription

**Target Audience**: Organizations with $20/month Claude Code Pro subscriptions
**Context Window**: 200k tokens per session
**Goal**: Maximize productivity while staying within budget

---

## Quick Reference: Token Usage at a Glance

| Token Usage | Action | Status |
|-------------|--------|--------|
| **0-80k** | ✅ Normal operation | Safe |
| **80k-120k** | ⚠️ Monitor usage | Caution |
| **120k-160k** | 🟡 Consider new session soon | Warning |
| **160k-200k** | 🔴 Start new session NOW | Critical |

**Current Session**: Check token usage in UI bottom-right corner

---

## Core Principles

### 1. Use Direct Tools Over Task Tool

**Task tool is expensive** because it spawns a new agent with full context.

#### ❌ Token-Wasteful Pattern:
```
User: "Find the config file and read it"
Claude: Uses Task tool → Explore agent (20k tokens)
Result: 20k tokens to find and read one file
```

#### ✅ Token-Efficient Pattern:
```
User: "Find the config file and read it"
Claude: Glob "**/*config*" → Read file (2k tokens)
Result: 2k tokens for same task (10x savings!)
```

### 2. Use Edit Over Write for Existing Files

**Writing entire files is expensive** - you're sending all content through the API.

#### ❌ Token-Wasteful:
```typescript
// Read file (500 lines = 2k tokens)
// Write entire file (500 lines = 2k tokens)
// Total: 4k tokens to change 1 line
```

#### ✅ Token-Efficient:
```typescript
// Read file (500 lines = 2k tokens)
// Edit with old_string/new_string (100 tokens)
// Total: 2.1k tokens (50% savings!)
```

### 3. Chain Bash Commands Instead of Multiple Calls

#### ❌ Token-Wasteful:
```bash
# Call 1: cd /path/to/project (500 tokens)
# Call 2: npm test (500 tokens)
# Call 3: npm run build (500 tokens)
# Total: 1500 tokens
```

#### ✅ Token-Efficient:
```bash
# Single call: cd /path/to/project && npm test && npm run build
# Total: 600 tokens (60% savings!)
```

### 4. Use Grep/Glob Over Task for Searches

#### ❌ Token-Wasteful:
```
Task → Explore agent to "find all TypeScript files with User type"
Cost: 15k-25k tokens
```

#### ✅ Token-Efficient:
```
Grep "interface User" --type ts
Cost: 2k-3k tokens (8x savings!)
```

---

## Agent Model Selection (Cost Optimization)

### Haiku (3x cheaper than Opus)

**Use for** (90% of Sonnet capability):
- `build-error-resolver` - Fix build errors ✅ **UPDATED**
- `refactor-cleaner` - Dead code removal ✅ **UPDATED**
- `doc-updater` - Documentation sync ✅ **UPDATED**
- `code-reviewer` - Quality checks ✅ **UPDATED**
- `e2e-runner` - Test execution

**Why**: These are pattern-matching tasks that don't need deep reasoning.

### Sonnet (Default - Best Balance)

**Use for**:
- Main conversation (you're reading this with Sonnet)
- `planner` - Feature planning
- `tdd-guide` - Test-driven development
- `laravel-specialist` - PHP/Laravel tasks
- `python-django-specialist` - Python/Django tasks

**Why**: Most complex coding tasks need Sonnet's full capability.

### Opus (Use Sparingly - Most Expensive)

**Use for**:
- `architect` - System design decisions
- `security-reviewer` - Security analysis (critical)
- Complex architectural refactoring

**Why**: Only when you need maximum reasoning depth.

**Cost Comparison**:
```
Opus:   $15 per 1M input tokens
Sonnet: $3 per 1M input tokens (5x cheaper)
Haiku:  $1 per 1M input tokens (15x cheaper!)
```

---

## MCP Server Optimization

### The Problem

Each MCP server adds ~1k-5k tokens to EVERY request.

**Example**:
- 10 MCPs enabled = 10k-50k tokens per request
- With 20 requests per session = 200k-1M tokens wasted!

### The Solution

**Only enable MCPs you're actively using.**

#### Example: Laravel Project

```json
{
  "mcpServers": {
    "filesystem": { /* ALWAYS enabled */ },
    "github": { /* ALWAYS enabled */ },
    "supabase": { /* Enable if using Supabase */ },
    "laravel": { /* Enable for Laravel projects */ }
  },
  "disabledMcpServers": [
    "vercel",        // Not using Vercel
    "railway",       // Not using Railway
    "redis",         // Not using Redis
    "clickhouse",    // Not using ClickHouse
    "sentry",        // Not debugging errors
    "flutter",       // Not a Flutter project
    "django",        // Not a Django project
    "react-native",  // Not a React Native project
    "spring-boot"    // Not a Java project
  ]
}
```

#### Example: Python/Django Project

```json
{
  "mcpServers": {
    "filesystem": { /* ALWAYS enabled */ },
    "github": { /* ALWAYS enabled */ },
    "postgres": { /* Enable if using PostgreSQL */ },
    "django": { /* Enable for Django projects */ }
  },
  "disabledMcpServers": [
    "vercel",
    "railway",
    "redis",
    "sentry",
    "laravel",       // Not a Laravel project
    "flutter",       // Not a Flutter project
    "react-native",  // Not a React Native project
    "spring-boot",   // Not a Java project
    "supabase"       // Using postgres instead
  ]
}
```

**Rule**: **<10 MCPs enabled per project**

---

## Tool Optimization

### The Problem

Each tool adds to context. 80+ tools = bloated context.

### The Solution

**Disable tools you don't need for current project.**

```json
{
  "disabledTools": [
    "WebSearch",      // Only enable when researching APIs
    "WebFetch",       // Only enable when fetching docs
    "NotebookEdit",   // Only enable for Jupyter notebooks
    "Skill"           // Only enable if using skills marketplace
  ]
}
```

**When to re-enable**:
- `WebSearch`/`WebFetch`: Researching new libraries, checking docs
- `NotebookEdit`: Working with Jupyter notebooks
- `Skill`: Using skills from marketplace

---

## Session Recovery Strategy

### When to Start New Session

**Hard Limits**:
- **160k tokens used** (80% of budget)
- Session feels slow or unresponsive
- Before large-scale refactoring (50k+ token tasks)

**Soft Limits**:
- Completing a major feature
- Switching between projects
- End of work day

### How to Recover Session

1. **Before ending session**:
   ```
   User: "Update SESSION_RECOVERY.md with current state"
   Claude: Documents all completed work, next steps, pending tasks
   ```

2. **Starting new session**:
   ```
   User: "Read SESSION_RECOVERY.md and RESUME_INSTRUCTIONS.md"
   Claude: Resumes from where you left off (5k tokens vs 160k!)
   ```

3. **Benefits**:
   - Fresh 200k token budget
   - 32x faster response times
   - Better model performance
   - Significant cost savings

---

## Real-World Cost Examples

### Scenario 1: Bug Fix

**Wasteful Approach** (80k tokens):
```
1. Task → Explore codebase (30k)
2. Task → TDD agent to write tests (20k)
3. Multiple file reads (15k)
4. Task → Code review (15k)
Total: 80k tokens
```

**Efficient Approach** (12k tokens):
```
1. Grep error message (2k)
2. Read specific file (5k)
3. Edit file with fix (3k)
4. Bash: run test (2k)
Total: 12k tokens (6.6x savings!)
```

### Scenario 2: Feature Implementation

**Wasteful Approach** (150k tokens):
```
1. Task → Plan (30k)
2. Task → Explore (30k)
3. Task → Implement (40k)
4. Task → Test (25k)
5. Task → Review (25k)
Total: 150k tokens (need new session)
```

**Efficient Approach** (65k tokens):
```
1. Task → Explore (haiku: 15k)
2. Read relevant files (15k)
3. Edit files directly (10k)
4. Bash: run tests (5k)
5. Task → TDD agent (haiku: 12k)
6. Grep/Edit fixes (8k)
Total: 65k tokens (2.3x savings!)
```

### Scenario 3: Large Refactor

**Smart Approach** (session management):
```
Session 1 (140k tokens):
1. Task → Architect (opus: 40k)
2. Task → Plan implementation (20k)
3. Read/analyze codebase (40k)
4. Begin refactoring (40k)
5. Update SESSION_RECOVERY.md

Session 2 (120k tokens):
1. Resume from SESSION_RECOVERY.md (5k)
2. Continue refactoring (60k)
3. Task → TDD agent (haiku: 15k)
4. Task → Code review (haiku: 20k)
5. Final testing (20k)

Total: 260k tokens across 2 sessions vs 400k in single session
Result: 35% savings + better performance!
```

---

## Pro Tips for Token Efficiency

### 1. Use Language-Specific Agents Wisely

Only invoke when you need deep language expertise:

```
✅ GOOD: "Fix this Laravel validation error" → Main Sonnet
✅ GOOD: "Design a new Laravel service architecture" → laravel-specialist

❌ WASTEFUL: "Read this PHP file" → laravel-specialist (15k)
✅ EFFICIENT: Read tool directly (3k)
```

### 2. Batch Related Operations

```
❌ WASTEFUL:
User: "Read file A"
User: "Read file B"
User: "Read file C"
Cost: 3 requests × 5k = 15k tokens

✅ EFFICIENT:
User: "Read files A, B, and C"
Cost: 1 request × 7k = 7k tokens (53% savings!)
```

### 3. Use Git Strategically

```
❌ WASTEFUL:
Read 10 files to see what changed (50k)

✅ EFFICIENT:
Bash: git diff (5k) → Shows exactly what changed
```

### 4. Avoid Redundant Context

```
❌ WASTEFUL:
User: "What did we just implement?"
Claude: Summarizes everything (10k tokens)

✅ EFFICIENT:
User: "Commit these changes"
Claude: Creates commit from recent work (2k tokens)
Git log shows what was done
```

### 5. Use Plan Mode for Complex Tasks

Plan mode helps Claude think through the task before acting:

```
✅ EFFICIENT:
User: "Plan implementation of user authentication"
Claude: Creates detailed plan (8k tokens)
User: "Execute the plan"
Claude: Implements efficiently (40k tokens)
Total: 48k tokens

❌ WASTEFUL:
User: "Implement user authentication"
Claude: Explores, tries approaches, backtracks (80k tokens)
```

---

## Monitoring Token Usage

### In Claude Code UI

**Bottom-right corner shows**:
- Current token count
- Percentage of budget used
- Model being used

**Color coding**:
- 🟢 Green (0-80k): Safe
- 🟡 Yellow (80k-160k): Monitor
- 🔴 Red (160k+): Start new session

### Token Estimation Guide

| Operation | Approximate Tokens |
|-----------|-------------------|
| Read small file (100 lines) | 500-1k |
| Read medium file (500 lines) | 2k-3k |
| Read large file (2000 lines) | 8k-10k |
| Edit file (small change) | 100-500 |
| Grep search | 1k-3k |
| Glob pattern match | 500-1k |
| Bash command | 500-1k |
| Task → Haiku agent | 10k-20k |
| Task → Sonnet agent | 15k-30k |
| Task → Opus agent | 20k-40k |

---

## Organization Rollout Strategy

### For Teams with $20/user Subscriptions

**Week 1: Setup**
1. Copy this configuration to each developer's `~/.claude/`
2. Configure language-specific MCPs per developer:
   - Frontend devs: React Native, TypeScript
   - Backend devs: Laravel, Django, Spring Boot
   - Mobile devs: Flutter
3. Train on token optimization strategies

**Week 2: Monitor Usage**
1. Each dev tracks token usage per task
2. Identify token-heavy operations
3. Share optimization strategies

**Week 3: Optimize**
1. Disable unused MCPs team-wide
2. Update agent model selections
3. Implement session recovery workflow

**Expected Results**:
- 40-60% token usage reduction
- 2-3x more work per session
- Faster response times
- Better model performance

---

## Common Mistakes to Avoid

### ❌ Mistake 1: Using Task Tool for Everything
```
"Find the bug" → Task → Explore (30k tokens)
Should be: Grep for error message (2k tokens)
```

### ❌ Mistake 2: Reading Entire Files Repeatedly
```
Read 10 files 3 times each = 90k tokens wasted
Should be: Read once, use Edit tool for changes
```

### ❌ Mistake 3: Enabling All MCPs
```
15 MCPs × 3k tokens × 20 requests = 900k tokens wasted
Should be: <10 MCPs, disable unused ones
```

### ❌ Mistake 4: Not Using Session Recovery
```
Single 180k token session (hits limit, loses context)
Should be: Two 90k sessions with SESSION_RECOVERY.md
```

### ❌ Mistake 5: Using Opus for Simple Tasks
```
Code review with Opus = 30k tokens
Code review with Haiku = 10k tokens (same quality!)
```

---

## Summary: Token Optimization Checklist

- [ ] Monitor token usage regularly (check UI)
- [ ] Use direct tools (Read, Grep, Glob) instead of Task when possible
- [ ] Use Edit instead of Write for existing files
- [ ] Chain Bash commands with && instead of multiple calls
- [ ] Enable <10 MCPs per project (disable unused ones)
- [ ] Disable unused tools (WebSearch, WebFetch, etc.)
- [ ] Use Haiku for simple agents (3x cost savings)
- [ ] Use Sonnet for main work (default)
- [ ] Use Opus only for architecture/security (sparingly)
- [ ] Start new session at 160k tokens
- [ ] Use SESSION_RECOVERY.md for session continuity
- [ ] Batch related operations in single request
- [ ] Use Plan Mode for complex tasks
- [ ] Avoid redundant context and summaries

**Target**: Stay under 160k tokens per session for optimal performance and cost.

---

## Quick Wins for Immediate Savings

### 1. Update Agent Models (3 minutes)
```bash
# Already done in this repo!
# build-error-resolver, refactor-cleaner, doc-updater, code-reviewer
# Changed from Opus → Haiku (3x cost savings)
```

### 2. Disable Unused MCPs (2 minutes)
```json
{
  "disabledMcpServers": ["vercel", "railway", "sentry", "clickhouse"]
}
```
**Savings**: 10k-20k tokens per session

### 3. Use Session Recovery (5 minutes)
```
Before ending: "Update SESSION_RECOVERY.md"
Next session: "Read SESSION_RECOVERY.md and continue"
```
**Savings**: 50-100k tokens per resumed session

### 4. Use Direct Tools (ongoing)
```
Replace: Task → Explore
With: Grep + Read
```
**Savings**: 10-20k tokens per search

**Total Immediate Savings**: 40-60% reduction in token usage

---

**Questions? Check rules/performance.md for more details.**
