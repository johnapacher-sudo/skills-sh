# Skills SH

Claude Code Agent Skills Shell 脚本集合。

## 简介

本项目用于存放和管理 Claude Code Agent 的自定义技能（Skills）Shell 脚本。Skills 允许您扩展 Claude Code 的能力，通过定义特定的触发条件和指令来执行自定义任务。

## 结构

```
skills-sh/
├── skills/              # Skills 脚本目录
│   └── *.sh             # 各类 skill 脚本
├── LICENSE
└── README.md
```

## 使用方法

### 安装 Skill

将脚本复制到 Claude Code 的 skills 目录：

```bash
# 克隆仓库
git clone https://github.com/johnapacher-sudo/skills-sh.git

# 复制需要的 skill 到本地 skills 目录
cp skills-sh/skills/*.sh ~/.claude/skills/
```

### Skill 开发

每个 Skill 脚本应包含以下基本信息：

- **触发条件**：定义何时触发该 skill
- **执行指令**：定义 skill 执行的具体操作
- **描述**：清晰说明 skill 的功能

示例 skill 脚本模板：

```bash
#!/bin/bash
# skill-name: Your Skill Name
# trigger: when the user asks to do something specific
# description: Brief description of what this skill does

# Your script logic here
```

## 许可证

本项目基于 [MIT License](LICENSE) 开源。