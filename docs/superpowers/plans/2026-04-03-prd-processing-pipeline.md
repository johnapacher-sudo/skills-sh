# PRD 需求处理流水线 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建 3 个 Claude Code Skill（prd-parse、prd-clarify、prd-research），实现 PRD 文档去噪拆分、多轮澄清、汇总输出的需求处理流水线。

**Architecture:** 3 Skill 流水线架构。prd-parse 负责 PRD 去噪和按前端页面/后端模块拆分需求到独立文件；prd-clarify 负责多轮批量澄清不确定项并更新子需求文件；prd-research 作为总调度串联全流程。每个 Skill 使用 Claude Code 标准的 SKILL.md 格式（YAML frontmatter + markdown 指令体），配合 `references/` 目录存放输出模板。

**Tech Stack:** Claude Code Skill（SKILL.md + YAML frontmatter）、Markdown 输出

---

## File Structure

```
skills-sh/
└── skills/
    ├── prd-parse/
    │   ├── SKILL.md                              # PRD 去噪 + 需求拆分 Skill
    │   └── references/
    │       ├── requirement-template.md            # 子需求文件模板
    │       └── split-overview-template.md         # 索引文件模板
    ├── prd-clarify/
    │   ├── SKILL.md                              # 需求澄清 Skill
    │   └── references/
    │       └── clarify-template.md               # 澄清清单模板
    └── prd-research/
        ├── SKILL.md                              # 总调度 Skill
        └── references/
            └── final-template.md                 # 最终需求清单模板
```

---

### Task 1: 创建目录结构

**Files:**
- Create: `skills/prd-parse/references/`
- Create: `skills/prd-clarify/references/`
- Create: `skills/prd-research/references/`

- [ ] **Step 1: 创建所有目录**

```bash
mkdir -p skills/prd-parse/references
mkdir -p skills/prd-clarify/references
mkdir -p skills/prd-research/references
```

- [ ] **Step 2: 验证目录结构**

```bash
find skills/prd-* -type d | sort
```

Expected:
```
skills/prd-clarify
skills/prd-clarify/references
skills/prd-parse
skills/prd-parse/references
skills/prd-research
skills/prd-research/references
```

- [ ] **Step 3: Commit**

```bash
git add skills/prd-parse/ skills/prd-clarify/ skills/prd-research/
git commit -m "chore: create directory structure for PRD processing pipeline skills"
```

---

### Task 2: 创建 prd-parse Skill

**Files:**
- Create: `skills/prd-parse/SKILL.md`
- Create: `skills/prd-parse/references/requirement-template.md`
- Create: `skills/prd-parse/references/split-overview-template.md`

- [ ] **Step 1: 创建子需求文件模板**

Write file `skills/prd-parse/references/requirement-template.md`:

```markdown
# {{编号}}: {{需求名称}}

## 基本信息
- **类型：** {{前端/后端/通用}}
- **来源：** {{PRD 第 X 节 / 第 X-Y 页}}
- **优先级：** 待定

## 需求描述
{{清晰描述该需求要做什么}}

## 功能清单
1. {{功能1}}
2. {{功能2}}

## 验收标准
- [ ] {{标准1}}
- [ ] {{标准2}}

## 不确定项
- [ ] {{待澄清问题1}}
- [ ] {{待澄清问题2}}
```

- [ ] **Step 2: 创建索引文件模板**

Write file `skills/prd-parse/references/split-overview-template.md`:

```markdown
# {{项目名称}} - 需求拆分总览

## 前端需求
- [FE-001 {{页面名称}}](fe-{{page-name}}.md)
- [FE-002 {{页面名称}}](fe-{{page-name}}.md)

## 后端需求
- [BE-001 {{模块名称}}](be-{{module-name}}.md)

## 通用需求
- [CM-001 {{需求名称}}](cm-{{name}}.md)

## 待用户确认
- [ ] 拆分粒度是否合适？
- [ ] 是否有遗漏的页面或模块？
- [ ] 前后端边界是否准确？
```

- [ ] **Step 3: 创建 SKILL.md**

Write file `skills/prd-parse/SKILL.md` with YAML frontmatter:

```yaml
---
name: prd-parse
description: This skill should be used when the user asks to "解析这个 PRD", "拆分需求", "分析需求文档", "整理 PRD", or provides a PRD document that needs to be cleaned and split into structured requirements. Denoises the PRD, splits requirements by frontend pages and backend modules, outputs individual requirement files with an index.
version: 1.0.0
allowed-tools: [Read, Write, Glob, Grep, Bash]
---
```

Body — Chinese imperative form, covering these sections:

1. **概述** — 目的：对 PRD 文档进行去噪过滤和需求拆分，按前端页面维度和后端模块维度将需求拆分为独立文件，产出结构化的需求拆分结果。

2. **执行流程** — 4 个步骤：

   **第一步：读取输入**
   - 检测用户输入类型：直接粘贴的 PRD 文本，或文件路径（.md / .txt）
   - 文件路径：用 Read 工具读取内容
   - 直接文本：直接处理
   - 如果输入既非文件路径也非 PRD 内容，提示用户提供 PRD

   **第二步：去噪过滤**
   - 识别并过滤干扰信息：市场背景、行业分析、商业策略、竞品分析、项目进度计划、里程碑、非功能性约束中的泛泛描述（如"系统要稳定"）、冗余的管理性描述
   - 保留核心信息：功能描述、交互说明、UI 描述、接口需求、数据需求、数据模型、页面/模块描述、业务规则、计算逻辑、权限、角色、状态流转
   - 过滤内容不丢弃，存入 `00-prd-raw.md` 的附录区

   **第三步：需求拆分**
   - 前端需求：按页面维度拆分，每个页面一个独立文件（`fe-<page-name>.md`）
     - 包含：页面名称、功能描述、交互要点、UI 元素、依赖的后端接口
     - 识别页面间的关联和跳转关系
   - 后端需求：按功能模块拆分，每个模块一个独立文件（`be-<module-name>.md`）
     - 包含：模块名称、功能描述、接口设计要点、数据模型、业务规则
   - 通用需求：跨前后端的需求单独列出（`cm-<name>.md`）
     - 包含：权限控制、国际化、主题配置等
   - 编号系统：FE-NNN（前端）、BE-NNN（后端）、CM-NNN（通用），按顺序编号

   **第四步：生成输出**
   - 创建 `requirements/<project-name>/` 目录
   - 生成 `00-prd-raw.md`（去噪后 PRD + 过滤附录）
   - 为每个子需求生成独立文件，使用 `references/requirement-template.md` 模板
   - 生成 `01-split-overview.md` 索引文件，使用 `references/split-overview-template.md` 模板
   - 向用户展示拆分总览，询问是否确认拆分
   - 文件命名规则：前端 `fe-<page-name>.md`、后端 `be-<module-name>.md`、通用 `cm-<name>.md`，名称用 kebab-case

3. **质量约束** — 5 条硬性规则：
   - 每个子需求必须标注来源（PRD 的哪一节/哪一页）
   - 不确定的内容标记为"不确定项"，禁止猜测或编造
   - 过滤内容保留在附录区，不丢弃
   - 去噪保留率应 > 90% 的需求相关内容
   - 术语全文统一，首次出现非中文术语附英文原文

4. **输出规范** — 文件路径、格式说明

- [ ] **Step 4: 验证 SKILL.md 格式**

```bash
head -6 skills/prd-parse/SKILL.md
```

Expected: 文件以 `---` 开头，包含 name/description/version/allowed-tools

- [ ] **Step 5: Commit**

```bash
git add skills/prd-parse/
git commit -m "feat: add prd-parse skill for PRD denoising and requirement splitting"
```

---

### Task 3: 创建 prd-clarify Skill

**Files:**
- Create: `skills/prd-clarify/SKILL.md`
- Create: `skills/prd-clarify/references/clarify-template.md`

- [ ] **Step 1: 创建澄清清单模板**

Write file `skills/prd-clarify/references/clarify-template.md`:

```markdown
# 需求澄清 - 第 {{N}} 轮

## 待澄清项汇总

### {{FE-001}}: {{页面名称}}
- [ ] **Q{{序号}}:** {{具体问题}}
- [ ] **Q{{序号}}:** {{具体问题}}

### {{BE-001}}: {{模块名称}}
- [ ] **Q{{序号}}:** {{具体问题}}

---

## 请逐项回复
> 在对应问题后直接填写回答，不需要的可以标注"不需要"
```

- [ ] **Step 2: 创建 SKILL.md**

Write file `skills/prd-clarify/SKILL.md` with YAML frontmatter:

```yaml
---
name: prd-clarify
description: This skill should be used when the user asks to "澄清需求", "整理待澄清项", "确认需求", or after prd-parse completes. Reads split requirement files, extracts uncertain items, generates clarification checklists, processes user responses, and updates requirement files iteratively until all uncertainties are resolved.
version: 1.0.0
allowed-tools: [Read, Write, Glob, Grep, Bash]
---
```

Body — Chinese imperative form, covering these sections:

1. **概述** — 目的：对拆分后的子需求进行多轮澄清，将不确定项通过批量问答转化为确定需求，直到所有子需求文件无待澄清项。

2. **前置条件** — 读取 `requirements/<project-name>/01-split-overview.md` 获取所有子需求文件列表，逐个读取每个子需求文件。

3. **执行流程** — 5 个步骤：

   **第一步：收集不确定项**
   - 遍历所有子需求文件的"不确定项"部分
   - 检查"需求描述"和"功能清单"中模糊的描述，补充为不确定项
   - 模糊描述识别标准：缺少具体数值（如"大量用户"）、缺少明确条件（如"适当时候"）、缺少边界定义（如"相关数据"）、互相矛盾的描述

   **第二步：生成待澄清清单**
   - 按 `references/clarify-template.md` 模板格式化
   - 按子需求分组列出所有待澄清问题
   - 每个问题必须具体、可回答（禁止模糊问题）
   - 每轮问题数不超过 15 个，超出部分留到下一轮
   - 保存到 `requirements/<project-name>/02-clarify-round-N.md`

   **第三步：等待用户批量回复**
   - 向用户展示待澄清清单
   - 用户在清单中逐项回答

   **第四步：处理回复并更新文件**
   - 已回答的问题：从子需求文件的"不确定项"中移除，回答内容补充到"需求描述"/"功能清单"/"验收标准"中
   - 回答中发现的新不确定项：添加到子需求文件
   - 用户标注"不需要"：从需求中移除，记录到澄清文件
   - 更新 `01-split-overview.md` 索引文件状态

   **第五步：判断是否继续**
   - 所有子需求文件的"不确定项"都为空 → 结束，向用户汇报澄清完成
   - 仍有不确定项 → 递增轮次 N，回到第二步

4. **质量约束** — 5 条硬性规则：
   - 每个澄清问题必须具体、可回答（禁止"你觉得呢？"等模糊问题）
   - 澄清回答必须直接更新到对应子需求文件中
   - 保留所有澄清轮次记录，不覆盖
   - 每轮问题数不超过 15 个（超出拆到下一轮）
   - 更新子需求文件时保持模板格式完整

5. **输出规范** — 文件路径、轮次命名规则

- [ ] **Step 3: 验证 SKILL.md 格式**

```bash
head -6 skills/prd-clarify/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git add skills/prd-clarify/
git commit -m "feat: add prd-clarify skill for multi-round requirement clarification"
```

---

### Task 4: 创建 prd-research 总调度 Skill

**Files:**
- Create: `skills/prd-research/SKILL.md`
- Create: `skills/prd-research/references/final-template.md`

- [ ] **Step 1: 创建最终需求清单模板**

Write file `skills/prd-research/references/final-template.md`:

```markdown
# {{项目名称}} - 最终需求清单

## 概览
- **前端页面数：** {{N}}
- **后端模块数：** {{M}}
- **通用需求数：** {{K}}
- **总需求数：** {{T}}
- **澄清轮次：** {{R}}

---

## 前端需求
- [FE-001 {{页面名称}}](fe-{{page-name}}.md) - 已澄清
- [FE-002 {{页面名称}}](fe-{{page-name}}.md) - 已澄清

## 后端需求
- [BE-001 {{模块名称}}](be-{{module-name}}.md) - 已澄清

## 通用需求
- [CM-001 {{需求名称}}](cm-{{name}}.md) - 已澄清

---

## 需求统计
| 类型 | 数量 | 已澄清 | 待澄清 |
|------|------|--------|--------|
| 前端 | {{N}} | {{N}} | 0 |
| 后端 | {{M}} | {{M}} | 0 |
| 通用 | {{K}} | {{K}} | 0 |
| **合计** | **{{T}}** | **{{T}}** | **0** |

---

## 澄清历史
- [第 1 轮澄清](02-clarify-round-1.md) - 解决 {{X}} 个问题
- [第 2 轮澄清](02-clarify-round-2.md) - 解决 {{Y}} 个问题
```

- [ ] **Step 2: 创建 SKILL.md**

Write file `skills/prd-research/SKILL.md` with YAML frontmatter:

```yaml
---
name: prd-research
description: This skill should be used when the user asks to "处理这个 PRD", "整理需求", "分析 PRD", "需求分析", "拆解 PRD", or provides a PRD document needing full processing. Orchestrates the complete pipeline: parse (denoise + split) → user confirms split → clarify (multi-round) → final summary output.
version: 1.0.0
allowed-tools: [Read, Write, Glob, Grep, Bash]
---
```

Body — Chinese imperative form, covering these sections:

1. **概述** — 目的：作为 PRD 需求处理流水线的总调度入口，协调 parse → clarify → 汇总三个阶段，将复杂 PRD 转化为清晰、经过澄清的需求清单。

2. **执行流程** — 3 个 Phase：

   **Phase 1 - 解析拆分：**
   - 从用户输入中提取 PRD 内容（文本或文件路径）
   - 提取或询问项目名称（用于目录命名）
   - 创建 `requirements/<project-name>/` 目录
   - 调用 prd-parse 执行去噪 + 拆分
   - 展示拆分总览（`01-split-overview.md` 内容）
   - 询问用户是否确认拆分
     - 确认 → 进入 Phase 2
     - 不确认 → 根据反馈调整拆分（合并/拆细/重新归类），重新展示
   - 进度通知：`🔍 阶段一：PRD 解析与需求拆分...` / `✅ 需求拆分完成，共 N 个子需求`

   **Phase 2 - 多轮澄清：**
   - 调用 prd-clarify 生成本轮待澄清清单
   - 展示清单，等待用户批量回复
   - 处理回复，更新各子需求文件
   - 如仍有不确定项，继续下一轮澄清
   - 澄清完成时通知用户
   - 进度通知：`🔍 阶段二：需求澄清（第 N 轮）...` / `✅ 所有不确定项已澄清`

   **Phase 3 - 汇总输出：**
   - 读取所有子需求文件
   - 按 `references/final-template.md` 模板生成最终需求清单 `03-final.md`
   - 包含：概览统计、需求列表（链接到各子需求文件）、需求统计表、澄清历史
   - 进度通知：`📋 阶段三：汇总输出...` / `🎯 需求处理完成！`
   - 向用户输出完成汇总：项目名称、输出目录、文件列表、需求统计

3. **质量约束（贯穿全流程）** — 6 条硬性规则：
   - 全中文输出
   - 禁止编造需求，PRD 中没有的内容标注为"PRD 未提及"
   - 每个需求可追溯来源
   - 澄清问题具体可回答
   - 所有子需求文件使用统一模板
   - 每个阶段完成后向用户汇报进度

4. **进度汇报** — 使用以下格式：
   ```
   🔍 阶段一：PRD 解析与需求拆分...
   ✅ 需求拆分完成，共 N 个子需求

   🔍 阶段二：需求澄清（第 1 轮）...
   ✅ 本轮解决了 X 个问题

   🔍 阶段二：需求澄清（第 2 轮）...
   ✅ 所有不确定项已澄清

   📋 阶段三：汇总输出...
   🎯 需求处理完成！
   ```

5. **输出规范** — 完成汇总格式：
   ```
   ═══════════════════════════════════════
     需求处理完成 — <项目名称>
   ═══════════════════════════════════════

   📁 输出目录: requirements/<project-name>/

   📄 生成文件:
     ├─ 00-prd-raw.md          — 去噪后 PRD
     ├─ 01-split-overview.md   — 拆分总览
     ├─ fe-*.md / be-*.md      — 子需求文件
     ├─ 02-clarify-round-*.md  — 澄清记录
     └─ 03-final.md            — 最终需求清单

   📊 需求统计:
     前端页面: N 个 | 后端模块: M 个 | 通用: K 个
     澄清轮次: R 轮

   ═══════════════════════════════════════
   ```

- [ ] **Step 3: 验证 SKILL.md 格式**

```bash
head -6 skills/prd-research/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git add skills/prd-research/
git commit -m "feat: add prd-research orchestrator skill for full PRD processing pipeline"
```

---

### Task 5: 更新 README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: 读取当前 README.md**

Read `/Users/cuijianwei/workspace/skills-sh/README.md` to understand current content.

- [ ] **Step 2: 在 Skill 列表中添加 PRD 处理 Skill**

在现有 tech-research 系列的说明之后，添加 PRD 处理流水线的说明，包含：

- 新增一个章节 "PRD 需求处理流水线"
- 列出 3 个 Skill 的名称、功能描述、触发词
- 说明流水线架构：parse → clarify → 汇总
- 更新项目结构图，加入 prd-parse/prd-clarify/prd-research 目录
- 更新使用示例，加入 "处理这个 PRD" 的用法示例

- [ ] **Step 3: 验证更新**

```bash
grep -c "prd-" README.md
```

Expected: 至少出现 10 次（3 个 skill 名 + 触发词 + 说明中的引用）

- [ ] **Step 4: Commit**

```bash
git add README.md
git commit -m "docs: update README with PRD processing pipeline skills"
```

---

### Task 6: 最终验证

**Files:**
- Verify: All files in `skills/prd-*`

- [ ] **Step 1: 验证完整文件结构**

```bash
find skills/prd-* -type f | sort
```

Expected:
```
skills/prd-clarify/SKILL.md
skills/prd-clarify/references/clarify-template.md
skills/prd-parse/SKILL.md
skills/prd-parse/references/requirement-template.md
skills/prd-parse/references/split-overview-template.md
skills/prd-research/SKILL.md
skills/prd-research/references/final-template.md
```

- [ ] **Step 2: 验证所有 SKILL.md 的 frontmatter**

```bash
for f in skills/prd-*/SKILL.md; do echo "=== $f ==="; head -6 "$f"; echo; done
```

Expected: 每个文件以 `---` 开头，包含 name/description/version/allowed-tools 字段

- [ ] **Step 3: 验证 Skill name 无冲突**

```bash
grep -r "^name:" skills/prd-*/SKILL.md
```

Expected 3 个不同的 name：prd-parse, prd-clarify, prd-research

- [ ] **Step 4: 验证与已有 Skill 无冲突**

```bash
grep -r "^name:" skills/*/SKILL.md
```

Expected 8 个不同的 name（5 个 tech-* + 3 个 prd-*），无重复
