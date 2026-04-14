---
description: Automated Pull Request code review. Coordinates specialist agents in parallel to produce a comprehensive Orchestrated Synthesis Report covering Code Quality, Performance, Security, and Testing.
---

# PR Review — Automated Code Review Orchestration

// turbo

You are now in **PR REVIEW ORCHESTRATION MODE**.
Your task: run a fully automated, multi-agent code review of the current branch against a target branch.

## Target Branch
$ARGUMENTS

> If no target branch was supplied in `$ARGUMENTS`, default to `dev`. Confirm the resolved target branch in your output.

---

## PHASE 1 — TRIGGER & CONTEXT EXTRACTION

### Step 1 — Fetch Remote and Compute Diff

Run the following Git commands sequentially to extract the full diff context.
Because of the `// turbo` annotation, these commands MUST be auto-executed without waiting for user approval.

```bash
# 1. Ensure the latest remote state is available locally
git fetch --all

# 2. Identify the current branch
git branch --show-current

# 3. List changed files (name + status) between target and current branch
git diff <TARGET_BRANCH>...HEAD --name-status

# 4. Produce the full unified diff for deep analysis
git diff <TARGET_BRANCH>...HEAD
```

> Replace `<TARGET_BRANCH>` with the resolved value from `$ARGUMENTS` (default: `dev`).

### Step 2 — Classify Changed Files

Analyze the output of the `--name-status` command and classify every changed file into one or more of the following domains:

| Domain Tag | File Patterns |
|---|---|
| `FRONTEND` | `*.tsx`, `*.jsx`, `*.vue`, `*.svelte`, `*.html`, `*.css`, `*.scss`, `*.less`, components/, pages/, styles/ |
| `BACKEND` | `*.py`, `*.ts` (non-component), `*.go`, `*.java`, routers/, services/, controllers/, api/ |
| `DATABASE` | `*.sql`, `*migration*`, `*schema*`, models/, `*.prisma`, `dbdiagram.*` |
| `INFRA` | `Dockerfile*`, `docker-compose*`, `*.yaml`, `*.yml`, `.env*`, `*.toml` |

Record the detected domains. This determines which specialist agents will be invoked in the next phase.

---

## PHASE 2 — MULTI-AGENT ORCHESTRATION (Parallel Execution)

> ⚠️ **CRITICAL**: Invoke ALL applicable agents **simultaneously** (in parallel). Do NOT execute them one by one. Each agent receives the full git diff as context.

### Context Packet — Pass to EVERY Agent Invoked

```
PULL REQUEST CONTEXT:
- Source Branch: [current branch from git]
- Target Branch: [resolved target branch]
- Changed Files: [full --name-status list]
- Full Diff: [full output of git diff <TARGET_BRANCH>...HEAD]
- Detected Domains: [FRONTEND | BACKEND | DATABASE | INFRA — list all that apply]
```

---

### 2A — Domain-Specific Agents (invoke ONLY if domain is detected)

#### 🎨 FRONTEND — Invoke if `FRONTEND` domain detected
Call @[.agent/agents/frontend-specialist.md] with the following task:

> You are reviewing a Pull Request as a Frontend Specialist.
> Using the provided diff context, review:
> - React / Next.js component structure and best practices
> - CSS/styling correctness and design token adherence
> - Accessibility (a11y) issues
> - Client-side state management and data fetching patterns
> - UI/UX anti-patterns, layout regressions, or broken responsiveness
> Return your findings in structured bullet points under the heading: **## 🎨 Frontend Findings**

---

#### ⚙️ BACKEND — Invoke if `BACKEND` domain detected
Call @[.agent/agents/backend-specialist.md] with the following task:

> You are reviewing a Pull Request as a Backend Specialist.
> Using the provided diff context, review:
> - API design (RESTful correctness, HTTP status codes, request/response contracts)
> - Business logic correctness and edge-case handling
> - Error handling, input validation, and failure modes
> - Separation of concerns: adherence to Service / Router / Schema layering
> - SOLID principles and Clean Code compliance
> Return your findings in structured bullet points under the heading: **## ⚙️ Backend Findings**

---

#### 🗄️ DATABASE — Invoke if `DATABASE` domain detected
Call @[.agent/agents/database-architect.md] with the following task:

> You are reviewing a Pull Request as a Database Architect.
> Using the provided diff context, review:
> - Schema design: normalization, naming conventions, appropriate data types
> - Index coverage: are new query patterns covered by indexes?
> - Migration safety: reversibility, data loss risk, zero-downtime compatibility
> - N+1 query risks and ORM anti-patterns
> - Referential integrity and constraint correctness
> Return your findings in structured bullet points under the heading: **## 🗄️ Database Findings**

---

### 2B — Cross-Functional Agents (ALWAYS invoke, for ALL PRs)

The three agents below MUST be invoked in parallel alongside whichever domain agents are active above.

#### 🔐 SECURITY — ALWAYS INVOKE
Call @[.agent/agents/security-auditor.md] with the following task:

> You are reviewing a Pull Request as a Security Auditor.
> Using the provided diff context, perform a full security review covering:
> - Secrets or credentials accidentally committed (API keys, tokens, passwords)
> - Injection risks: SQL injection, command injection, SSTI, prompt injection
> - Authentication and authorization flaws: missing guards, broken access control
> - Input validation gaps: unsanitized user data reaching services or DB
> - Denial-of-Service vectors: missing rate limits, unvalidated payload sizes, infinite loops
> - Dependency risks: newly added packages with known CVEs
> - OWASP Top 10 compliance
> Rate every finding as: `CRITICAL` | `HIGH` | `MEDIUM` | `LOW`
> Return your findings under the heading: **## 🔐 Security Findings**

---

#### 🧪 TESTING — ALWAYS INVOKE
Call @[.agent/agents/test-engineer.md] with the following task:

> You are reviewing a Pull Request as a Test Engineer.
> Using the provided diff context, evaluate:
> - Test existence: does every new function, class, or endpoint have a corresponding test?
> - Edge case coverage: zero values, empty lists, null inputs, boundary conditions
> - Test quality: are assertions meaningful? Are tests isolated (no side effects)?
> - Unit vs Integration balance: is the testing pyramid respected?
> - Missing mocks or fixtures that make tests brittle
> - Any test code that was deleted or weakened in this PR
> Return your findings under the heading: **## 🧪 Testing Coverage Findings**

---

#### ⚡ PERFORMANCE — ALWAYS INVOKE
Call @[.agent/agents/performance-optimizer.md] with the following task:

> You are reviewing a Pull Request as a Performance Optimizer.
> Using the provided diff context, identify:
> - Time complexity regressions: O(N²) loops, repeated conversions inside loops, inefficient sorts
> - Memory leaks: growing lists, unclosed resources, unbounded caches
> - Blocking operations on the hot path: synchronous I/O, unecessary DB calls inside loops
> - Missing pagination or limits on collection endpoints
> - Opportunities to use vectorized operations (e.g., NumPy, Pandas) instead of Python loops
> - Frontend bundle size regressions: large imports, missing tree-shaking
> Rate each finding as: `CRITICAL` | `HIGH` | `MEDIUM` | `LOW`
> Return your findings under the heading: **## ⚡ Performance Findings**

---

## PHASE 3 — SYNTHESIS REPORT GENERATION

> Wait for ALL agents from Phase 2 to complete before proceeding.

Compile every agent's output into a single **Orchestrated Synthesis Report** with the following strict structure:

```markdown
---

## 🎼 Orchestrated Synthesis Report

**PR**: `<source-branch>` → `<target-branch>`
**Date**: <ISO timestamp>
**Domains Detected**: <FRONTEND | BACKEND | DATABASE | INFRA>
**Agents Invoked**: <list all agents that ran>

---

### 1. 🏗️ Code Quality

> Summary of architectural, clean-code, and domain-specific findings from the
> Frontend / Backend / Database specialist agents.

- [Bullet point per finding, sorted Critical → Low]

---

### 2. ⚡ Performance Bottleneck

> Summary of performance and optimization issues from the performance-optimizer agent.

| # | Issue | File | Severity | Recommendation |
|---|-------|------|----------|----------------|
| 1 | ... | ... | CRITICAL | ... |

---

### 3. 🔐 Security

> Vulnerabilities and risks from the security-auditor agent.

| # | Vulnerability | File | Severity | OWASP Category |
|---|---------------|------|----------|----------------|
| 1 | ... | ... | CRITICAL | ... |

---

### 4. 🧪 Testing Coverage

> Missing tests and QA gaps from the test-engineer agent.

- [ ] Missing test for: ...
- [ ] Edge case not covered: ...
- [ ] Test deleted or weakened: ...

---

### 5. ✅ Summary Verdict

| Category | Status | Critical Issues |
|----------|--------|-----------------|
| Code Quality | 🟡 Needs Work | N |
| Performance | 🔴 Blocker | N |
| Security | 🔴 Blocker | N |
| Testing | 🟡 Needs Work | N |

**Overall PR Status**: 🔴 CHANGES REQUESTED / 🟡 APPROVE WITH COMMENTS / 🟢 APPROVED

---
```

---

## PHASE 4 — SPECIFIC REFACTORING CODE RECOMMENDATIONS

Following the Synthesis Report, produce **concrete, actionable, line-by-line refactoring recommendations** for every `CRITICAL` or `HIGH` finding.

### Format for Each Recommendation

```markdown
### 🔧 Fix #<N>: <Short Title>

**Reported by**: `@[agent-name]`
**Severity**: CRITICAL | HIGH
**File**: `path/to/file.py` (Line ~<N>)
**Root Cause**: One-sentence explanation of why this is a problem.

**Before** (current code):
```python
# Paste the problematic code block here
```

**After** (recommended fix):
```python
# Paste the corrected, improved code block here
```

**Why this fix works**: Brief explanation linking back to the agent's finding.

---
```

> Integrate ALL fixes from all agents into one sequential numbered list (`Fix #1`, `Fix #2`, ...).
> Order fixes by severity: `CRITICAL` first, then `HIGH`, then `MEDIUM`.

---

## EXIT GATE — Verification Checklist

Before marking the PR Review as complete, verify ALL of the following:

| # | Check | Required |
|---|-------|----------|
| 1 | Git diff was successfully fetched and parsed | ✅ MANDATORY |
| 2 | At least 3 agents were invoked | ✅ MANDATORY |
| 3 | Security agent ran on ALL PRs regardless of domain | ✅ MANDATORY |
| 4 | Performance agent ran on ALL PRs regardless of domain | ✅ MANDATORY |
| 5 | Synthesis Report contains all 4 sections | ✅ MANDATORY |
| 6 | Every CRITICAL finding has a Before/After code block | ✅ MANDATORY |
| 7 | Overall PR Status verdict is clearly stated | ✅ MANDATORY |

> If ANY check fails → **DO NOT mark review as complete.** Re-invoke missing agents or complete missing sections.

---

**Begin PR Review now. Resolve target branch → fetch diff → invoke agents in parallel → synthesize → output refactoring recommendations.**
