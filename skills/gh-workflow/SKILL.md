---
name: gh-workflow
description: 根据 user 需求自动创建各类 GitHub Action workflow 文件。当 user 提到 "创建 workflow"、"GitHub Actions"、"CI/CD 配置"、"自动部署"、"npm 发布流程"、"Docker 构建流水线"、"定时任务 workflow"、"PR 检查"、"release workflow"、"构建流水线"、"自动化流水线"、"添加 CI"、"配置 CD" 等与 GitHub Actions 相关的需求时触发本技能。即使用户没有明确说 "workflow"，只要涉及 GitHub 上的自动化构建、测试、部署、发布等场景，也应触发。
---

# gh-workflow — GitHub Action Workflow 生成器

## 概述

本技能根据用户描述的需求，自动生成符合最佳实践的 GitHub Action workflow YAML 文件。支持常见的 CI/CD 场景，包括持续集成测试、自动部署、包发布、Docker 构建、定时任务、PR 检查、Release 工作流等。

核心原则：生成的 workflow 文件必须**直接可用**，不需要用户手动补充关键配置。所有模板基于 GitHub Actions 官方推荐实践。

## 执行流程

按以下步骤执行：

### 第一步：理解用户需求

分析用户输入，确定需要创建的 workflow 类型。识别以下信息：

- **workflow 类型**：用户想要什么类型的自动化（测试、部署、发布等）
- **技术栈**：项目使用的语言/框架（Node.js、Python、Go、Java、Docker 等）
- **触发条件**：何时运行（push、PR、tag、定时、手动等）
- **目标环境**：部署到哪里（npm、Docker Hub、AWS、Vercel 等）

如果用户描述不够明确，主动提问以补全信息。提问应简洁，一次最多问 2-3 个关键问题。

### 第二步：检查项目上下文

在生成 workflow 之前，先检查项目根目录的现有配置，以便生成更贴合项目的 workflow：

1. 读取 `package.json`（如果存在）— 获取 Node.js 项目的 scripts、依赖、包名
2. 读取 `Dockerfile`（如果存在）— 获取 Docker 构建配置
3. 读取 `.github/workflows/` 目录（如果存在）— 避免与已有 workflow 冲突或重复
4. 读取项目根目录文件列表 — 识别项目类型（前端、后端、monorepo 等）

这些信息帮助生成更精确的 workflow，但不是必须的——如果项目还没有初始化，也能根据用户描述生成通用模板。

### 第三步：选择并生成 workflow

根据用户需求选择对应的 workflow 类型，生成 YAML 文件。参考 `references/workflow-patterns.md` 中的模板库，每种类型都有标准模板。

支持的 workflow 类型：

| 类型 | 适用场景 | 典型触发条件 |
|------|---------|-------------|
| CI 测试 | 自动运行测试、lint、类型检查 | push、PR |
| CD 部署 | 自动部署到服务器/云平台 | push to main、tag |
| npm 发布 | 发布 Node.js 包到 npm registry | tag v* |
| Docker 构建 | 构建并推送 Docker 镜像 | tag v* |
| 定时任务 | 周期性执行检查、清理、同步 | schedule cron |
| PR 检查 | PR 提交时的自动化检查 | pull_request |
| Release | 创建 GitHub Release 并上传产物 | tag v*、手动触发 |
| 自定义 | 以上类型组合或特殊需求 | 按用户要求 |

生成时遵循以下规则：

- 文件放在 `.github/workflows/` 目录下
- 文件名使用 kebab-case 格式（如 `ci-test.yml`、`publish-npm.yml`）
- 包含清晰的注释说明每个 step 的用途
- 使用最新稳定版本的 Actions（checkout@v4、setup-node@v4 等）
- 设置合理的 `permissions`，遵循最小权限原则

### 第四步：输出与确认

生成 workflow 文件后，向用户展示：

1. **文件路径**：生成的 workflow 文件位置
2. **触发方式**：说明如何触发这个 workflow
3. **前置配置**（如有）：需要配置的 Secrets、环境变量等
4. **使用提示**：如何验证 workflow 是否正常工作

如果生成了多个 workflow 文件，逐一说明每个文件的用途和触发方式。

## 质量约束

### YAML 格式正确

生成的 YAML 必须语法正确、可以直接使用。避免常见错误：缩进不一致、引号未配对、布尔值格式错误。YAML 中字符串值包含特殊字符时使用引号包裹。

### 版本固定

引用的第三方 Actions 必须指定主要版本号（如 `actions/checkout@v4`），不要使用 `@main` 或 `@master`。这确保 workflow 行为稳定，不会因上游更新而意外中断。

### 最小权限

每个 job 必须设置最小必要的 `permissions`。例如，CI 测试只需要 `contents: read`，创建 Release 才需要 `contents: write`。不要使用 `permissions: write-all`。

### 错误处理

workflow 中应包含基本的错误处理：
- 关键步骤使用 `if` 条件判断前置依赖是否成功
- 上传构建产物（artifacts）以便调试失败原因
- 设置合理的 timeout（默认 30 分钟）

## 输出规范

### 文件路径

所有 workflow 文件放在项目根目录的 `.github/workflows/` 下：

```
.github/workflows/
├── ci.yml              # CI 测试
├── deploy.yml          # CD 部署
├── publish-npm.yml     # npm 发布
├── docker.yml          # Docker 构建
├── scheduled.yml       # 定时任务
└── release.yml         # Release
```

### 文件格式

- 格式：YAML
- 编码：UTF-8
- 缩进：2 个空格（GitHub Actions 标准）
- 注释：中文注释说明关键配置项

### Secrets 配置提示

如果 workflow 需要 Secrets（如 `NPM_TOKEN`、`DOCKER_PASSWORD`、`AWS_ACCESS_KEY_ID`），在输出末尾附带配置说明：

```
前置配置：
1. 在 GitHub 仓库页面 Settings → Secrets → Actions 中添加以下 Secret：
   - NPM_TOKEN：npm 发布令牌（npm token create 生成）
2. 或使用 gh CLI：gh secret set NPM_TOKEN
```

## 进阶场景

### Monorepo 支持

如果检测到项目是 monorepo（存在 `packages/` 目录、`turbo.json`、`lerna.json`、`pnpm-workspace.yaml` 等标志），workflow 应该：

- 使用 `paths` 过滤器，只在相关目录变更时触发对应的 job
- 并行执行各子包的构建和测试
- 正确处理包之间的依赖关系

### 矩阵构建

当用户需要跨版本/跨平台测试时，使用 matrix 策略：

```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    os: [ubuntu-latest, macos-latest]
```

### 缓存优化

对于安装依赖的步骤，启用缓存以加速构建：

- Node.js：使用 `actions/setup-node` 内置缓存（`cache: 'npm'`）
- Docker：使用 `docker/build-push-action` 的 cache 配置
- 通用：使用 `actions/cache`

参考 `references/workflow-patterns.md` 获取各类 workflow 的完整模板和详细配置说明。
