# Skills SH

Claude Code Agent 自定义技能集合 -- 技术深度调研 & PRD 需求处理工具

## 简介

Skills SH 是一套面向 Claude Code Agent 的多技能流水线工具，目前包含两大工具集：

1. **技术深度调研流水线**：通过将调研过程拆分为发现、分析、教程、审阅四个阶段，配合总调度技能，实现从"了解一项技术"到"输出高质量调研报告"的全流程自动化。
2. **PRD 需求处理流水线**：通过去噪、拆分、多轮澄清三个阶段，将粗糙的产品需求文档（PRD）转化为清晰、结构化的开发需求清单。

无论是快速评估一个新框架，还是对多项技术进行深度对比分析，亦或是生成可落地的硬核教程，技术调研流水线都能系统化地完成。而 PRD 处理流水线则能帮助你将模糊的产品需求梳理为开发团队可直接执行的需求文档。

## Skill 列表

| Skill | 功能 | 触发词 |
|-------|------|--------|
| tech-research | 总调度，一键全流程调研 | "调研 xxx"、"深度研究 xxx" |
| tech-discover | 技术发现与信息检索 | "快速了解 xxx"、"xxx 是什么" |
| tech-analyze | 深度分析与对比 | "深度分析 xxx"、"对比 xxx 和 yyy" |
| tech-tutorial | 硬核教程生成 | "出一份 xxx 教程"、"学习 xxx" |
| tech-review | 内容质量审阅 | "审阅 xxx 调研"、"检查调研质量" |

### PRD 需求处理流水线

| Skill | 功能 | 触发词 |
|-------|------|--------|
| prd-research | 总调度，一键全流程 PRD 处理 | "处理这个 PRD"、"整理需求" |
| prd-parse | PRD 去噪 + 需求拆分 | "解析这个 PRD"、"拆分需求" |
| prd-clarify | 多轮需求澄清 | "澄清需求"、"确认需求" |

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

### PRD 需求处理流水线

```
PRD 输入 → prd-parse（去噪+拆分）→ 用户确认拆分 → prd-clarify（多轮澄清）→ 汇总输出
```

- **prd-parse**：对原始 PRD 进行去噪处理，去除无关内容，然后将需求按前端（按页面）、后端（按模块）、通用等维度拆分为独立需求文件
- **prd-clarify**：针对拆分后的需求进行多轮澄清，发现并补充缺失信息、矛盾点和模糊描述，确保需求清晰可执行

**prd-research** 作为总调度器，可一键串联以上阶段，自动完成从原始 PRD 到结构化需求清单的全流程。你也可以单独调用某个阶段的技能，按需使用。

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
│   ├── tech-research/
│   │   └── SKILL.md
│   ├── prd-parse/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── requirement-template.md
│   │       └── split-overview-template.md
│   ├── prd-clarify/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── clarify-template.md
│   └── prd-research/
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

#### PRD 需求处理

对 Claude Code 说 `处理这个 PRD`，即可一键启动全流程 PRD 处理。也可以单独调用各阶段：

- **单独阶段**：`解析这个 PRD`、`拆分需求`、`澄清需求`、`确认需求`

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

### PRD 处理输出示例

使用 prd-research 进行全流程 PRD 处理后，会生成以下文件结构：

```
requirements/<project-name>/
├── 00-prd-raw.md          # 去噪后 PRD
├── 01-split-overview.md   # 拆分总览
├── fe-*.md                # 前端需求（按页面）
├── be-*.md                # 后端需求（按模块）
├── cm-*.md                # 通用需求
├── 02-clarify-round-*.md  # 澄清记录
└── 03-final.md            # 最终需求清单
```

每个阶段的产出文件可独立使用，最终生成的 `03-final.md` 即为可直接交付给开发团队的结构化需求清单。

## 许可证

本项目基于 [MIT License](LICENSE) 开源。
