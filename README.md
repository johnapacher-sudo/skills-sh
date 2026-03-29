# Skills SH

Claude Code Agent 自定义技能集合 -- 技术深度调研工具

## 简介

Skills SH 是一套面向 Claude Code Agent 的多技能流水线工具，专注于技术深度调研。通过将调研过程拆分为发现、分析、教程、审阅四个阶段，配合总调度技能，实现从"了解一项技术"到"输出高质量调研报告"的全流程自动化。

无论是快速评估一个新框架，还是对多项技术进行深度对比分析，亦或是生成可落地的硬核教程，这套技能流水线都能系统化地完成。

## Skill 列表

| Skill | 功能 | 触发词 |
|-------|------|--------|
| tech-research | 总调度，一键全流程调研 | "调研 xxx"、"深度研究 xxx" |
| tech-discover | 技术发现与信息检索 | "快速了解 xxx"、"xxx 是什么" |
| tech-analyze | 深度分析与对比 | "深度分析 xxx"、"对比 xxx 和 yyy" |
| tech-tutorial | 硬核教程生成 | "出一份 xxx 教程"、"学习 xxx" |
| tech-review | 内容质量审阅 | "审阅 xxx 调研"、"检查调研质量" |

## 流水线架构

整套工具采用分阶段流水线设计：

```
tech-discover → tech-analyze → tech-tutorial → tech-review
```

- **tech-discover**：信息发现与初步评估，快速了解技术全貌
- **tech-analyze**：深度技术分析，包括架构、生态、对比等维度
- **tech-tutorial**：基于分析结果生成结构化的硬核教程
- **tech-review**：对产出内容进行质量审阅，确保完整性和准确性

**tech-research** 作为总调度器，可一键串联以上四个阶段，自动完成全流程调研。你也可以单独调用某个阶段的技能，按需使用。

## 项目结构

```
skills-sh/
├── skills/
│   ├── tech-discover/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── output-template.md
│   ├── tech-analyze/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── output-template.md
│   ├── tech-tutorial/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── output-template.md
│   ├── tech-review/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── review-checklist.md
│   └── tech-research/
│       └── SKILL.md
├── docs/
│   └── deep-research/
├── LICENSE
└── README.md
```

## 使用方法

### 安装

将 skills 目录复制到 Claude Code 的技能目录：

```bash
# 克隆仓库
git clone https://github.com/johnapacher-sudo/skills-sh.git

# 复制所有技能到本地 skills 目录
cp -r skills-sh/skills/* ~/.claude/skills/
```

### 触发使用

安装完成后，在 Claude Code 中直接使用触发词即可：

- **全流程调研**：`调研 React Server Components` 或 `深度研究 Rust vs Go`
- **单独阶段**：`快速了解 WebAssembly`、`深度分析 Next.js 15`、`出一份 Kubernetes 教程`、`审阅 React 调研质量`

## Skill 开发

每个 Skill 是一个独立目录，核心文件为 `SKILL.md`，采用 YAML frontmatter + Markdown body 格式：

```
skill-name/
├── SKILL.md              # 技能定义文件
└── references/           # 参考模板（可选）
    └── template.md
```

### SKILL.md 格式

```markdown
---
name: skill-name
description: 技能描述
triggers:
  - 触发词1
  - 触发词2
---

# Skill 标题

技能的具体指令和逻辑...

## 步骤

1. 第一步
2. 第二步

## 输出格式

期望的输出格式定义...
```

- **YAML frontmatter**：定义技能名称、描述、触发词等元信息
- **Markdown body**：编写技能的完整执行指令和流程
- **references/**：存放输出模板、检查清单等参考文件

## 输出示例

使用 tech-research 进行全流程调研后，会生成以下文件结构：

```
research/<tech-name>/
├── 01-discovery.md       # 技术发现报告
├── 02-analysis.md        # 深度分析报告
├── 03-tutorial.md        # 结构化教程
└── 04-review.md          # 质量审阅报告
```

每个阶段的产出文件可独立使用，也可作为完整调研报告的一部分。

## 许可证

本项目基于 [MIT License](LICENSE) 开源。
