---
name: tech-research
description: 技术调研编排技能。作为编排器按顺序调用子 skill pipeline 完成完整技术调研。当用户想要调研、学习新技术时触发。触发词：调研、学习新技术、技术选型、技术对比、了解xxx、xxx是什么、xxx怎么用、深度调研。
---

# 技术调研技能（编排器）

本技能是一个 **编排器（Orchestrator）**，负责根据用户意图调度子 skill pipeline，不直接执行搜索或生成报告。

## 可调度的子 Skill

| 子 Skill | 用途 | 产出文件 |
|----------|------|----------|
| `tech-discover` | 多源检索、信息收集、生成技术发现报告 | `<tech-name>-research/01-discovery.md` |
| `tech-analyze` | 深度分析架构、原理、竞品对比 | `<tech-name>-research/02-analysis.md` |
| `tech-tutorial` | 生成从零到一的完整学习教程 | `<tech-name>-research/03-tutorial.md` |
| `tech-review` | 审阅全部文档、修复问题、质量评分 | `<tech-name>-research/04-review.md` |

## 调度策略

根据用户意图，选择对应的执行模式：

### 模式 1：快速概览

**触发词**：
- "帮我快速了解 xxx"
- "xxx 是什么"
- "简单介绍下 xxx"
- "快速了解 xxx"
- "xxx 概览"

**执行**：调用 `tech-discover`

```
使用 Skill 工具，参数如下：
skill: tech-discover
args: xxx（用户要了解的技术名称）
```

产出：`<tech-name>-research/01-discovery.md`

### 模式 2：深度分析

**触发词**：
- "深度分析 xxx"
- "分析 xxx 的原理"
- "xxx 架构"
- "对比 xxx 和 yyy"

**前置条件**：需要先有 `01-discovery.md`。如果没有，先调用 `tech-discover`。

**执行**：调用 `tech-analyze`

```
使用 Skill 工具，参数如下：
skill: tech-analyze
args: xxx（用户要分析的技术名称）
```

产出：`<tech-name>-research/02-analysis.md`

### 模式 3：生成教程

**触发词**：
- "出一份 xxx 的教程"
- "生成 xxx 学习教程"
- "xxx 教程"
- "学习 xxx"

**前置条件**：需要先有 `01-discovery.md` 和 `02-analysis.md`。如果没有，先按顺序补调 `tech-discover` 和 `tech-analyze`。

**执行**：调用 `tech-tutorial`

```
使用 Skill 工具，参数如下：
skill: tech-tutorial
args: xxx（用户要学习的技术名称）
```

产出：`<tech-name>-research/03-tutorial.md`

### 模式 4：审阅报告

**触发词**：
- "审阅 xxx 的调研报告"
- "检查调研质量"
- "review xxx 调研"

**前置条件**：需要先有 `01-discovery.md`、`02-analysis.md`、`03-tutorial.md`。

**执行**：调用 `tech-review`

```
使用 Skill 工具，参数如下：
skill: tech-review
args: xxx（用户要审阅的技术名称）
```

产出：`<tech-name>-research/04-review.md`

### 模式 5：完整调研（Full Pipeline）

**触发词**：
- "深度调研 xxx"
- "全面调研 xxx"
- "出一分 xxx 的调研报告"
- "详细了解 xxx"
- "research xxx"

**执行**：按以下顺序 **依次调用** 4 个子 skill，每个完成后再调用下一个：

#### Step 1: 技术发现

```
使用 Skill 工具，参数如下：
skill: tech-discover
args: xxx（用户要调研的技术名称）
```

等待完成后确认 `<tech-name>-research/01-discovery.md` 已生成。

#### Step 2: 深度分析

```
使用 Skill 工具，参数如下：
skill: tech-analyze
args: xxx
```

等待完成后确认 `<tech-name>-research/02-analysis.md` 已生成。

#### Step 3: 生成教程

```
使用 Skill 工具，参数如下：
skill: tech-tutorial
args: xxx
```

等待完成后确认 `<tech-name>-research/03-tutorial.md` 已生成。

#### Step 4: 质量审阅

```
使用 Skill 工具，参数如下：
skill: tech-review
args: xxx
```

等待完成后确认 `<tech-name>-research/04-review.md` 已生成。

#### 完成汇总

全部步骤完成后，向用户输出：
1. 所有生成文件的路径列表
2. 质量评分（来自 04-review.md）
3. 关键发现摘要（2-3 句话）

## 重要规则

1. **必须使用 Skill 工具调用子 skill**，禁止跳过子 skill 直接用底层工具（WebSearch、Agent 等）完成任务。本技能只做调度，不做执行。
2. **必须按顺序调用**，后一步依赖前一步的产出文件。不可并行调用。
3. **如果前置文件缺失，必须先补调前面的子 skill**。例如用户要求"分析 xxx"但没有 discovery 报告，先调 `tech-discover` 再调 `tech-analyze`。
4. **每个子 skill 完成后，确认产出文件存在再继续下一步**。如果子 skill 执行失败，向用户报告错误并询问是否重试。
