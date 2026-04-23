# Patterns — Proven Multi-Agent Workflows

## Pattern 1: Paper Analysis Pipeline

```
User: "分析这篇钙钛矿论文"

main → literature:
  objective: 深度拆解论文结构、提取实验参数
  deliverables: [PEEL笔记, 参数表]
  
literature → main: status done
  keyFindings: [论文要点, VASP参数, 稳定性数据]

main → writing (optional):
  objective: 将PEEL笔记转为PPT大纲
  deliverables: [PPT大纲]

writing → main: status done
main → user: 整合输出
```

## Pattern 2: Research → Code → Analysis

```
User: "找最新WBG-PSC文献并分析趋势"

main → search:
  objective: 搜索2025-2026 WBG-PSC最新进展
  deliverables: [文献列表, 关键数据]

main → literature (parallel):
  objective: 分析search找到的top 5论文
  deliverables: [PEEL笔记]

search & literature → main

main → coding:
  objective: 从PEEL笔记提取数据做趋势图
  deliverables: [Python脚本, 图表]

coding → main

main → analyst (optional):
  objective: 统计效能数据
  deliverables: [数据透视表]

main → user: 综合报告
```

## Pattern 3: Code Review Loop

```
User: "写个VASP数据处理脚本"

main → coding:
  objective: 实现VASP数据处理
  deliverables: [脚本文件, 测试用例]

coding → main: status done

main → coding (review request):
  objective: 自查代码质量
  scope: 检查错误处理、输入验证、边界情况

coding → main: status done

main → analyst (optional):
  objective: 验证输出数据准确性
  deliverables: [数据校验报告]

main → user: 交付
```

## Pattern 4: Daily Brief Automation

```
Cron trigger: 08:00 daily

secretary → search:
  objective: 获取 overnight AI/科研新闻
  
search → secretary: status done

secretary → analyst:
  objective: 整理昨日agent运行状态
  
analyst → secretary: status done

secretary → writing:
  objective: 生成格式化的日报
  
writing → secretary: status done

secretary → Discord: 推送日报
```

## Escalation Pattern

```
Agent hits blocker:
1. Comment: "Blocked: [specific problem]"
2. Continue other work if possible
3. main sees blocker, decides:
   a. Resolve directly
   b. Reassign to more capable agent
   c. Escalate to human
   d. Deprioritize
4. main comments decision
```

Escalation triggers:
- Missing access/credentials
- Ambiguous requirements needing product decisions
- Technical blocker outside expertise
- Scope exceeded by 2x+
