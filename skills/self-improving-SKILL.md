---
name: Self-Improving Agent
slug: self-improving
version: 2.0.0
homepage: https://clawic.com/skills/self-improving
description: "Auto-detect systematic failures and convert them into memory rules. Trigger when: (1) same error repeats twice in a row; (2) search results show noise instead of target for 2+ queries; (3) workflow needs 3+ corrections; (4) agent communication shows BLOCKED or timeout; (5) user explicitly asks to optimize memory or review rules. MUST write findings to agents/[agent]/memory/ with verification_count."
changelog: "v2.0.0: Refactored from philosophy to actionable triggers. Bound to OpenClaw 5-tier memory system."
metadata: {"clawdbot":{"emoji":"🧠","requires":{"bins":[]},"os":["linux","darwin","win32"],"configPaths":["~/.openclaw/agents/"],"configPaths.optional":["./AGENTS.md","./SOUL.md","./HEARTBEAT.md"]}}
---

## When to Use

This skill triggers on **specific failure signals**, not vague introspection.

| Signal | Example | Priority |
|--------|---------|----------|
| **Same error repeats twice** | `memory_search` fails with same error 2x | P0 |
| **Search returns noise** | Query "VASP" returns irrelevant results 2x | P1 |
| **Workflow needs 3+ corrections** | User corrects routing decision 3 times | P1 |
| **Agent communication BLOCKED** | `sessions_send` timeout or DRI conflict | P1 |
| **User explicitly asks** | "优化记忆","review rules","检查流程" | P2 |
| **New pattern validated 3x** | Same workaround works 3 times | P2 |

**DO NOT trigger on:**
- One-time instructions ("现在做X")
- Context-specific tweaks ("在这个文件里...")
- Silence or absence of feedback

## Architecture

Uses OpenClaw 5-tier memory system. All outputs write to `~/.openclaw/agents/[agent]/memory/`.

```
~/.openclaw/agents/[agent]/memory/
├── SYSTEM_COORDINATION.md   # L3: Routing rules, ≥3x verified
├── USER_PREFERENCES.md      # L4: User preferences
├── AGENT_PERFORMANCE.md     # L4: Timeout rates, error patterns
├── LEARNINGS.md             # L4: Daily lessons (unverified)
└── corrections.md           # L4: Explicit corrections
```

## Trigger Detection

### P0: Repeated Failures
```
IF same tool/API fails twice with same error:
  1. Read agents/[current]/memory/AGENT_PERFORMANCE.md
  2. Check if error pattern already recorded
  3. IF not recorded → generate self-improve report
  4. Write to LEARNINGS.md with verification_count: 0
```

### P1: Search Quality Issues
```
IF memory_search returns irrelevant results for 2+ related queries:
  1. Check if seed memories exist for that topic
  2. IF seeds exist but not hit → check embedding/scope issue
  3. IF no seeds → generate seed import JSON
  4. Write search strategy fix to SYSTEM_COORDINATION.md
```

### P1: Workflow Rework
```
IF user corrects same workflow 3+ times:
  1. Read current routing rules in SYSTEM_COORDINATION.md
  2. Identify where decision tree failed
  3. Propose rule modification
  4. Ask user confirm before writing
```

## Output Template

Every trigger produces this exact format:

```markdown
## Self-Improve Report — [timestamp]

### Trigger Signal
[Which of the 6 signals fired, with evidence]

### Current Problem
[One sentence: what keeps failing/what's wrong]

### Root Cause
[Technical reason: tool limit? wrong rule? missing context?]

### Proposed Fix
- **Target file**: `agents/[agent]/memory/[file].md`
- **Category**: decision / fact / preference
- **Change type**: add / modify / delete
- **Exact content**: [what to write]

### Verification Plan
[How to confirm this fix works next time]

### User Confirmation
- [ ] Auto-apply (P0 failures only)
- [ ] Ask before applying (P1/P2)
- [ ] Discard (false positive)
```

## Memory Integration

### Writing Rules

| Content | Destination | Category | Verification |
|---------|-------------|----------|--------------|
| Routing rule fix | SYSTEM_COORDINATION.md | decision | 3x success → promote |
| User preference | USER_PREFERENCES.md | preference | immediate |
| Error pattern | AGENT_PERFORMANCE.md | fact | 2x recurrence |
| New lesson | LEARNINGS.md | fact | 0, wait for validation |
| Explicit correction | corrections.md | — | user-triggered |

### Promotion Rules
- `verification_count: 0` → newly written, tentative
- `verification_count: 1` → observed once, warming
- `verification_count: 2` → observed twice, likely valid
- `verification_count: 3` → promote to L3 (SYSTEM_COORDINATION.md)
- Unused 30 days → demote to L5 (archive)

## Examples

**Example 1: Repeated memory_search failure**
```
Trigger: memory_search returns empty for "EDADI" twice
Problem: Literature agent's material parameters not retrievable
Root Cause: No seed memory for EDADI in agent:literature scope
Fix: Create seed memory in tmp/memory_seed_literature.json, import
Verification: Search "EDADI" again, confirm top result is relevant
```

**Example 2: Workflow rework**
```
Trigger: User corrected routing from search→literature 3 times for paper queries
Problem: Search agent handling paper interpretation instead of literature
Root Cause: Decision tree Q3→Q4 path not explicit for "paper" keywords
Fix: Add explicit "paper/论文" → literature routing in SYSTEM_COORDINATION.md
Verification: Next paper query routes to literature without correction
```

**Example 3: Search noise**
```
Trigger: Query "A2A protocol" returns old TASK_ASSIGN logs instead of rules
Problem: Historical noise outranks structured seeds
Root Cause: Old entries not cleaned, seeds not dense enough
Fix: Delete noisy old entries, add more specific seed variants
Verification: Search again, confirm seeds rank in top 3
```

## Verification Checklist

Before writing any fix:
- [ ] Is this actually a pattern? (not one-time)
- [ ] Is the root cause identified? (not symptom-only)
- [ ] Is the target file correct? (matches content type)
- [ ] Does this contradict existing rules? (check first)
- [ ] Can I verify this works? (has verification plan)

## Anti-Patterns

- ❌ Don't log one-time events as patterns
- ❌ Don't modify L3 rules without 3x verification
- ❌ Don't infer from silence
- ❌ Don't write to `~/self-improving/` — use `~/.openclaw/agents/[agent]/memory/`
- ❌ Don't auto-apply P1/P2 without user confirm

## Scope

**ONLY:**
- Detects systematic failures
- Writes to OpenClaw memory system
- Reads AGENT_PERFORMANCE.md and SYSTEM_COORDINATION.md
- Asks user confirm for P1/P2 changes

**NEVER:**
- Accesses external APIs
- Modifies its own SKILL.md
- Deletes without archiving
- Infers from silence
- Writes to old `~/self-improving/` path
