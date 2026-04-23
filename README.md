# OpenClaw Skills

OpenClaw-native skills for 7-agent architecture.

## Skills

### openclaw-agent-orchestration v1.0.0
OpenClaw-native multi-agent orchestration skill. Replaces generic `agent-team-orchestration` and `automation-workflows-0-1-0`.
- 7-Agent role map (main/coding/literature/writing/search/secretary/analyst)
- A2A-v1 protocol integration
- 4 standard workflow patterns
- Memory-aware routing with capability-registry.json

### self-improving v2.0.0
Auto-detect systematic failures and convert to memory rules.
- 6 precise trigger signals (P0/P1/P2)
- Fixed output template
- 5-tier memory system integration

### halthelobster-proactive-agent v4.0.0
Suggest next actions before user asks.
- 6 Chinese trigger keywords
- Pre-check routing rules and performance data
- Max 3 suggestions per trigger

## Removed Skills
- `automation-workflows-0-1-0` — Generic Zapier/Make/n8n automation, not applicable to OpenClaw
- `agent-team-orchestration` — Generic multi-agent template, not adapted to A2A-v1 protocol
- `delegate-task` — Duplicate of existing `sessions_send` mechanism

## Architecture

```
main (Orchestrator)
├── coding (Builder)
├── literature (Researcher)
├── writing (Writer)
├── search (Scout)
├── secretary (Ops)
└── analyst (Analyst)
```

All skills use OpenClaw 5-tier memory system:
L1 SOUL → L2 IDENTITY → L3 SYSTEM_COORDINATION → L4 LEARNINGS → L5 ARCHIVE