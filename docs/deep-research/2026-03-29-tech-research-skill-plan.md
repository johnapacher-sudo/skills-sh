# 技术深度调研 Skill 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建 4+1 个 Claude Code Skill（SKILL.md 格式），实现技术深度调研流水线。

**Architecture:** 多 Skill 流水线架构，按 discover → analyze → tutorial → review 顺序执行，tech-research 作为总调度入口。每个 Skill 使用 Claude Code 标准的 `SKILL.md` 格式（YAML frontmatter + markdown 指令体），配合 `references/` 目录存放输出模板。

**Tech Stack:** Claude Code Skill（SKILL.md + YAML frontmatter）、Markdown 输出、WebSearch/WebFetch 工具

**Design deviation:** 设计文档中指定使用 `.sh` 脚本格式，但 Claude Code 的 Skill 系统实际使用 `SKILL.md` 目录格式（YAML frontmatter + markdown body）。本计划采用 SKILL.md 格式，以确保 Skill 能被 Claude Code 正确发现和触发。输出模板放在各 Skill 的 `references/` 子目录中。

---

## File Structure

```
skills-sh/
├── skills/
│   ├── tech-discover/
│   │   ├── SKILL.md                          # 技术发现与检索 Skill
│   │   └── references/
│   │       └── output-template.md            # discovery 输出模板
│   ├── tech-analyze/
│   │   ├── SKILL.md                          # 深度分析 Skill
│   │   └── references/
│   │       └── output-template.md            # analysis 输出模板
│   ├── tech-tutorial/
│   │   ├── SKILL.md                          # 硬核教程生成 Skill
│   │   └── references/
│   │       └── output-template.md            # tutorial 输出模板
│   ├── tech-review/
│   │   ├── SKILL.md                          # 内容质量审阅 Skill
│   │   └── references/
│   │       └── review-checklist.md           # 审阅清单
│   └── tech-research/
│       └── SKILL.md                          # 总调度入口 Skill
├── docs/
│   └── deep-research/
│       ├── 2026-03-28-tech-research-skill-design.md
│       └── 2026-03-29-tech-research-skill-plan.md
├── LICENSE
└── README.md
```

---

### Task 1: 创建项目目录结构

**Files:**
- Create: `skills/tech-discover/references/`
- Create: `skills/tech-analyze/references/`
- Create: `skills/tech-tutorial/references/`
- Create: `skills/tech-review/references/`
- Create: `skills/tech-research/`

- [ ] **Step 1: 创建所有目录**

```bash
mkdir -p skills/tech-discover/references
mkdir -p skills/tech-analyze/references
mkdir -p skills/tech-tutorial/references
mkdir -p skills/tech-review/references
mkdir -p skills/tech-research
```

- [ ] **Step 2: 验证目录结构**

```bash
find skills/ -type d | sort
```

Expected output:
```
skills
skills/tech-analyze
skills/tech-analyze/references
skills/tech-discover
skills/tech-discover/references
skills/tech-research
skills/tech-review
skills/tech-review/references
skills/tech-tutorial
skills/tech-tutorial/references
```

- [ ] **Step 3: Commit**

```bash
git add skills/
git commit -m "chore: create skill directory structure for tech research pipeline"
```

---

### Task 2: 创建 tech-discover Skill

**Files:**
- Create: `skills/tech-discover/SKILL.md`
- Create: `skills/tech-discover/references/output-template.md`

- [ ] **Step 1: 创建输出模板文件**

```markdown
<!-- File: skills/tech-discover/references/output-template.md -->
# {{技术名称}} - 技术发现报告

## 基本信息
- **名称：**
- **官方地址：**
- **GitHub 仓库：**
- **当前版本：**
- **License：**
- **主要编程语言：**
- **首次发布日期：**
- **创建者/维护团队：**

## 一句话定位

> {{用一句话概括这个技术是什么、解决什么问题}}

## 核心特性

1. **{{特性1}}** - {{简述}}
2. **{{特性2}}** - {{简述}}
3. **{{特性3}}** - {{简述}}
...

## 社区生态
- **GitHub Stars：**
- **Contributors：**
- **最近更新日期：**
- **社区讨论热度：** （Reddit/HackerNews/StackOverflow 活跃度）
- **npm/pypi 下载量（如适用）：**

## 技术栈定位
- **所属领域：**
- **解决的核心问题：**
- **替代/竞品技术：**
- **依赖的上游技术：**
- **下游使用者/集成方：**

## 关键链接汇总

### 官方资源
- [官方网站]()
- [官方文档]()
- [GitHub 仓库]()
- [更新日志]()

### 教程资源
- [快速入门]()
- [官方示例]()

### 社区资源
- [社区论坛/Discord]()
- [技术博客文章]()

## 信息来源
- [来源1](url) - 获取日期：{{date}}
- [来源2](url) - 获取日期：{{date}}
- [来源3](url) - 获取日期：{{date}}
```

- [ ] **Step 2: 创建 SKILL.md**

SKILL.md 内容（YAML frontmatter + markdown 指令体）：

**Frontmatter:**
- `name`: tech-discover
- `description`: This skill should be used when the user asks to "快速了解 xxx", "xxx 是什么", "发现 xxx 的信息", "介绍一下 xxx", "xxx 概览", or wants a quick overview of a technology. Performs multi-source web search and information gathering to produce a structured technology discovery report.
- `allowed-tools`: [WebSearch, WebFetch, Read, Write, Glob, Grep, Bash]

**Body 核心指令：**
1. 解析用户提供的技术名称/关键词/URL
2. 执行多维度搜索（至少 3 个搜索查询）：技术名称 + "official site" / 技术名称 + "github" / 技术名称 + "tutorial review"
3. 抓取关键页面内容：官网首页、GitHub README、文档 Getting Started
4. 交叉验证信息：对比多个来源，标注不一致处
5. 质量规则：禁止编造信息，不确定标注"待验证"，每个关键信息标注来源
6. 按 `references/output-template.md` 模板格式化输出
7. 保存到 `research/<tech-name>/01-discovery.md`

- [ ] **Step 3: 验证 Skill 文件格式**

```bash
head -5 skills/tech-discover/SKILL.md
```

Expected: 文件以 `---` 开头，包含 YAML frontmatter

- [ ] **Step 4: Commit**

```bash
git add skills/tech-discover/
git commit -m "feat: add tech-discover skill for technology discovery and information gathering"
```

---

### Task 3: 创建 tech-analyze Skill

**Files:**
- Create: `skills/tech-analyze/SKILL.md`
- Create: `skills/tech-analyze/references/output-template.md`

- [ ] **Step 1: 创建输出模板文件**

```markdown
<!-- File: skills/tech-analyze/references/output-template.md -->
# {{技术名称}} - 深度分析报告

## 技术背景与动机

### 行业背景
{{该技术诞生时的行业背景和痛点}}

### 创立动机
{{为什么被创造，解决什么具体问题}}

### 发展历程
{{关键版本和里程碑，用时间线形式}}

## 核心原理

### 设计哲学
{{该技术的设计理念和核心思想}}

### 核心算法/机制
{{深入讲解核心工作原理，配图或流程说明}}

### 数据流/执行流程
{{输入到输出的完整流程}}

## 架构设计

### 整体架构
{{架构图或分层描述}}

### 核心模块
- **{{模块1}}** - {{职责}}
- **{{模块2}}** - {{职责}}

### 扩展机制
{{如何扩展/插件机制}}

## 关键概念详解

### {{概念1}}
- **定义：**
- **作用：**
- **使用场景：**
- **代码示例：**

### {{概念2}}
...

## 同类技术横向对比

| 维度 | {{本技术}} | {{竞品A}} | {{竞品B}} |
|------|-----------|----------|----------|
| 核心理念 | | | |
| 性能 | | | |
| 易用性 | | | |
| 生态丰富度 | | | |
| 社区规模 | | | |
| 学习曲线 | | | |
| 生产就绪度 | | | |
| 适用场景 | | | |

## 适用场景分析

### 最佳场景
1. {{场景1}}
2. {{场景2}}

### 不适用场景
1. {{场景1}}
2. {{场景2}}

## 优缺点深度分析

### 优势
1. {{优势1}} - {{详细说明}}
...

### 劣势
1. {{劣势1}} - {{详细说明}}
...

### 风险点
1. {{风险1}} - {{影响和缓解措施}}

## 生态成熟度评估
- **插件/扩展数量：**
- **第三方库支持：**
- **企业采用案例：**
- **文档质量：**

## 生产环境就绪度评估
- **稳定性：**
- **性能表现：**
- **监控/可观测性：**
- **故障恢复：**
- **安全合规：**

## 学习曲线评估
- **前置知识要求：**
- **入门时间估计：**
- **精通时间估计：**

## 总结与建议
{{综合评价和使用建议}}

## 信息来源与版本说明
- **分析基于版本：** {{version}}
- **信息获取日期：** {{date}}
- **信息来源列表：**
  - [来源1](url)
  - [来源2](url)
```

- [ ] **Step 2: 创建 SKILL.md**

**Frontmatter:**
- `name`: tech-analyze
- `description`: This skill should be used when the user asks to "深度分析 xxx", "对比 xxx 和 yyy", "分析 xxx 的原理", "xxx 架构", or wants a deep technical analysis of a technology. Reads the discovery report and performs in-depth analysis of architecture, principles, and comparison with alternatives.
- `allowed-tools`: [WebSearch, WebFetch, Read, Write, Glob, Grep, Bash]

**Body 核心指令：**
1. 读取 `research/<tech-name>/01-discovery.md`
2. 识别需要深入了解的关键点
3. 搜索补充信息：源码分析文章、架构设计文档、性能评测、技术博客深度文章
4. 抓取并分析技术原理和架构
5. 进行同类技术横向对比（表格形式，至少 3 个对比维度 × 3 个技术）
6. 质量规则：
   - 代码示例必须基于官方文档，标注"基于官方文档"或"基于实际测试"
   - 标注分析所基于的具体版本号
   - 对比维度必须一致，数据可查
   - 不确信的结论标注置信度（高/中/低）
7. 按 `references/output-template.md` 模板格式化输出
8. 保存到 `research/<tech-name>/02-analysis.md`

- [ ] **Step 3: 验证 Skill 文件格式**

```bash
head -5 skills/tech-analyze/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git add skills/tech-analyze/
git commit -m "feat: add tech-analyze skill for deep technical analysis"
```

---

### Task 4: 创建 tech-tutorial Skill

**Files:**
- Create: `skills/tech-tutorial/SKILL.md`
- Create: `skills/tech-tutorial/references/output-template.md`

- [ ] **Step 1: 创建输出模板文件**

```markdown
<!-- File: skills/tech-tutorial/references/output-template.md -->
# {{技术名称}} - 完整学习教程

> **教程级别：** 从零到一
> **预计学习时间：** {{hours}} 小时
> **前置知识：** {{prerequisites}}

## 环境搭建指南

### 系统要求
- 操作系统：
- 运行时/依赖版本：

### 安装步骤

```bash
# 完整安装命令
```

### 验证安装

```bash
# 验证命令
```

---

## 第一部分：入门篇

### 1.1 {{核心概念A}}

**概念讲解：**
{{详细解释}}

**代码示例：**
```bash
# 完整可运行的示例代码
```

**执行结果：**
```
{{预期输出}}
```

**练习题：**
1. {{练习1}}
2. {{练习2}}

---

### 1.2 {{核心概念B}}

**概念讲解：**
{{详细解释，建立在前一节基础之上}}

**代码示例：**
```bash
# 完整可运行的示例代码
```

**执行结果：**
```
{{预期输出}}
```

**练习题：**
1. {{练习1}}
2. {{练习2}}

---

## 第二部分：进阶篇

### 2.1 {{关键特性X}}

**详细讲解：**
{{基于入门篇知识展开}}

**代码示例：**
```bash
# 完整可运行的示例代码
```

**注意事项：**
- {{坑点1}}
- {{坑点2}}

**练习题：**
1. {{练习1}}

---

### 2.2 {{关键特性Y}}
...

---

## 第三部分：高级篇

### 3.1 高级用法

### 3.2 性能优化
- **优化策略1：** {{说明}} + 代码示例
- **优化策略2：** {{说明}} + 代码示例

### 3.3 最佳实践
1. {{实践1}}
2. {{实践2}}

---

## 第四部分：实战项目

### 项目需求
{{描述一个综合运用所学知识的实际项目}}

### 项目设计
{{架构设计、技术选型}}

### 完整实现代码

```bash
# 完整可运行的项目代码
```

### 代码解析
{{逐段解析关键实现}}

### 扩展挑战
1. {{挑战1}}
2. {{挑战2}}

---

## 第五部分：常见问题与排查指南

### 常见错误及解决方案
| 错误信息 | 原因 | 解决方案 |
|----------|------|----------|
| {{error}} | {{cause}} | {{solution}} |

### 调试技巧
1. {{技巧1}}
2. {{技巧2}}

---

## 第六部分：学习路线推荐

### 官方文档推荐阅读顺序
1. {{章节1}} - {{重点}}
2. {{章节2}} - {{重点}}

### 推荐进阶资源
- {{书籍/课程1}}
- {{书籍/课程2}}
```

- [ ] **Step 2: 创建 SKILL.md**

**Frontmatter:**
- `name`: tech-tutorial
- `description`: This skill should be used when the user asks to "出一份 xxx 的教程", "生成 xxx 学习教程", "xxx 教程", "学习 xxx", or wants a comprehensive hands-on tutorial for a technology. Reads discovery and analysis reports to produce a structured tutorial with runnable code examples.
- `allowed-tools`: [WebSearch, WebFetch, Read, Write, Glob, Grep, Bash]

**Body 核心指令：**
1. 读取 `research/<tech-name>/01-discovery.md` 和 `research/<tech-name>/02-analysis.md`
2. 从 analysis 报告中提取所有核心概念和知识点
3. 设计教程结构（入门→进阶→高级→实战），每个知识点建立在前一个之上
4. 为每个知识点：
   - 写清晰的概念讲解
   - 编写**完整可运行的代码示例**（非伪代码、非片段）
   - 标注预期输出
   - 提供 1-2 个练习题
5. 设计一个贯穿教程的综合实战项目
6. 编写常见错误排查表和调试技巧
7. 质量规则：
   - 循序渐进，每节建立在前一节之上
   - 每个代码示例都是完整可运行的，不写伪代码
   - 标注常见坑点和避坑方法
   - 实战项目必须综合运用教程中的知识点
8. 按 `references/output-template.md` 模板格式化输出
9. 保存到 `research/<tech-name>/03-tutorial.md`

- [ ] **Step 3: 验证 Skill 文件格式**

```bash
head -5 skills/tech-tutorial/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git add skills/tech-tutorial/
git commit -m "feat: add tech-tutorial skill for generating comprehensive hands-on tutorials"
```

---

### Task 5: 创建 tech-review Skill

**Files:**
- Create: `skills/tech-review/SKILL.md`
- Create: `skills/tech-review/references/review-checklist.md`

- [ ] **Step 1: 创建审阅清单文件**

```markdown
<!-- File: skills/tech-review/references/review-checklist.md -->
# 调研报告质量审阅清单

## 1. 事实准确性
- [ ] 所有技术声明都有来源支撑
- [ ] 版本号与官方最新版本一致
- [ ] 性能数据有出处
- [ ] 无编造或猜测的信息

## 2. 代码可运行性
- [ ] 所有代码示例语法正确
- [ ] 代码示例包含完整的导入和依赖
- [ ] 代码示例可在指定环境下直接运行
- [ ] 预期输出与实际一致

## 3. 完整性
- [ ] discovery 覆盖了所有基本信息项
- [ ] analysis 覆盖了所有核心概念
- [ ] tutorial 覆盖了从入门到高级的所有关键知识点
- [ ] 横向对比包含至少 3 个竞品

## 4. 逻辑递进
- [ ] tutorial 章节顺序由浅入深
- [ ] 每个知识点建立在前一个之上
- [ ] 实战项目综合运用了教程中的知识

## 5. 术语一致性
- [ ] 同一概念全文使用统一术语
- [ ] 中英文术语对应关系一致
- [ ] 代码中的命名与文字描述一致

## 6. 时效性
- [ ] 信息基于最新版本
- [ ] 获取日期已标注
- [ ] 过时信息已标记

## 7. 来源可溯
- [ ] 关键信息标注了来源链接
- [ ] 至少 3 个独立信息来源
- [ ] 来源链接可访问

---

## 问题分级标准

| 级别 | 定义 | 示例 | 处理要求 |
|------|------|------|----------|
| **P0** | 事实错误 | API 签名写错、原理描述错误 | **必须修复** |
| **P0** | 代码无法运行 | 缺少导入、语法错误 | **必须修复** |
| **P0** | 严重遗漏 | 遗漏核心技术概念 | **必须修复** |
| **P1** | 重要信息不完整 | 缺少关键参数说明 | **应当修复** |
| **P1** | 代码示例不清晰 | 缺少注释或上下文 | **应当修复** |
| **P1** | 对比分析缺失维度 | 遗漏重要对比维度 | **应当修复** |
| **P2** | 表述优化 | 可以更精炼的表达 | 建议改进 |
| **P2** | 排版改进 | 格式不统一 | 建议改进 |
| **P2** | 补充资源 | 可增加推荐阅读 | 建议改进 |
```

- [ ] **Step 2: 创建 SKILL.md**

**Frontmatter:**
- `name`: tech-review
- `description`: This skill should be used when the user asks to "审阅 xxx 的调研报告", "检查调研质量", "review xxx 调研", or after tech-tutorial completes. Reviews all generated research documents for accuracy, completeness, consistency, and quality.
- `allowed-tools`: [WebSearch, WebFetch, Read, Write, Glob, Grep, Bash]

**Body 核心指令：**
1. 读取 `research/<tech-name>/` 下所有已生成的文档（01-03）
2. 按 `references/review-checklist.md` 中的审阅清单逐项检查
3. 对发现的每个问题进行分级（P0/P1/P2）
4. 针对 P0/P1 问题：
   - 通过 WebSearch/WebFetch 验证关键事实
   - 修正事实错误和代码问题
   - 补充遗漏内容
5. 生成审阅报告，包含：
   - 审阅清单检查结果
   - 发现的问题列表（按级别排列）
   - 每个问题的修正说明
   - 已执行的修正内容
   - 最终质量评分（A/B/C/D）
6. 如存在 P0/P1 问题，直接修改对应的源文件（01-03），在 04-review.md 中记录修改内容
7. 保存到 `research/<tech-name>/04-review.md`

- [ ] **Step 3: 验证 Skill 文件格式**

```bash
head -5 skills/tech-review/SKILL.md
```

- [ ] **Step 4: Commit**

```bash
git add skills/tech-review/
git commit -m "feat: add tech-review skill for quality review of research documents"
```

---

### Task 6: 创建 tech-research 总调度 Skill

**Files:**
- Create: `skills/tech-research/SKILL.md`

- [ ] **Step 1: 创建 SKILL.md**

**Frontmatter:**
- `name`: tech-research
- `description`: This skill should be used when the user asks to "调研 xxx", "全面调研 xxx", "深度研究 xxx", "research xxx", or wants a complete technology research report. Orchestrates the full pipeline: discover → analyze → tutorial → review.
- `allowed-tools`: [WebSearch, WebFetch, Read, Write, Glob, Grep, Bash]

**Body 核心指令：**

整个调研流水线的总调度入口。收到用户请求后，按以下顺序执行：

**Phase 1 - 准备：**
1. 从用户输入中提取技术名称（可能是一个关键词、GitHub URL 或技术全称）
2. 将技术名称转为 kebab-case 目录名（如 "React Server Components" → "react-server-components"）
3. 创建 `research/<tech-name>/` 目录

**Phase 2 - 执行流水线：**
4. 执行 `tech-discover` Skill 的完整指令流程，生成 `01-discovery.md`
5. 执行 `tech-analyze` Skill 的完整指令流程，生成 `02-analysis.md`
6. 执行 `tech-tutorial` Skill 的完整指令流程，生成 `03-tutorial.md`
7. 执行 `tech-review` Skill 的完整指令流程，生成 `04-review.md`

**Phase 3 - 收尾：**
8. 如 review 发现 P0/P1 问题，回溯修复相关文档
9. 输出完成汇总：
   ```markdown
   ## 调研完成汇总

   **技术：** {{name}}
   **输出目录：** research/{{tech-name}}/
   **生成文件：**
   - 01-discovery.md (技术发现)
   - 02-analysis.md (深度分析)
   - 03-tutorial.md (学习教程)
   - 04-review.md (质量审阅)

   **质量评分：** {{grade}}
   **P0 问题：** {{count}} (全部已修复)
   **P1 问题：** {{count}} (全部已修复)
   **P2 建议：** {{count}}
   ```

**质量约束（贯穿全流程）：**
- 全中文输出
- 禁止编造信息，不确定标注"待验证"
- 每个关键信息标注来源
- 代码示例必须完整可运行
- 所有 P0/P1 问题必须修复

- [ ] **Step 2: 验证 Skill 文件格式**

```bash
head -5 skills/tech-research/SKILL.md
```

- [ ] **Step 3: Commit**

```bash
git add skills/tech-research/
git commit -m "feat: add tech-research orchestrator skill for full research pipeline"
```

---

### Task 7: 更新 README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: 更新 README.md 内容**

更新项目结构说明和 Skill 列表，反映新的 SKILL.md 目录格式和 5 个调研 Skill。

关键更新：
- 项目结构改为目录格式
- 添加 5 个调研 Skill 的说明
- 更新安装方式说明
- 更新 Skill 开发格式说明（从 `.sh` 改为 `SKILL.md`）

- [ ] **Step 2: 验证 README 内容**

```bash
cat README.md | head -40
```

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: update README with tech research skills documentation"
```

---

### Task 8: 最终验证

**Files:**
- Verify: All files in `skills/`

- [ ] **Step 1: 验证完整目录结构**

```bash
find skills/ -type f | sort
```

Expected:
```
skills/tech-analyze/SKILL.md
skills/tech-analyze/references/output-template.md
skills/tech-discover/SKILL.md
skills/tech-discover/references/output-template.md
skills/tech-research/SKILL.md
skills/tech-review/SKILL.md
skills/tech-review/references/review-checklist.md
skills/tech-tutorial/SKILL.md
skills/tech-tutorial/references/output-template.md
```

- [ ] **Step 2: 验证所有 SKILL.md 的 frontmatter 格式**

```bash
for f in skills/*/SKILL.md; do echo "=== $f ==="; head -10 "$f"; echo; done
```

Expected: 每个文件以 `---` 开头，包含 `name:`、`description:` 字段

- [ ] **Step 3: 验证所有 Skill 的 name 字段无冲突**

```bash
grep -r "^name:" skills/*/SKILL.md
```

Expected 5 个不同的 name：tech-discover, tech-analyze, tech-tutorial, tech-review, tech-research

- [ ] **Step 4: Final commit**

```bash
git add -A
git status
# 确认无未追踪文件后
git commit -m "chore: final verification of tech research skill pipeline"
```
