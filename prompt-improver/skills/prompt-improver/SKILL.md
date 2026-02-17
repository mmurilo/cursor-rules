---
name: prompt-improver
description: Enriches vague or ambiguous prompts with targeted research and clarifying questions before execution. Use when a prompt lacks specifics, context, or a clear target — e.g. "fix the bug", "add tests", "refactor the code". Also useful when the user asks to clarify, research, or improve a request before acting on it.
---

# Prompt Improver Skill

## Purpose

Transform vague, ambiguous prompts into actionable, well-defined requests through systematic research, targeted clarification, and explicit prompt enrichment.

## When to Use

This skill can be invoked in three ways:

**1. Rule-triggered (automatic via prompt-improver rule):**
- The `prompt-improver` rule evaluates every prompt and invokes this skill when it determines the prompt is vague
- Evaluation is already done — proceed directly to Phase 1 (Triage)

**2. Agent-decided (automatic by Cursor):**
- Cursor's agent detects the prompt is vague or ambiguous
- Examples: "fix the bug", "add tests", "improve performance", "refactor the code"
- Briefly note why the prompt is vague, then proceed to Phase 1 (Triage)

**3. Manual invocation (`/prompt-improver`):**
- User explicitly invokes the skill
- Briefly note what will be researched, then proceed to Phase 1 (Triage)

**For all invocation modes:**
- If the prompt is already clear and specific, say so and proceed with execution — do not force unnecessary questions
- If the prompt is vague, follow the workflow below

## Vagueness Evaluation Criteria

Before invoking this skill, evaluate the prompt against these four dimensions:

| Dimension | Question | Pass (✅) | Fail (❌) |
|-----------|----------|-----------|-----------|
| **Target** | Is there a specific file, function, component, or system? | "Fix auth.ts:145" | "Fix the bug" |
| **Action** | Is the desired change or behavior clear? | "Convert to async/await" | "Make it better" |
| **Criteria** | Can you objectively verify success? | "All tests pass" | "Improve quality" |
| **Context** | Is there enough background to act? | Error message provided | No context at all |

**Decision matrix:**
- **4/4 pass** → Prompt is clear. Proceed immediately, do NOT invoke this skill.
- **3/4 pass** → Check conversation history and open files for the missing piece. If found, proceed. If not, invoke skill with light research.
- **≤2/4 pass** → Prompt is vague. Invoke this skill.

**Bypass detection (skip evaluation entirely):**
- `*` prefix → Strip prefix, pass through as-is
- `/` prefix → Slash command, pass through
- `#` prefix → Memory/note command, pass through

## Core Workflow

### Phase 1: Triage

Quickly assess the effort level to determine research depth.

**Think step by step:**
1. What type of change is this? (bug fix, feature, refactor, config, docs)
2. How many files/systems are likely involved?
3. What's the blast radius if done wrong?

**Assign a triage level:**

| Level | Signal | Research Depth | Max Questions |
|-------|--------|---------------|---------------|
| **Light** | Single file, small scope, low risk | 1-2 searches | 1-2 |
| **Moderate** | Multiple files, unclear approach | 3-4 searches | 3-4 |
| **Deep** | Architectural, multi-system, high risk | 5+ searches | 4-6 |

### Phase 2: Research

Create and execute a research plan before asking questions.

**Research checklist:**
1. **Check conversation history first** — Avoid redundant exploration if context already exists
2. **Review codebase** as needed:
   - Search for relevant files, architecture, and project structure
   - Search for specific patterns, related files, error messages
   - Check git log for recent changes
   - Look for TODOs, FIXMEs, failing tests
3. **Gather additional context** as needed:
   - Read local documentation (README, docs/, CONTRIBUTING)
   - Search the web for best practices, common approaches
4. **Tag each finding with confidence:**
   - **HIGH** — Direct evidence (error message, failing test, explicit code)
   - **MEDIUM** — Inferred from patterns (similar code, naming conventions)
   - **LOW** — External best practice or assumption

**Critical rules:**
- NEVER skip research
- Check conversation history before exploring codebase
- Questions must be grounded in actual findings, not assumptions

For detailed strategies, see [references/research-strategies.md](references/research-strategies.md).

### Phase 3: Generate Targeted Questions

Based on research findings, formulate questions that resolve the ambiguity.

**Self-evaluation checkpoint (before presenting questions):**
- [ ] Does each question address a gap identified in evaluation?
- [ ] Is every option grounded in a research finding (not an assumption)?
- [ ] Are there zero vague options like "Other approach" or "Best practice"?
- [ ] Does the set of questions, once answered, make the prompt fully actionable?

If any check fails, go back and research more or refine the questions.

**Question format (present as numbered list in chat):**
```
Based on my research, I have a few questions:

1. **[Short Label]** — [Clear question ending with ?]
   - **[Option A]** — [Context from research, trade-offs]
   - **[Option B]** — [Context from research, trade-offs]
   - **[Option C]** — [Context from research, trade-offs]
```

**Guidelines:**
- 1-6 questions based on triage level
- 2-4 concrete options per question, each grounded in research
- Include trade-offs and implications in each option
- End each question with `?`
- User can always provide a custom answer beyond listed options

For question templates and anti-patterns, see [references/question-patterns.md](references/question-patterns.md).

### Phase 4: Get Clarification

Present your research-grounded questions to the user. Wait for answers.

**If user answers introduce new ambiguity:**
- Ask at most 1 follow-up question to resolve it
- If still unclear after follow-up, make a reasonable choice and state your assumption explicitly

### Phase 5: Synthesize Enriched Prompt

**Before executing, compose an enriched prompt** that combines:
- Original user intent
- Research findings (with confidence tags)
- User's clarification answers
- Conversation history context

**Present the enriched prompt to the user:**
```
Here's what I'll do:

**Task:** [Clear, specific description of what will be done]
**Target:** [Specific files/functions/systems]
**Approach:** [How it will be implemented]
**Success criteria:** [How to verify it worked]

Proceeding now.
```

This serves as a final confirmation and creates an auditable record of the transformation from vague → specific.

### Phase 6: Execute

Proceed with the enriched prompt as if it had been clear from the start.

## Examples

### Example 1: Vague Prompt → Research → Questions → Enriched Execution

**Original prompt:** "fix the bug"
**Evaluation:** Target ❌, Action ❌, Criteria ~, Context ❌ → **VAGUE**
**Triage:** Light (bug fix, likely single file)

**Research findings:**
- [HIGH] Recent conversation mentions login failures
- [HIGH] auth.py:145 has try/catch swallowing errors, 2 tests failing
- [MEDIUM] Recent commit "fix login redirect" 2 days ago

**Question:**
1. **Bug target** — Which bug should be fixed?
   - **Token validation (auth.py:145)** — FIXME comment, 2 failing tests in test_auth.py. Likely highest priority.
   - **Login redirect (recent commit)** — Attempted fix 2 days ago, may have residual issues.

**User answer:** Token validation

**Enriched prompt:**
> **Task:** Fix token validation bug in auth.py:145
> **Target:** auth.py:145, test_auth.py
> **Approach:** Fix the try/catch that swallows errors in token validation
> **Success criteria:** 2 failing tests in test_auth.py pass

### Example 2: Clear Prompt (Skill Not Invoked)

**Original prompt:** "Refactor the getUserById function in src/api/users.ts to use async/await instead of promises"
**Evaluation:** Target ✅, Action ✅, Criteria ✅, Context ✅ → **CLEAR**
**Decision:** Proceed immediately. No research or questions needed.

For more examples across different languages and scenarios, see [references/examples.md](references/examples.md).

## Key Principles

1. **Evaluate First** — Use the 4-dimension checklist before invoking
2. **Triage Effort** — Match research depth to prompt complexity
3. **Research Before Asking** — Always gather context before formulating questions
4. **Ground Everything** — Use research findings, not assumptions
5. **Self-Evaluate** — Verify question quality before presenting
6. **Synthesize Explicitly** — Compose an enriched prompt before executing
7. **Limit Friction** — Max 1 follow-up round if answers introduce ambiguity
8. **Think Step by Step** — Reason through vagueness assessment and research planning

## Progressive Disclosure

This SKILL.md contains the core workflow and essentials. For deeper guidance:

- **Research strategies**: [references/research-strategies.md](references/research-strategies.md) — How to gather context efficiently
- **Question patterns**: [references/question-patterns.md](references/question-patterns.md) — Templates and anti-patterns
- **Comprehensive examples**: [references/examples.md](references/examples.md) — Full flows across multiple languages

Load these references only when detailed guidance is needed.
