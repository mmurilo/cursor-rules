# Research Strategies

Systematic approaches for gathering context before formulating clarifying questions. Research quality directly determines question quality.

## Research Planning

### Step 1: Identify Gaps

Before researching, name the specific gaps from the vagueness evaluation:

- **Target gap** — "Which file/function/component?"
- **Approach gap** — "What pattern/method/library?"
- **Scope gap** — "How much should change?"
- **Context gap** — "What's the current state?"

### Step 2: Create a Research Checklist

Write a quick checklist before executing research. This ensures systematic investigation and prevents ad-hoc exploration.

```
Research plan for "[prompt]":
- [ ] Check conversation history for [specific thing]
- [ ] Search codebase for [pattern/file/error]
- [ ] Check git history for [recent changes]
- [ ] Search web for [best practice / docs]
```

### Step 3: Execute and Tag Findings

Tag every finding with a confidence level:
- **HIGH** — Direct evidence (error message, failing test, explicit code)
- **MEDIUM** — Inferred (naming patterns, similar code, project conventions)
- **LOW** — External (web best practice, general knowledge)

## Codebase Exploration Strategies

### Pattern Discovery
**When:** Need to understand architecture or find similar implementations.

Search for related files and explore directory structure. Look at entry points (index, main, app files) and configuration files (package.json, pyproject.toml, Cargo.toml, go.mod).

**Example:** For "refactor the API" → find the entry point, map the route/controller/service layers, read a representative file.

### Targeted Search
**When:** Looking for specific patterns, function calls, or keywords.

Effective search patterns:
```
# Authentication
"authenticate|login|auth|session|jwt"

# Known issues
"TODO|FIXME|XXX|HACK|BUG"

# Error handling
"try|catch|throw|Error|except|raise"

# Configuration
"config|env|settings|\.env"

# Test patterns
"test_|_test\.|\.spec\.|\.test\."
```

### Historical Context
**When:** Understanding recent changes, finding related commits.

Useful approaches:
```bash
git log --oneline -20                        # Recent commits
git log --oneline -- path/to/file            # File history
git log --grep="keyword" --oneline           # Search commit messages
git log -S "functionName" --oneline          # When code appeared
git diff HEAD~5..HEAD --stat                 # Recent change scope
```

## Documentation Research

**Priority order for local docs:**
1. README.md at project root
2. docs/ directory
3. Package/module-specific READMEs
4. CONTRIBUTING.md, ARCHITECTURE.md
5. Inline code comments (`NOTE:`, `WARNING:`, `IMPORTANT:`)

**For external docs:** Search the web for official documentation, best practices, and library comparisons relevant to the project's stack.

## Conversation History Mining

**Always check first.** The conversation often contains the missing context:
- Error messages in recent messages
- File names mentioned or opened
- Decisions already made
- Code shown or discussed

**Example:** If user says "fix it" after showing an error → the error IS the context. No codebase research needed.

## Multi-Tool Research Patterns

### Light Research (triage: light)
```
1. Check conversation history
2. Search codebase for 1-2 relevant patterns
→ Enough for simple target/scope clarification
```

### Moderate Research (triage: moderate)
```
1. Check conversation history
2. Search codebase for patterns and related files
3. Read key files to understand current state
4. Search web for best practices (if approach is unclear)
→ Enough for approach + scope decisions
```

### Deep Research (triage: deep)
```
1. Check conversation history
2. Map architecture (entry points, config, key modules)
3. Search for existing patterns and similar implementations
4. Check git history for recent changes and past attempts
5. Read documentation (local + web)
6. Search web for framework-specific best practices
→ Enough for architectural decisions
```

## Summary

- **Always** check conversation history first
- **Tag** every finding with HIGH/MEDIUM/LOW confidence
- **Match** research depth to triage level — don't over-research simple prompts
- **Ground** every question option in a specific finding
- Research is complete when you can generate specific, actionable question options for every identified gap
