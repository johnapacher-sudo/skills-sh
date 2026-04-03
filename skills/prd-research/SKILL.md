---
name: prd-research
description: This skill should be used when the user asks to "处理这个 PRD", "整理需求", "分析 PRD", "需求分析", "拆解 PRD", or provides a PRD document needing full processing. Orchestrates the complete pipeline: parse (denoise + split) → clarify (multi-round) → final summary output.
---

# PRD 需求处理总调度（prd-research）

## 概述

本 Skill 作为 PRD 需求处理流水线的总调度入口，协调 parse（解析拆分）、clarify（多轮澄清）、汇总（最终输出）三个阶段，将复杂 PRD 转化为清晰、经过澄清的需求清单。当用户提到"处理这个 PRD"、"整理需求"、"分析 PRD"、"需求分析"、"拆解 PRD"等意图，或直接提供一份 PRD 文档要求处理时，自动触发本技能。

核心目标：端到端管理 PRD 处理全流程，通过 Skill 工具调用 prd-parse 和 prd-clarify 两个子技能，确保每个阶段使用子技能的最新版本执行，最终输出完整的、经过澄清的需求清单和统计汇总。

本技能仅负责编排调度，不直接实现去噪、拆分或澄清逻辑。

## 执行流程

整个处理流程分为三个 Phase，严格按顺序执行。每个 Phase 开始时发送进度通知，完成后发送完成通知。

### Phase 1 - 解析拆分

本阶段准备输入参数，通过 Skill 工具调用 prd-parse 完成去噪过滤和需求拆分。prd-parse 会自行完成全部流程并等待用户确认拆分结果。

**准备阶段：**

1. **提取 PRD 内容：** 从用户输入中识别 PRD 内容。如果用户提供的是文件路径（以 `.md` 或 `.txt` 结尾），使用 Read 工具读取文件内容；如果用户直接粘贴了 PRD 文本，直接使用该文本作为输入。如果用户未提供有效的 PRD 内容，提示用户提供 PRD 文档或文件路径。

2. **提取项目名称：** 从 PRD 内容中识别项目名称，用于后续目录命名。项目名称转换为 kebab-case 格式（如"用户管理系统"转为 `user-management-system`）。如果 PRD 中未明确项目名称，主动向用户询问。

3. **创建输出目录：** 使用 Bash 工具创建 `requirements/<project-name>/` 目录，确保目录存在且可写。

**调用子技能：**

发送进度通知 `🔍 阶段一：PRD 解析与需求拆分...`，然后使用 Skill 工具调用 `prd-parse`，通过 args 参数传入 PRD 内容和项目名称：

```
Skill 工具调用：
  skill: prd-parse
  args: PRD 内容和项目名称
```

prd-parse 将完整执行其自身的 4 步流程（读取输入、去噪过滤、需求拆分、生成输出），包括向用户展示拆分总览并等待确认。prd-research 不重复执行这些步骤，不重复请求用户确认。

**调用后校验：**

prd-parse 完成后，读取 `requirements/<project-name>/01-split-overview.md` 确认文件存在且内容完整。如果索引文件缺失或子需求文件列表为空，向用户报告错误并询问是否重试。

**进度通知：**

```
🔍 阶段一：PRD 解析与需求拆分...
✅ 需求拆分完成，共 N 个子需求
```

### Phase 2 - 多轮澄清

本阶段通过 Skill 工具调用 prd-clarify，对拆分后的子需求进行逐轮澄清，直到所有不确定项全部清零。prd-clarify 会自行管理多轮迭代和用户交互。

**调用子技能：**

发送进度通知 `🔍 阶段二：需求澄清...`，然后使用 Skill 工具调用 `prd-clarify`，通过 args 参数传入项目名称：

```
Skill 工具调用：
  skill: prd-clarify
  args: 项目名称
```

prd-clarify 将完整执行其自身的 5 步流程（收集不确定项、生成待澄清清单、等待用户批量回复、处理回复并更新文件、判断是否继续），包括多轮迭代直到所有不确定项清零。prd-research 不重复执行这些步骤，不接管 prd-clarify 的轮次进度汇报。

**调用后校验：**

prd-clarify 完成后，遍历所有子需求文件（`fe-*.md`、`be-*.md`、`cm-*.md`），确认每个文件的"不确定项"章节为空。如果仍有未清零的不确定项，向用户报告并询问是否继续澄清或跳过进入汇总。

**进度通知：**

```
🔍 阶段二：需求澄清...
✅ 所有不确定项已澄清
```

### Phase 3 - 汇总输出

本阶段由 prd-research 直接执行，读取所有子需求文件，按模板生成最终的需求清单汇总文件，并向用户输出完成报告。

**读取子需求文件：** 根据索引文件 `01-split-overview.md` 中记录的文件列表，逐个读取所有子需求文件（`fe-*.md`、`be-*.md`、`cm-*.md`），确认每个文件内容完整。

**生成最终清单：** 读取 `references/final-template.md` 模板文件，将模板中的 `{{占位符}}` 替换为实际内容，生成 `03-final.md` 文件。替换内容包括：

- 项目名称
- 前端页面数（N）、后端模块数（M）、通用需求数（K）、总需求数（T = N + M + K）
- 澄清轮次（R）
- 每个子需求的编号、名称和文件链接
- 需求统计表中的各分类数量
- 澄清历史中每轮解决的问题数

**包含完整信息：** 最终清单文件必须包含概览统计、需求列表（链接到各子需求文件）、需求统计表和澄清历史四个部分。需求列表按前端、后端、通用三个分类组织，每个需求项包含编号、名称、文件链接和状态（已澄清）。

**进度通知：**

```
📋 阶段三：汇总输出...
🎯 需求处理完成！
```

**向用户输出完成汇总：**

```
🎯 PRD 需求处理完成！

📁 输出目录：requirements/<project-name>/
📄 生成文件：
   - 00-prd-raw.md（去噪后原始文档）
   - 01-split-overview.md（需求拆分总览）
   - 02-clarify-round-1.md ~ 02-clarify-round-N.md（澄清记录）
   - 03-final.md（最终需求清单）
   - fe-*.md（前端需求，共 N 个）
   - be-*.md（后端需求，共 M 个）
   - cm-*.md（通用需求，共 K 个）

📊 需求统计：
   前端需求：N 个
   后端需求：M 个
   通用需求：K 个
   合计：T 个
   澄清轮次：R 轮
```

## 错误处理

以下场景需要特殊处理，确保流程不会中断：

### 子技能调用失败

如果 Skill 工具调用 prd-parse 或 prd-clarify 时返回错误（如技能不存在、执行异常），向用户展示具体的错误信息，并提供以下选项：
- **重试：** 重新调用该子技能
- **跳过：** 跳过当前阶段进入下一阶段（仅 Phase 2 可跳过，Phase 1 不可跳过）
- **中止：** 终止整个流程

### 前置文件缺失

如果进入 Phase 2 时 `01-split-overview.md` 或子需求文件不存在（如用户手动删除了文件），提示用户先执行 prd-parse，或询问是否使用当前目录下的现有文件继续。

### 澄清后仍有不确定项

如果 Phase 2 结束后仍有子需求文件存在未清零的不确定项（如用户主动跳过了某些问题），在 Phase 3 汇总时将这些项标注为"未澄清"状态，不阻塞最终清单生成。

## 质量约束

以下规则为硬性要求，贯穿全流程的每个阶段，任何情况下不得违反。

### 全中文输出

所有向用户展示的内容、生成的文件内容、进度通知和完成汇总均使用中文。文件中的技术术语保留英文原文，首次出现时附中文说明。子需求文件中的描述性内容使用中文撰写。

### 禁止编造需求

严格忠于 PRD 原文内容，不猜测、不推断、不编造 PRD 中未提及的需求。对于 PRD 中表述模糊或信息不完整的部分，统一标记为"不确定项"，纳入澄清流程由用户确认。如果用户在澄清中无法提供信息，标注为"PRD 未提及"，保持原始状态而非自行补充。

### 统一模板格式

所有子需求文件使用 prd-parse 的 `references/requirement-template.md` 模板生成，保持章节结构一致。索引文件使用 `references/split-overview-template.md` 模板。最终清单使用 `references/final-template.md` 模板。在澄清过程中更新子需求文件时，不改变文件的模板结构，仅更新对应章节的内容。

### 阶段进度汇报

每个 Phase 开始时发送进度通知，每个 Phase 完成时发送完成通知。进度通知使用指定的格式和图标。进度通知内容准确反映当前状态，不夸大进展，不隐瞒问题。

## 输出规范

### 完成汇总格式

流程全部完成后，向用户输出以下格式的汇总信息：

```
🎯 PRD 需求处理完成！

📁 输出目录：requirements/<project-name>/
📄 生成文件：
   - 00-prd-raw.md（去噪后原始文档）
   - 01-split-overview.md（需求拆分总览）
   - 02-clarify-round-1.md ~ 02-clarify-round-N.md（澄清记录，共 R 轮）
   - 03-final.md（最终需求清单）
   - fe-*.md（前端需求，共 N 个）
   - be-*.md（后端需求，共 M 个）
   - cm-*.md（通用需求，共 K 个）

📊 需求统计：
   前端需求：N 个（已澄清 N 个）
   后端需求：M 个（已澄清 M 个）
   通用需求：K 个（已澄清 K 个）
   合计：T 个
   澄清轮次：R 轮
```

### 最终文件路径

所有输出文件存放在以下路径：

```
requirements/<project-name>/
├── 00-prd-raw.md              # 去噪后 PRD + 过滤附录
├── 01-split-overview.md       # 需求拆分总览索引
├── 02-clarify-round-1.md      # 第 1 轮澄清记录
├── 02-clarify-round-2.md      # 第 2 轮澄清记录
├── ...                        # 更多澄清轮次（如需要）
├── 03-final.md                # 最终需求清单汇总
├── fe-<page-name>.md          # 前端需求文件
├── be-<module-name>.md        # 后端需求文件
└── cm-<name>.md               # 通用需求文件
```

### 文件格式

- 格式：Markdown，UTF-8 编码
- 模板：子需求文件遵循 prd-parse 的 `references/requirement-template.md`；索引文件遵循 `references/split-overview-template.md`；最终清单遵循 `references/final-template.md`
- 文件命名统一使用 kebab-case 格式
- 每个文件独立完整，可脱离其他文件单独阅读
