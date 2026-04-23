# Task Lifecycle

## States

```
Inbox → Assigned → In Progress → Review → Done | Failed
```

| State | Meaning | Owner |
|-------|---------|-------|
| Inbox | New task, unassigned | main |
| Assigned | Agent selected, not started | main |
| In Progress | Agent working | Specialist |
| Review | Awaiting verification | main / Reviewer |
| Done | Verified and delivered | main |
| Failed | Abandoned with reason | main |

## Transitions

**main transitions:**
- Inbox → Assigned: picks agent using capability-registry.json
- Assigned → In Progress: spawns agent with TASK_ASSIGN
- Review → Done: accepts deliverable
- Any → Failed: with documented reason

**Specialists transition:**
- In Progress → Review: submits with TASK_RESULT

**Reviewers transition:**
- Review → In Progress: returns with feedback
- Review → Done: approves

## Comment Format

```
[Agent] [Action]: [Details]
```

Examples:
```
[literature] Starting: Analyzing WBG-PSC paper #3
[coding] Blocked: Need VASP output format sample
[writing] Handoff: PPT draft at C:\work\draft.pptx, 12 slides
[main] Done: Delivered integrated report
```

## Decision Logging

Architecture decisions during execution:

```markdown
# Decision: [Title]
**Date:** YYYY-MM-DD
**Agent:** [main/coding/literature/...]
**Status:** Proposed | Accepted | Rejected
**Task:** [taskId]

## Context
Why this came up.

## Decision
What was decided.

## Rationale
Why this option was chosen.

## Consequences
What this affects.
```

Store in: `workspace/memory/decisions/` or agent-specific memory
