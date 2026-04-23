---
name: openclaw-agent-orchestration
version: 1.0.0
description: "OpenClaw-native multi-agent orchestration for 7-agent architecture. Trigger when: (1) task involves 2+ agents; (2) need cross-agent handoff; (3) setting up workflow between main/coding/literature/writing/search/secretary/analyst. MUST follow A2A-v1 protocol, use capability-registry.json for routing, and query memory before delegation."
---

# OpenClaw Agent Orchestration

Production playbook for coordinating 7 specialized agents within OpenClaw.

## When to Use

| Scenario | Trigger |
|----------|---------|
| Task spans 2+ domains | "写个爬虫分析论文" (coding + literature) |
| Cross-agent handoff needed | literature finishes → writing polishes |
| Workflow setup | Establish recurring collaboration pattern |
| Main agent unsure routing | Query capability-registry before deciding |
| Agent returns BLOCKED | Escalate to correct specialist |

**DO NOT use for:**
- Single-agent tasks (direct assignment is faster)
- One-off questions (use sessions_send directly)
- Tasks already covered by existing cron jobs

---

## 7-Agent Role Map

```
main        — 总调度员 | Orchestrator (judgment, routing, integration)
coding      — 工程狮 | Builder (code, data, scripts, CLI)
literature  — 文献分析钻研者 | Researcher (papers, PEEL, BibTeX)
writing     — 写作编辑 | Writer (polish, PPT, visualization)
search      — 信息检索雷达 | Scout (browser, news, fact-check)
secretary   — 行政管家 | Ops (weather, calendar, Discord, daily brief)
analyst     — 数据分析师 | Analyst (statistics, Excel, performance)
```

→ Read `memory/capability-registry.json` before every routing decision.

---

## Routing Rules (from SYSTEM_COORDINATION.md)

### Keyword → Agent Mapping

| Keywords | Target | Avoid |
|----------|--------|-------|
| paper/论文/文献/综述/PEEL | literature | search |
| code/编程/脚本/实现/debug | coding | literature/writing |
| 搜索/查找/最新/新闻/verify | search | literature |
| 写/润色/文档/PPT/图/翻译 | writing | coding |
| 日程/提醒/天气/安排/通知 | secretary | main |
| 分析/数据/统计/健康/Excel | analyst | coding |

### Multi-Agent Patterns

**Pattern A: Sequential Chain**
```
main → literature (analyze paper) → writing (polish) → main (deliver)
```

**Pattern B: Parallel Research**
```
main → search (find sources)
     → literature (check related work)
     → analyst (get data)
     → main (merge results)
```

**Pattern C: Build-Review Loop**
```
main → coding (implement) → coding (self-test) → main (review) → done
Or:
main → coding (implement) → analyst (validate data) → main (ship)
```

---

## A2A-v1 Protocol (Mandatory)

### Task Assignment

```yaml
[TASK_ASSIGN]
protocolVersion: a2a-v1
taskId: <YYYYMMDDHHmmss-rand4>
fromAgent: main
toAgent: <target-agent-id>
callbackSessionKey: agent:main:main
notifyTarget:
  channel: discord
  to: <频道 ID>   # notification only
objective: <clear, specific goal>
scope: <boundaries: what to do / what NOT to do>
deliverables:
  - <artifact path with absolute path>
context:
  - <relevant memory or prior taskId>
```

### Task Result

```yaml
[TASK_RESULT]
protocolVersion: a2a-v1
taskId: <same as request>
fromAgent: <executing agent>
toAgent: main
status: <accepted|blocked|done|failed>
summary: <one sentence conclusion>
keyFindings:
  - <point 1>
  - <point 2>
artifactManifest:
  - path: <absolute path>
    exists: true
    bytes: <file size>
risks:
  - <uncertainty or limitation>
nextStep: <suggested follow-up>
```

### Routing Rules

1. **Addressing**: Use `sessionKey: "agent:<id>:main"` — NEVER use `label`
2. **Async**: `timeoutSeconds=0` for fire-and-forget
3. **Serial deps**: Wait for `status: done` before next step
4. **Parallel deps**: Spawn multiple, collect all results
5. **Cross-chain**: `callbackSessionKey` always points to main

---

## Pre-Delegation Checklist

Before assigning ANY task:

1. **Query capability registry**:
   ```json
   // Read memory/capability-registry.json
   // Confirm target agent's "do" list covers the task
   // Confirm task not in target's "dont" list
   ```

2. **Search memory for pitfalls**:
   ```
   memory_search("[task topic] routing")
   // Check SYSTEM_COORDINATION.md for known issues
   // Check AGENT_PERFORMANCE.md for timeout rates
   ```

3. **Check agent health** (for non-urgent tasks):
   ```sql
   -- Query system_monitor.db
   SELECT type, title FROM issues 
   WHERE status = 'open' AND title LIKE '%<agent>%'
   ```

4. **Verify deliverable paths**:
   - Absolute path required
   - Parent directory must exist
   - Agent must have write permission

---

## Task State Tracking

Track cross-agent tasks in `system_monitor.db` or shared memory:

```
Inbox → Assigned → In Progress → Review → Done | Failed
```

| State | Owner | Transition Trigger |
|-------|-------|-------------------|
| Inbox | main | Task received, not yet routed |
| Assigned | main | Target agent selected |
| In Progress | specialist | `status: accepted` received |
| Review | main | `status: done` received |
| Done | main | Output verified and delivered |
| Failed | main | `status: failed` or timeout |

**Rules:**
- main owns all state transitions
- Every transition gets a comment (who, what, why)
- Failed is valid — document reason and move on

---

## Handoff Quality Standards

Every handoff MUST include:

1. **What was done** — summary of changes
2. **Where artifacts are** — exact file paths
3. **How to verify** — test commands or acceptance criteria
4. **Known issues** — anything incomplete or risky
5. **What's next** — clear next action

Bad: *"Done, check the files."*
Good: *"Built auth module at C:\workspace\artifacts\auth\. Run npm test auth. Known: rate limit not implemented. Next: reviewer checks edge cases."*

---

## Quality Gates

### Builder → Reviewer Handoff
- Builder produces artifact → Reviewer checks → main ships or returns
- Builder does NOT review own work
- Reviewer uses different model/perspective

### Main Integration Rules
1. Review result completeness (not just forward)
2. Challenge if results conflict with verified facts
3. Format for human readability
4. Task sessions auto-prune after 7 days

---

## Common Pitfalls

| Pitfall | Prevention |
|---------|-----------|
| Wrong agent routing | Check capability-registry.json first |
| Using `label` instead of `sessionKey` | Enforce `agent:<id>:main` format |
| Missing deliverable paths | Require absolute path in TASK_ASSIGN |
| No review step | Every non-trivial artifact gets review |
| Silent agents >10min | Assume blocked, check status |
| Orchestrator doing build work | Route execution to specialists |
| Cross-chain callback to wrong agent | callbackSessionKey always = main |

---

## Memory Integration

After workflow completes:

1. **If successful**: Add to `SYSTEM_COORDINATION.md` as verified pattern
2. **If failed**: Add to `AGENT_PERFORMANCE.md` with error pattern
3. **If corrected**: Log correction in `corrections.md`

Promotion rules:
- 1x success → LEARNINGS.md (verification_count: 1)
- 3x success → SYSTEM_COORDINATION.md (verified pattern)
- 30 days unused → archive

---

## Scope

**ONLY:**
- Routes tasks among 7 OpenClaw agents
- Uses A2A-v1 protocol with sessions_send
- Reads capability-registry.json and memory system
- Tracks task states

**NEVER:**
- Spawns arbitrary subprocesses
- Downloads external skills without verification
- Modifies agent SOUL.md files
- Bypasses memory system for routing decisions
