# Question Patterns

Templates and anti-patterns for formulating clarifying questions grounded in research.

## Core Principles

1. **Ground in research** — Every option must trace back to a finding (codebase, docs, web search)
2. **Be specific** — "Use JWT with HttpOnly cookies" not "Use a different approach"
3. **Include trade-offs** — Why this option, when it's appropriate, what it costs
4. **One decision per question** — Don't combine "which file AND which approach"
5. **2-4 options per question** — Fewer than 2 isn't a choice; more than 4 is overwhelming

## Question Format

Present questions as a numbered markdown list in chat:

```
Based on my research, I have a few questions:

1. **[Short Label]** — [Clear question ending with ?]
   - **[Option A]** — [What, why, trade-offs. Grounded in: specific finding]
   - **[Option B]** — [What, why, trade-offs. Grounded in: specific finding]
   - **[Option C]** — [What, why, trade-offs. Grounded in: specific finding]
```

## Templates by Category

### Target Identification
**When:** Unclear which file, function, or component to modify.

```
1. **Target** — Which file should be modified?
   - **src/auth/login.ts** — Main login handler, 245 lines. Contains auth logic.
   - **src/auth/middleware.ts** — Auth middleware for protected routes, 89 lines.
   - **src/auth/session.ts** — Session management and validation, 156 lines.
```

### Approach Selection
**When:** Target is clear but implementation method is ambiguous.

```
1. **Approach** — How should validation be organized?
   - **Middleware** — Runs before route handlers. Reusable, centralized error handling.
   - **Service layer** — Inside business logic classes. Easier to unit test.
   - **Schema-based** — Declarative JSON schemas. Auto-generates docs, type-safe.
```

### Scope Definition
**When:** Unclear how much work should be done.

```
1. **Scope** — What should this refactoring cover?
   - **Single function** — Just getUserById(). Minimal change, quick to test.
   - **Entire class** — All UserRepository methods (8 functions). Consistent patterns.
   - **All repositories** — User, Product, Order (3 classes, 24 functions). Codebase-wide.
```

### Configuration Choices
**When:** Implementation requires choosing tools, libraries, or values.

```
1. **Library** — Which validation library should be used?
   - **Zod** — TypeScript-first, excellent type inference. Already used in frontend.
   - **Joi** — Mature, large ecosystem. Common in enterprise codebases.
```

## Number of Questions by Triage Level

| Triage | Questions | When |
|--------|-----------|------|
| **Light** | 1-2 | Single ambiguity (which file? which bug?) |
| **Moderate** | 3-4 | Multiple decisions (scope + approach + tool) |
| **Deep** | 4-6 | Architectural choices with many decision points |

## Anti-Patterns

### ❌ Generic options
**Bad:** `"Best practice approach" — Use industry standard methods`
**Good:** `"Repository pattern with DI" — Follows OrderService pattern in src/services/order.service.ts:15`

### ❌ Missing trade-offs
**Bad:** `"Microservices" — Modern, scalable approach`
**Good:** `"Microservices" — Better scaling, but adds deployment complexity. Team has Docker expertise.`

### ❌ Leading questions
**Bad:** `"Should we use the superior JWT approach?"`
**Good:** `"Which authentication mechanism should be implemented?"`

### ❌ Compound questions
**Bad:** `"Which library and what configuration?"`
**Good:** Split into separate questions — one for library, one for configuration.

### ❌ Ungrounded options
**Bad:** `"Some approach" — I think this might work`
**Good:** Research first, then generate options from actual findings.

## Self-Evaluation Checklist

Before presenting questions, verify:

- [ ] Every option traces to a research finding
- [ ] Every option is specific and actionable
- [ ] Trade-offs are included in descriptions
- [ ] Questions are independent (answerable in any order)
- [ ] Total questions match triage level
- [ ] Answering all questions makes the prompt fully actionable
