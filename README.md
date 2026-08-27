# github-collab

> 让 AI 助手帮你自动化 GitHub 开发全流程：从建仓库到发版本，从提交到 PR，网络断了自动换通道，密钥大文件自动拦截。

## 这是什么

一个豆包（Doubao）Skill。装上之后，你只需要跟 AI 说"帮我提交代码""建个仓库""发个版本"，它会自动走完规范的 GitHub 协作流程——不需要你记 git 命令，不需要担心误提交密钥，网络不好也能传上去。

## 你只需要说

| 你说 | 它做 |
|------|------|
| "帮我建个仓库传上去" | 建仓库 → 生成 .gitignore → 首次提交 → 推送 → 问你要不要公开 |
| "提交代码" | 检查改动 → 拦截密钥/大文件 → 规范 commit message → 推送 |
| "发个版本" | 打 tag → 生成 changelog → 创建 GitHub Release |
| "帮我公开仓库" | 自动装 gh CLI → 引导授权 → 切换可见性 |
| 网络断了 | 自动降级到 MCP 通道继续传，每批探测网络恢复后切回 git |

## 核心能力

- **零配置开箱即用**：MCP 通道已内置认证，建仓库、PR、Issue、文件上传直接用
- **安全红线**：自动拦截密钥、token、.env、大文件（>50MB）、构建产物，仓库默认私有
- **网络容错**：git push 失败自动降级 MCP 逐批上传，每批探测网络恢复，能切回 git 就切回
- **懒加载 gh CLI**：MCP 不支持的操作（改可见性、Release 附件、CI/CD）自动安装 gh CLI，Device Flow 认证一次长期有效
- **规范自动化**：分支命名、Conventional Commits、PR 自查清单、AGENTS.md 项目约定
- **开发记录**：关键节点提醒记 DEVLOG，方便复盘和内容创作
- **自扩展**：使用中遇到未覆盖的场景自动更新规则，支持回滚

## 安装

将本目录放到豆包 Skill 目录下：

```
<skill_root>/github-collab/
├── SKILL.md              # 核心流程
└── references/
    ├── agents-template.md
    ├── gh-cli-setup.md
    ├── git-operations.md
    └── skill-evolution.md
```

## 三通道架构

```
本地 git（提交/分支/暂存）──网络不通──→ MCP（建仓库/PR/Issue/上传）──不支持──→ gh CLI（仓库设置/Release/CI）
```

每个通道自动降级，任务必须完成，不留尾巴。

## License

MIT
