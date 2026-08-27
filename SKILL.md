---
name: github-collab
description: 标准化GitHub协作开发流程。当用户进行软件开发、写代码、提交代码、推送、建仓库、建分支、Pull Request、代码审查、合并、版本发布、打tag、release、Issue管理、git操作、回退代码、暂存改动、解决冲突、初始化项目等任务时使用。触发词包括但不限于：开发、写代码、改代码、提交、推送、推上去、建仓库、建分支、提PR、合并、发版、release、打tag、issue、bug、代码审查、git、撤销、回退、stash、冲突、clone、fork。
---

# GitHub 协作流程

## 启动检查（每次触发时自动执行）

1. 确定工作目录：用户指定的项目路径，或当前对话中正在操作的目录
2. 检查是否为 git 仓库：
   - 不是 → 询问用户是否初始化（走"新项目初始化"流程）
   - 是 → 执行 `git status` 和 `git branch --show-current` 确认状态
3. 检查项目根目录是否有 AGENTS.md：
   - 有 → 读取并遵循其中的项目约定
   - 没有 → 扫描项目结构，按 [references/agents-template.md](references/agents-template.md) 生成一份，展示给用户确认后写入
4. 如果是已有仓库，开始工作前先 `git pull` 拉取最新代码

## 工具分工

| 操作类型 | 使用工具 |
|----------|----------|
| 本地文件暂存、提交、查看差异、日志、分支切换、stash | 本地 git 命令 |
| 推送、拉取（push/pull/fetch） | 本地 git 命令 |
| 创建仓库、Fork | GitHub MCP（create_repository / fork_repository） |
| Pull Request 创建、审查、合并 | GitHub MCP（create_pull_request / pull_request_review_write / merge_pull_request） |
| Issue 创建、评论、标签 | GitHub MCP（issue_write / add_issue_comment） |
| Release 发布 | GitHub MCP（get_latest_release / list_releases） |
| 代码搜索、仓库搜索 | GitHub MCP（search_code / search_repositories） |
| GitHub 上的文件创建/修改（非本地仓库场景） | GitHub MCP（create_or_update_file / push_files） |

原则：涉及本地工作区文件的操作用 git 命令；涉及 GitHub 平台功能的用 MCP。

## 分支策略

- `main`：稳定主干，始终可运行，**禁止直接提交**
- `feat/*`：新功能，如 `feat/bilibili-8k`
- `fix/*`：Bug 修复，如 `fix/cctv-m3u8`
- `chore/*`：构建、配置、依赖
- `docs/*`：文档
- `refactor/*`：重构（不改变功能）
- `perf/*`：性能优化
- `test/*`：测试相关

分支名用英文小写，单词间用连字符。一个分支只做一件事。

## Commit 规范

格式：`<类型>: <简述>`

类型：feat / fix / chore / docs / refactor / style / test / perf

- 简述用中文，50字以内，结尾不加句号
- 正文（可选）说明"为什么改"，不描述"改了什么"（diff 已说明）
- 关联 Issue 时写 `(#123)`，关闭 Issue 写 `Closes #123`

示例：
```
feat: B站插件支持8K分辨率下载
fix: 修复CCTV m3u8地址解析失败的问题
```

## 标准开发流程

### 开始工作
1. `git status` 确认工作区干净
2. `git pull` 拉取最新代码
3. 从 main 创建工作分支：`git checkout -b feat/xxx`

### 开发中
- 小步提交，每个 commit 是一个完整逻辑单元
- 临时切换任务时用 `git stash` 暂存（详见 [references/git-operations.md](references/git-operations.md)）

### 提交前检查
- `git status` 确认无误加的文件
- `git diff` 检查改动内容
- 确认无硬编码密钥、token、个人路径
- 确认无大文件（>50MB）、构建产物
- 确认 .gitignore 已覆盖敏感文件和产物目录
- 如果改了项目架构或依赖，同步更新 AGENTS.md

### 推送与合并
1. `git push -u origin <分支名>`
2. 通过 MCP 创建 Pull Request
3. PR 描述包含：改了什么、为什么、怎么验证的
4. 合并前过一遍下方自查清单
5. 合并后切回 main、pull、删除本地和远程分支

### PR 自查清单
- [ ] 代码能运行/构建通过
- [ ] 没有误提交的文件（密钥、配置、产物）
- [ ] commit message 清晰规范
- [ ] 如改了架构/依赖，AGENTS.md 已更新
- [ ] 个人项目小改动可自行确认；大功能或协作项目需等待 review

### 个人项目简化
- 个人项目的小改动（修typo、改配置、小bugfix）：可在 feat/fix 分支上直接 push 后合并，不必走完整 review
- 大功能、架构变更、涉及多文件的重构：仍走 PR，留记录
- 协作者 ≥2 人的项目：所有改动必须 PR + review

## 新项目初始化

1. 通过 MCP `create_repository` 创建仓库（默认私有，除非用户要求公开）
2. 在本地项目目录 `git init`
3. 根据语言/框架生成 .gitignore（Python/Node/Go/Java 等常见模板）
4. 生成 AGENTS.md（按 [references/agents-template.md](references/agents-template.md)）
5. 初始提交：`git add . && git commit -m "chore: 初始化项目"`
6. 关联远程并推送：`git remote add origin <url> && git push -u origin main`

## 安全红线

- 永远不提交：密码、API Key、Token、.env、私钥、Cookie/Session 文件
- 永远不提交：>50MB 二进制文件（视频、exe、依赖二进制）；需要时引导用户用 Git LFS
- 永远不提交：node_modules/、__pycache__/、build/、dist/ 等产物
- 发现误提交密钥：立即告知用户轮换密钥，再清理 git 历史
- 仓库默认私有，用户明确要求才公开
- 破坏性操作（force push、删除分支、重置历史）必须说明后果并经用户确认

## 故障降级与恢复

### 本地 git push 网络失败
当 `git push` 因网络问题失败（连接超时、连接重置、无法解析 host、443 端口不通）时：
1. 确认是网络问题而非认证问题（检查报错信息含 `Connection was reset` / `Could not resolve host` / `Failed to connect` / `Recv failure`）
2. 降级到 MCP `push_files` 批量上传本次改动的文件（不要逐文件 `create_or_update_file`）
3. 上传后用 MCP `get_file_contents` 校验关键文件大小与本地一致
4. 告知用户：本地 commit 仍在，网络恢复后需在项目目录执行 `git fetch origin && git reset --hard origin/main` 同步（本地未推送的 commit 会被远程版本替代，因内容一致无损失）

### 认证失败
当 git 操作返回 403/Authentication failed：
1. 检查 credential helper：`git config --global credential.helper`
   - Windows：应为 `manager`（Git Credential Manager）
   - macOS：应为 `osxkeychain`
   - Linux：应为 `libsecret` 或 `store`
2. 未配置则设置对应 helper
3. 引导用户完成认证：GCM 会弹出浏览器 OAuth 授权；无 GUI 环境用 Personal Access Token
4. 认证完成后重试

### Windows PowerShell 注意事项
- git 的进度信息输出到 stderr，不代表命令失败
- 判断成功要看输出内容（`-> main`、`up to date`、`[new branch]`）和 `$LASTEXITCODE -eq 0`
- 不要因 stderr 有输出就判定失败

### 本地与远程历史分叉
当本地 commit 通过 MCP 推送到远程后（SHA 不同），本地 git 历史与远程分叉：
- 不要 force push
- 用 `git fetch origin && git reset --hard origin/main` 对齐远程
- 如果本地有 MCP 未上传的 commit，先 `git format-patch` 备份再 reset，之后 `git am` 恢复

## 撤销与回退

常见场景速查（完整说明见 [references/git-operations.md](references/git-operations.md)）：

| 场景 | 命令 |
|------|------|
| 丢弃工作区改动 | `git checkout -- <file>` 或 `git restore <file>` |
| 撤销最后一次commit（保留改动） | `git reset --soft HEAD~1` |
| 撤销最后一次commit（丢弃改动） | `git reset --hard HEAD~1` |
| 已push的commit要撤销 | `git revert <sha>`（不要 force push） |
| 暂存当前改动 | `git stash` / 恢复 `git stash pop` |

## 版本发布

- 语义化版本：MAJOR.MINOR.PATCH（如 1.2.3）
  - PATCH：bug 修复
  - MINOR：新功能，向后兼容
  - MAJOR：不兼容的重大变更
- 打 tag：`git tag v1.2.3 && git push origin v1.2.3`
- 通过 MCP 创建 GitHub Release，附 changelog（基于 commit 历史生成）

## Skill 自扩展

本 Skill 可在使用中自动完善。当遇到未覆盖的场景或用户提出新的协作规范时，按 [references/skill-evolution.md](references/skill-evolution.md) 的流程自动更新本 Skill，无需用户手动编辑。用户也可直接说"给skill加个规则"或"回滚skill"。
