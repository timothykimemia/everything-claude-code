# Performance Optimization

**CRITICAL FOR $20 SUBSCRIPTION**: Optimize token usage to maximize value from 200k context budget.

---

## Token Budget Management ($20 Pro Subscription)

### Context Window: 200k tokens

**Current Session Token Usage Strategy:**
- Monitor usage regularly (shown in UI)
- Keep conversations under 160k tokens (80% threshold)
- Start new session when approaching 160k
- Use SESSION_RECOVERY.md and RESUME_INSTRUCTIONS.md to maintain context

**Token-Heavy Operations to Avoid:**
- ❌ Reading entire large files (>2000 lines) repeatedly
- ❌ Using Task tool when direct tools (Read, Grep, Glob) suffice
- ❌ Asking for summaries of work already completed
- ❌ Repeating context unnecessarily
- ❌ Enabling too many MCPs (>10 per project)
- ❌ Keeping >80 tools active simultaneously

**Token-Efficient Operations:**
- ✅ Use Read tool with offset/limit for large files
- ✅ Use Grep/Glob for targeted searches (don't use Task tool)
- ✅ Use Edit tool for small changes (not Write)
- ✅ Use Bash for sequential operations with && chaining
- ✅ Disable unused MCPs with disabledMcpServers
- ✅ Use language-specific agents only when needed

---

## Model Selection Strategy

**Haiku 4.5** (90% of Sonnet capability, 3x cost savings):
- ✅ Lightweight agents with frequent invocation
- ✅ Pair programming and code generation
- ✅ Worker agents in multi-agent systems
- ✅ Code reviews, style checks, lint fixes
- ✅ Single-file edits and bug fixes
- **Use for**: build-error-resolver, refactor-cleaner, doc-updater

**Sonnet 4.5** (Best coding model - DEFAULT):
- ✅ Main development work
- ✅ Orchestrating multi-agent workflows
- ✅ Complex coding tasks
- ✅ Multi-file refactoring
- **Use for**: Main conversation, planner, tdd-guide

**Opus 4.5** (Deepest reasoning - USE SPARINGLY):
- ✅ Complex architectural decisions
- ✅ Maximum reasoning requirements
- ✅ Research and analysis tasks
- **Use for**: architect, security-reviewer (when critical)

---

## Context Window Management

### When to Start New Session:
- **160k tokens used** (80% of 200k budget)
- Last 20% of context window approaching
- Session becoming slow or unresponsive
- Before large-scale refactoring

### Avoid Last 20% Context for:
- ❌ Large-scale refactoring
- ❌ Feature implementation spanning multiple files
- ❌ Debugging complex interactions
- ❌ Security-critical changes

### Lower Context Sensitivity Tasks (Safe at 160k+):
- ✅ Single-file edits
- ✅ Independent utility creation
- ✅ Documentation updates
- ✅ Simple bug fixes
- ✅ Running tests
- ✅ Git commits

---

## MCP Server Optimization

### Keep <10 MCPs Enabled Per Project

**Recommended Active MCPs (Universal):**
1. **filesystem** - Core file operations (ALWAYS enabled)
2. **github** - Git operations, PR management
3. **postgres** or **supabase** - Database (pick one)

**Enable Only When Needed:**
4. **vercel** - Only for Vercel deployments
5. **railway** - Only for Railway projects
6. **redis** - Only if using Redis
7. **sentry** - Only for error tracking tasks
8. **anthropic** - Only for Claude API integration

**Disable Unused MCPs:**
```json
{
  "disabledMcpServers": [
    "vercel",
    "railway",
    "sentry",
    "clickhouse",
    "redis"
  ]
}
```

### Language-Specific MCPs:
- **PHP/Laravel**: Enable `laravel` MCP only
- **Python/Django**: Enable `django` MCP only
- **Flutter**: Enable `flutter` MCP only
- **React Native**: Enable `react-native` MCP only
- **Java/Spring Boot**: Enable `spring-boot` MCP only

**Do NOT enable all language MCPs simultaneously** - only enable for your current project.

---

## Tool Optimization (Keep Under 80 Tools)

### Core Tools (Always Active):
- Read, Write, Edit (file operations)
- Bash (terminal commands)
- Grep, Glob (search operations)
- Task (agent delegation)

### Disable Unused Tools:
Use `.claude/settings.json` to disable tools not needed for your project:

```json
{
  "disabledTools": [
    "WebSearch",
    "WebFetch",
    "NotebookEdit",
    "Skill"
  ]
}
```

**Only enable WebSearch/WebFetch when:**
- Researching new APIs or libraries
- Checking latest framework versions
- Looking up documentation

---

## Agent Delegation Strategy (Token Efficient)

### When to Use Task Tool:
✅ **Complex multi-step operations** spanning multiple files
✅ **Exploration tasks** when you don't know exact file locations
✅ **Parallel work** - delegate to multiple agents simultaneously

### When NOT to Use Task Tool:
❌ **Simple file reads** - Use Read tool directly
❌ **Targeted searches** - Use Grep/Glob directly
❌ **Single-file edits** - Use Edit tool directly
❌ **Running tests** - Use Bash tool directly

**Example - Token Efficient:**
```
❌ WASTEFUL: Use Task tool to "find and read config file"
✅ EFFICIENT: Glob "**/*config*" → Read specific file
```

---

## Session Recovery for Token Management

### Before Reaching 160k Tokens:

1. **Save Session State**:
   - Update SESSION_RECOVERY.md with current state
   - Document completed work and next steps
   - List any pending tasks

2. **Start New Session**:
   - Read RESUME_INSTRUCTIONS.md
   - Read SESSION_RECOVERY.md
   - Continue where you left off

3. **Benefits**:
   - Fresh 200k token budget
   - Faster response times
   - Better model performance
   - Cost optimization

---

## Cost-Effective Workflow Examples

### Example 1: Bug Fix (Token Efficient)
```
1. Grep for error message (2k tokens)
2. Read specific file (5k tokens)
3. Edit file with fix (3k tokens)
4. Run test (2k tokens)
Total: ~12k tokens
```

### Example 2: Feature Implementation (Moderate Tokens)
```
1. Task → Explore codebase (20k tokens)
2. Read relevant files (15k tokens)
3. Edit multiple files (10k tokens)
4. Run tests (5k tokens)
5. Task → TDD agent for test coverage (15k tokens)
Total: ~65k tokens
```

### Example 3: Large Refactor (High Tokens - Start Fresh Session)
```
1. Task → Plan mode for architecture (30k tokens)
2. Task → Multiple agents for implementation (50k tokens)
3. Read/Edit many files (40k tokens)
4. Run full test suite (10k tokens)
5. Task → Code review (20k tokens)
Total: ~150k tokens (approaching limit - good time to commit and start new session)
```

## Ultrathink + Plan Mode

For complex tasks requiring deep reasoning:
1. Use `ultrathink` for enhanced thinking
2. Enable **Plan Mode** for structured approach
3. "Rev the engine" with multiple critique rounds
4. Use split role sub-agents for diverse analysis

## Build Troubleshooting

If build fails:
1. Use **build-error-resolver** agent
2. Analyze error messages
3. Fix incrementally
4. Verify after each fix
