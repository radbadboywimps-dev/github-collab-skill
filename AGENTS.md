# github-collab Skill

## 项目简介

豆包 AI 助手的 GitHub 协作开发流程 Skill。在任何对话中触发后，为 AI 提供标准化的 Git/GitHub 工作流规范，包括分支策略、Commit 规范、PR 流程、安全检查、AGENTS.md 自动生成和 Skill 自扩展机制。

## 技术栈

- 格式：Markdown（SKILL.md + references/）
- 脚本：Python 3（可选，用于辅助工具）
- 无构建步骤，无运行时依赖

## 目录结构

  github-collab/
  ├── SKILL.md                          # 核心流程规范（触发入口）
  └── references/
      ├── git-operations.md             # Git 操作详解（撤销/暂存/冲突/LFS）
      ├── agents-template.md            # AGENTS.md 模板与自动生成逻辑
      └── skill-evolution.md            # Skill 自扩展与回滚机制

## 开发约定

- SKILL.md 保持精简（<500行），详细内容拆分到 references/
- frontmatter 只包含 name 和 description，description 是触发依据需覆盖所有触发词
- 修改 Skill 时遵循 references/skill-evolution.md 的自扩展流程
- 新增跨平台支持时，命令需区分 Windows / macOS / Linux

## 跨平台开发计划

当前主要在 Windows 环境验证。待扩展：
- macOS：Keychain 凭证管理、路径格式、brew 安装 git
- Linux：gnome-keyring/libsecret、各包管理器安装 git
- 云环境/容器：无浏览器时的 token 认证、SSH key 配置
- 路径处理统一使用正斜杠或 os.path 语义

## Git 约定

- 主分支：main
- 远程仓库：https://github.com/radbadboywimps-dev/github-collab-skill（私有）
- 提交规范遵循本 Skill 自身定义的 Conventional Commits

## 注意事项

- backups/ 目录存放 Skill 修改前的备份，不提交
- 不要在 Skill 文件中写入任何个人 token、密钥或私有仓库 URL 之外的敏感信息
- Skill 安装位置为豆包 `.user_skills/github-collab/`，本仓库是其开发源
