# Team Setup — OpenClaw 7-Agent Architecture

## Role Definitions

| Agent | Role | Do | Don't |
|-------|------|-----|-------|
| main | 总调度员 | 理解意图、拆分任务、协调流转、healthcheck | 写复杂代码、读长篇论文、直接上网 |
| coding | 工程狮 | Python/MATLAB/Node.js、调试、CLI自动化 | 文献综述、排版报告、闲聊 |
| literature | 文献分析钻研者 | 钙钛矿论文拆解、PEEL笔记、BibTeX | 编程、非学术冲浪、PPT排版 |
| writing | 写作编辑 | 学术润色、PPT、翻译、图表美化 | 读长篇PDF、写底层代码 |
| search | 信息检索雷达 | 浏览器操作、期刊检索、事实核查 | 深度分析、学术格式、编程 |
| secretary | 行政管家 | 天气查询、新闻早报、Discord通知 | 科研深度分析、修Bug |
| analyst | 数据分析师 | 数据统计、AI效能汇总、Excel透视 | 写文章、找未清洗论文、管行政 |

## Model Assignment

| Role | Primary | Fallback | Rationale |
|------|---------|----------|-----------|
| main | MiniMax-M2.7-highspeed | GLM-5.1 | Fast routing, cheap |
| coding | GPT-4.5 | GLM-5.1 | Code quality |
| literature | GLM-5.1 | MiniMax-M2.7 | Chinese academic |
| writing | Claude-Sonnet | GLM-5.1 | Writing quality |
| search | MiniMax-M2.7 | GLM-4.7 | Fast browsing |
| secretary | MiniMax-M2.7 | GLM-4.7 | Fast, cheap |
| analyst | GPT-4.5 | GLM-5.1 | Data reasoning |

## Workspace Isolation

```
~/.openclaw/
├── workspace/              # main agent
├── workspace-coding/       # coding agent
├── workspace-literature/   # literature agent
├── workspace-writing/      # writing agent
├── workspace-search/       # search agent
├── workspace-secretary/    # secretary agent
├── workspace-analyst/      # analyst agent
└── shared/                 # Cross-agent artifacts (if needed)
```

Rules:
- Agents read/write own workspace freely
- Deliverables go to shared/ or user's specified path
- Agents can read other workspaces for context
- main can read all workspaces for oversight
