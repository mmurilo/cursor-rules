# Examples

Complete flows from vague prompts through research, clarification, and execution.

## Vague Prompts (Research Required)

### Example 1: Generic Bug Fix (Python)

**Prompt:** "fix the bug"
**Evaluation:** Target ❌, Action ❌, Criteria ~, Context ❌ → **VAGUE**
**Triage:** Light

**Research findings:**
- [HIGH] FIXME in auth.py:89: "Token validation is broken"
- [HIGH] 2 tests failing in test_auth.py related to token validation
- [MEDIUM] Recent commit "fix login redirect" 2 days ago
- [LOW] Error logging at auth.py:145 for invalid tokens

**Questions:**
1. **Bug target** — Which bug should be fixed?
   - **Token validation (auth.py:89)** — FIXME comment, 2 failing tests. Likely highest priority.
   - **Login redirect (recent commit)** — Fix attempt 2 days ago, may have residual issues.
   - **Invalid token logging (auth.py:145)** — May be symptom of validation bug or separate issue.

**User answer:** Token validation

**Enriched prompt:**
> **Task:** Fix token validation in auth.py:89
> **Target:** auth.py:89, test_auth.py
> **Approach:** Fix broken token validation causing 2 test failures
> **Success criteria:** Both failing tests in test_auth.py pass

---

### Example 2: Generic Refactoring (TypeScript)

**Prompt:** "refactor the code to be better"
**Evaluation:** Target ❌, Action ❌, Criteria ❌, Context: user has src/api/users.ts open → partial
**Triage:** Moderate

**Research findings:**
- [HIGH] users.ts is 450 lines, all logic in one file
- [HIGH] Other APIs (products.ts) use service layer pattern (ProductService class)
- [MEDIUM] Industry pattern: Controller → Service → Repository separation
- [MEDIUM] updateUser is 89 lines, createUser is 78 lines

**Questions:**
1. **Improvement** — What aspect of users.ts should improve?
   - **Extract service layer** — Create UserService class matching ProductService pattern. Separates concerns, improves testability.
   - **Split into modules** — Break 450-line file into validation.ts, service.ts, controller.ts. Easier navigation.
   - **Reduce function size** — Refactor long functions (updateUser: 89 lines, createUser: 78 lines) into smaller units.

2. **Pattern** — Should this follow the existing ProductService pattern?
   - **Yes, match ProductService** — Consistency with existing codebase. Familiar structure.
   - **No, different approach** — Opportunity to establish an improved pattern.

**User answers:** Extract service layer, Yes match ProductService

**Enriched prompt:**
> **Task:** Extract UserService class from src/api/users.ts
> **Target:** src/api/users.ts → new src/services/UserService.ts
> **Approach:** Follow ProductService pattern, move business logic to service, keep controller thin
> **Success criteria:** users.ts is a thin controller, UserService handles logic, all existing tests pass

---

### Example 3: Ambiguous Feature (Go)

**Prompt:** "add caching"
**Evaluation:** Target ❌, Action ~, Criteria ❌, Context ❌ → **VAGUE**
**Triage:** Deep

**Research findings:**
- [HIGH] No current caching in codebase
- [HIGH] Slow endpoint: /api/products (500ms avg), rarely changes (good cache candidate)
- [HIGH] Redis is in docker-compose.yml but not connected
- [MEDIUM] /api/users (200ms avg) changes frequently (poor cache candidate)
- [LOW] go-redis library is the standard choice for Go + Redis

**Questions:**
1. **Cache target** — What should be cached?
   - **Product catalog** — 500ms response, rarely changes. Best cache candidate.
   - **User profiles** — 200ms, changes often. Less benefit, needs careful invalidation.
   - **Both** — Maximum performance gain, more invalidation complexity.

2. **Strategy** — Which caching approach?
   - **Redis with TTL** — Already in docker-compose. Standard for distributed systems.
   - **In-memory (groupcache)** — Simpler, no external dependency, but not shared across instances.

3. **TTL** — How long should cached data live?
   - **5 minutes** — Good for static data like product catalog.
   - **1 minute** — Balanced freshness vs. performance.
   - **30 seconds** — Conservative, minimal staleness.

**User answers:** Product catalog, Redis with TTL, 1 minute

**Enriched prompt:**
> **Task:** Add Redis caching for product catalog endpoint
> **Target:** /api/products handler, new cache middleware
> **Approach:** Redis with 1-minute TTL using go-redis. Connect to existing Redis in docker-compose.
> **Success criteria:** /api/products response time < 50ms on cache hit, cache invalidates after 60s

---

## Clear Prompts (Proceed Immediately)

### Example 4: Specific File and Action

**Prompt:** "Refactor getUserById in src/api/users.ts to use async/await instead of promises"
**Evaluation:** Target ✅, Action ✅, Criteria ✅, Context ✅ → **CLEAR**
**Decision:** Proceed immediately.

### Example 5: Specific Bug with Error

**Prompt:** "Fix the TypeError at line 145 in src/auth/login.ts where user.profile.name is undefined"
**Evaluation:** Target ✅, Action ✅, Criteria ✅, Context ✅ → **CLEAR**
**Decision:** Proceed immediately.

### Example 6: Detailed Feature Request

**Prompt:** "Add input validation to the registration endpoint using Zod. Validate: email (required, valid format), password (min 8 chars, must include number), username (3-20 chars, alphanumeric)."
**Evaluation:** Target ✅, Action ✅, Criteria ✅, Context ✅ → **CLEAR**
**Decision:** Proceed immediately.

---

## Context-Dependent Prompts

### Example 7: File Context Makes Clear

**Context:** User has src/components/LoginForm.tsx open
**Prompt:** "refactor this to use hooks"
**Evaluation:** Target ✅ (from file context), Action ✅, Criteria ✅ → **CLEAR**
**Decision:** Proceed immediately. "This" = the open file.

### Example 8: Conversation Provides Context

**Previous messages:** User discussed using Prisma, said "ok let's go with Prisma"
**Prompt:** "set it up"
**Evaluation:** Target ✅ (Prisma), Action ✅ (install/configure) → **CLEAR**
**Decision:** Proceed. Context from conversation.

### Example 9: Error Message Provides Context

**Previous message:** `Error: ECONNREFUSED at 127.0.0.1:5432`
**Prompt:** "fix this"
**Evaluation:** Target ✅ (PostgreSQL connection), Action ✅, Criteria ✅ → **CLEAR**
**Decision:** Proceed. Error is the full context.

---

## Bypass Prompts

**`* just add a comment`** → Strip `*`, execute without evaluation.
**`/commit`** → Pass to slash command system.
**`# always use strict mode`** → Pass to memory system.

---

## Decision Summary

| Signal | Action |
|--------|--------|
| All 4 criteria pass | Proceed immediately |
| 3/4 pass, gap filled by context | Proceed immediately |
| ≤2/4 pass | Invoke skill, research, ask questions |
| Bypass prefix detected | Pass through unchanged |
| User answers introduce new ambiguity | Ask max 1 follow-up, then assume and state |
