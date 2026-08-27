# github-collab

标准化 GitHub 协作流程的豆包 Skill。

## 功能

- 自动初始化项目（.gitignore、AGENTS.md、首次提交）
- 分支管理与 Commit 规范
- Pull Request 创建与自查
- 版本发布（tag + release）
- 网络故障自动降级到 MCP 上传，逐批探测网络恢复
- 安全检查（密钥、大文件、产物拦截）
- 开发记录钩子（DEVLOG）
- Skill 自扩展（使用中自动完善规则）

## 安装

将本目录放到豆包 Skill 目录下：

```
<skill_root>/github-collab/
├── SKILL.md
└── references/
```

## License

MIT
