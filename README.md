# Everything Claude Code

**The complete collection of Claude Code configs from an Anthropic hackathon winner.**

This repo contains production-ready agents, skills, hooks, commands, rules, and MCP configurations that I use daily with Claude Code. These configs evolved over 10+ months of intensive use building real products.

---

## Read the Full Guide First

**Before diving into these configs, read the complete guide on X:**


<img width="592" height="445" alt="image" src="https://github.com/user-attachments/assets/1a471488-59cc-425b-8345-5245c7efbcef" />


**[The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)**



The guide explains:
- What each config type does and when to use it
- How to structure your Claude Code setup
- Context window management (critical for performance)
- Parallel workflows and advanced techniques
- The philosophy behind these configs

**This repo is configs only! Tips, tricks and more examples are in my X articles and videos (links will be appended to this readme as it evolves).**

---

## What's Inside

```
everything-claude-code/
|-- agents/           # Specialized subagents for delegation
|   |-- planner.md                # Feature implementation planning
|   |-- architect.md              # System design decisions
|   |-- tdd-guide.md              # Test-driven development
|   |-- code-reviewer.md          # Quality and security review
|   |-- security-reviewer.md      # Vulnerability analysis
|   |-- build-error-resolver.md   # Build failure resolution
|   |-- e2e-runner.md             # Playwright E2E testing
|   |-- refactor-cleaner.md       # Dead code cleanup
|   |-- doc-updater.md            # Documentation sync
|   |-- laravel-specialist.md     # PHP/Laravel specialist
|   |-- python-django-specialist.md # Python/Django specialist
|
|-- skills/           # Workflow definitions and domain knowledge
|   |-- coding-standards.md       # JavaScript/TypeScript best practices
|   |-- php-laravel-standards.md  # PHP & Laravel standards
|   |-- python-standards.md       # Python & Django/FastAPI standards
|   |-- flutter-dart-standards.md # Flutter & Dart standards
|   |-- react-native-standards.md # React Native standards
|   |-- java-springboot-standards.md # Java & Spring Boot standards
|   |-- backend-patterns.md       # API, database, caching patterns
|   |-- frontend-patterns.md      # React, Next.js patterns
|   |-- project-guidelines-example.md # Example project-specific skill
|   |-- tdd-workflow/             # TDD methodology
|   |-- security-review/          # Security checklist
|   |-- clickhouse-io.md          # ClickHouse analytics
|
|-- commands/         # Slash commands for quick execution
|   |-- tdd.md              # /tdd - Test-driven development
|   |-- plan.md             # /plan - Implementation planning
|   |-- e2e.md              # /e2e - E2E test generation
|   |-- code-review.md      # /code-review - Quality review
|   |-- build-fix.md        # /build-fix - Fix build errors
|   |-- refactor-clean.md   # /refactor-clean - Dead code removal
|   |-- test-coverage.md    # /test-coverage - Coverage analysis
|   |-- update-codemaps.md  # /update-codemaps - Refresh docs
|   |-- update-docs.md      # /update-docs - Sync documentation
|
|-- rules/            # Always-follow guidelines
|   |-- security.md         # Mandatory security checks
|   |-- coding-style.md     # Immutability, file organization
|   |-- testing.md          # TDD, 80% coverage requirement
|   |-- git-workflow.md     # Commit format, PR process
|   |-- agents.md           # When to delegate to subagents
|   |-- performance.md      # Model selection, context management
|   |-- patterns.md         # API response formats, hooks
|   |-- hooks.md            # Hook documentation
|
|-- hooks/            # Trigger-based automations
|   |-- hooks.json          # PreToolUse, PostToolUse, Stop hooks
|
|-- mcp-configs/      # MCP server configurations
|   |-- mcp-servers.json    # GitHub, Supabase, Vercel, Railway, etc.
|
|-- plugins/          # Plugin ecosystem documentation
|   |-- README.md           # Plugins, marketplaces, skills guide
|
|-- examples/         # Example configurations
|   |-- CLAUDE.md           # Example project-level config
|   |-- user-CLAUDE.md      # Example user-level config (~/.claude/CLAUDE.md)
|   |-- statusline.json     # Custom status line config
|
|-- SESSION_RECOVERY.md     # Session state tracking for recovery
|-- RESUME_INSTRUCTIONS.md  # Guide for resuming Claude sessions
```

---

## Multi-Language Support

This configuration now supports **6 major languages and frameworks**:

### Supported Languages

| Language | Frameworks | Skills | Agents | Standards |
|----------|-----------|--------|--------|-----------|
| **JavaScript/TypeScript** | React, Next.js, Node.js | ✅ | ✅ | Complete |
| **PHP** | Laravel | ✅ | ✅ | Complete |
| **Python** | Django, FastAPI, Flask | ✅ | ✅ | Complete |
| **Dart** | Flutter | ✅ | - | Complete |
| **JavaScript** | React Native | ✅ | - | Complete |
| **Java** | Spring Boot | ✅ | - | Complete |

### Language-Specific Features

**PHP/Laravel**:
- Eloquent ORM patterns & query optimization
- Migration and seeding best practices
- Form Request validation patterns
- API Resource transformations
- Artisan command development
- PHPUnit/Pest testing standards
- Laravel-specific security patterns

**Python/Django**:
- Django ORM optimization (N+1 prevention)
- Django REST Framework patterns
- Celery background task patterns
- Type hints (Python 3.10+)
- FastAPI async patterns
- pytest-django testing
- Python security best practices

**Flutter/Dart**:
- Null-safety patterns
- State management (Riverpod)
- Widget composition best practices
- Dart 3+ features (records, sealed classes)
- Navigation patterns (GoRouter)
- Clean architecture layers
- Widget and integration testing

**React Native**:
- Type-safe navigation (React Navigation)
- State management (Zustand)
- Platform-specific code patterns
- Performance optimization (FlatList, memoization)
- API integration patterns
- Testing Library best practices
- Accessibility standards

**Java/Spring Boot**:
- Modern Java features (17+: records, sealed classes, pattern matching)
- Spring Boot 3.x patterns
- JPA/Hibernate optimization
- REST API with proper validation
- MapStruct for DTO mapping
- JUnit 5 testing patterns
- Spring Security configuration

### Hooks Support

The hooks system now supports all languages:

- **Auto-formatting**: JavaScript/TypeScript (Prettier), PHP (php-cs-fixer), Python (black), Dart (dart format), Java (google-java-format)
- **Debug warnings**: console.log (JS/TS), var_dump/dd (PHP), print() (Python)
- **Dev servers**: npm, Laravel serve, Django runserver, Flutter run, React Native, Spring Boot
- **Test runners**: Jest, PHPUnit, Pest, pytest, Flutter test, JUnit/Maven

---

## Quick Start

### 1. Copy what you need

```bash
# Clone the repo
git clone https://github.com/affaan-m/everything-claude-code.git

# Copy agents to your Claude config
cp everything-claude-code/agents/*.md ~/.claude/agents/

# Copy rules
cp everything-claude-code/rules/*.md ~/.claude/rules/

# Copy commands
cp everything-claude-code/commands/*.md ~/.claude/commands/

# Copy skills
cp -r everything-claude-code/skills/* ~/.claude/skills/
```

### 2. Add hooks to settings.json

Copy the hooks from `hooks/hooks.json` to your `~/.claude/settings.json`.

### 3. Configure MCPs

Copy desired MCP servers from `mcp-configs/mcp-servers.json` to your `~/.claude.json`.

**Important:** Replace `YOUR_*_HERE` placeholders with your actual API keys.

### 4. Read the guide

Seriously, [read the guide](https://x.com/affaanmustafa/status/2012378465664745795). These configs make 10x more sense with context.

---

## Key Concepts

### Agents

Subagents handle delegated tasks with limited scope. Example:

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability
tools: Read, Grep, Glob, Bash
model: opus
---

You are a senior code reviewer...
```

### Skills

Skills are workflow definitions invoked by commands or agents:

```markdown
# TDD Workflow

1. Define interfaces first
2. Write failing tests (RED)
3. Implement minimal code (GREEN)
4. Refactor (IMPROVE)
5. Verify 80%+ coverage
```

### Hooks

Hooks fire on tool events. Example - warn about console.log:

```json
{
  "matcher": "tool == \"Edit\" && tool_input.file_path matches \"\\\\.(ts|tsx|js|jsx)$\"",
  "hooks": [{
    "type": "command",
    "command": "#!/bin/bash\ngrep -n 'console\\.log' \"$file_path\" && echo '[Hook] Remove console.log' >&2"
  }]
}
```

### Rules

Rules are always-follow guidelines. Keep them modular:

```
~/.claude/rules/
  security.md      # No hardcoded secrets
  coding-style.md  # Immutability, file limits
  testing.md       # TDD, coverage requirements
```

---

## Contributing

**Contributions are welcome and encouraged.**

This repo is meant to be a community resource. If you have:
- Useful agents or skills
- Clever hooks
- Better MCP configurations
- Improved rules

Please contribute! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Ideas for Contributions

- Additional language support (Go, Rust, C#, Kotlin)
- Framework-specific configs (Rails, ASP.NET, Express)
- DevOps agents (Kubernetes, Terraform, AWS)
- Testing strategies (different frameworks)
- Domain-specific knowledge (ML, data engineering, mobile)
- Additional agents for other frameworks

---

## Session Recovery

**New Feature**: Session recovery system to maintain context across Claude sessions.

### Why Session Recovery?

Claude Code sessions are stateless - each new session starts fresh with no memory of previous work. This can be frustrating when:
- Your session times out mid-task
- You need to switch contexts
- You want to continue work after a break

### How It Works

1. **SESSION_RECOVERY.md** - Automatically tracks:
   - Current tasks and their status
   - Files created/modified/deleted
   - Architecture decisions made
   - Problems encountered and solutions
   - Git status and recent commits
   - Testing status and coverage
   - Environment and dependency changes

2. **RESUME_INSTRUCTIONS.md** - Complete guide for:
   - Quick resume (2 minutes)
   - Detailed resume (5-10 minutes)
   - Language-specific resume procedures
   - Troubleshooting resume issues
   - Templates for different scenarios

### Usage

**When ending a session**:
```bash
# Update SESSION_RECOVERY.md with:
- Current task status
- Next steps
- Any blockers
- Modified files
```

**When starting a new session**:
```bash
# 1. Read session recovery
cat SESSION_RECOVERY.md

# 2. Check git status
git status && git log --oneline -5

# 3. Provide Claude with context from SESSION_RECOVERY.md
```

See [RESUME_INSTRUCTIONS.md](RESUME_INSTRUCTIONS.md) for detailed procedures.

---

## Background

I've been using Claude Code since the experimental rollout. Won the Anthropic x Forum Ventures hackathon in Sep 2025 building [zenith.chat](https://zenith.chat) with [@DRodriguezFX](https://x.com/DRodriguezFX) - entirely using Claude Code.

These configs are battle-tested across multiple production applications.

---

## Important Notes

### Context Window Management

**Critical:** Don't enable all MCPs at once. Your 200k context window can shrink to 70k with too many tools enabled.

Rule of thumb:
- Have 20-30 MCPs configured
- Keep under 10 enabled per project
- Under 80 tools active

Use `disabledMcpServers` in project config to disable unused ones.

### Customization

These configs work for my workflow. You should:
1. Start with what resonates
2. Modify for your stack
3. Remove what you don't use
4. Add your own patterns

---

## Links

- **Full Guide:** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **Follow:** [@affaanmustafa](https://x.com/affaanmustafa)
- **zenith.chat:** [zenith.chat](https://zenith.chat)

---

## License

MIT - Use freely, modify as needed, contribute back if you can.

---

**Star this repo if it helps. Read the guide. Build something great.**
