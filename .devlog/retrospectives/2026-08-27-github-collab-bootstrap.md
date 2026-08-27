# 复盘：GitHub 协作体系从 0 到 1 搭建

> 时间：2026-08-27 | 周期：2026-08-27 单日

## 概述

今天从"检查 GitHub MCP 是否连上"开始，最终搭建了一套完整的 AI 辅助开发协作体系：3 个 GitHub 仓库、2 个可复用 Skill（github-collab + dev-journal）、本地 git 环境配置、故障降级机制，并完成了视频下载器项目的首次上传。

## 做了什么

### 新增功能/产出

| # | 产出 | 说明 |
|---|------|------|
| 1 | video-downloader 仓库 | 视频下载器项目上传到 GitHub（私有），含 .gitignore、README、config.example.json、AGENTS.md |
| 2 | github-collab skill | 标准化 GitHub 协作流程 Skill，含分支策略、commit 规范、PR 流程、安全红线、故障降级、自扩展机制 |
| 3 | dev-journal skill | 开发记录与内容创作 Skill，支持快速记录、阶段复盘、多平台内容生成 |
| 4 | 本地 git 环境 | MinGit 2.47.1 安装、credential.helper 配置、user.name/email 设置 |
| 5 | 故障降级规则 | 网络失败→MCP 降级、认证排查、PowerShell 兼容、历史分叉处理 |
| 6 | 记录钩子 | github-collab 与 dev-journal 通过 DEVLOG.md 协作 |

### 关键决策

**1. 仓库设为私有**
- 背景：视频下载器涉及平台接口解析和 Cookie 处理逻辑
- 决定：三个仓库全部私有
- 理由：代码含平台逆向逻辑，公开可能有合规风险；Skill 本身也还在迭代初期

**2. 用 MCP 逐文件上传替代 git push**
- 背景：本地 git 无 GitHub 凭证，MinGit 不带 credential-manager
- 选项：配置 GCM 认证 / 用 PAT / MCP 上传
- 决定：先用 MCP push_files/create_or_update_file 上传，同时配置本地 git 环境
- 结果：MCP 通道稳定可用，但本地 commit 与远程 SHA 不同导致历史分叉，后续需 fetch+reset 同步

**3. Skill 拆成两个而非一个**
- 背景：用户想把开发过程内容化（图文/视频/IP），但这不是代码协作的核心功能
- 选项：全塞进 github-collab / 独立 dev-journal / 先不做
- 决定：github-collab 只加轻量记录钩子，dev-journal 独立成 Skill，通过 DEVLOG.md 协作
- 理由：关注点分离；dev-journal 可独立用于非代码项目；保持 github-collab 精简

**4. dev-journal 先轻量验证**
- 背景：用户自己也不确定"会不会真的用这些记录"
- 决定：不搞复杂目录结构，默认一句话记录到 DEVLOG.md，用一段时间再决定是否扩展
- 理由：记录本身不产生价值，产出内容才产生价值；先验证习惯再建设能力

**5. Skill 自扩展机制**
- 背景：用户希望 Skill 能在使用中自动完善，不想手动改
- 决定：复制→修改→验证→替换，带备份和回滚
- 理由：直接改原文件有风险；复制后验证通过再替换更安全

## 踩过的坑

### 1. MinGit 不带 Git Credential Manager
- 现象：配置 credential.helper=manager 后，git push 报 "could not read Username"
- 排查：检查 MinGit 安装目录，发现没有 git-credential-manager.exe
- 根因：MinGit 是精简版，不包含 GCM
- 解决：从 GCM 官方 GitHub 下载单独安装
- 预防：Skill 中补充 MinGit 用户需单独安装 GCM 的说明

### 2. PowerShell 中 git 进度输出到 stderr
- 现象：git push 成功了，但 Bash 工具把 stderr 当作错误，报 NativeCommandError
- 排查：检查输出内容发现有 `-> main` 和 exit code 0，实际成功
- 根因：git 把进度信息输出到 stderr（设计如此），PowerShell 的 NativeCommandError 把 stderr 当异常
- 解决：判断成功看输出内容和 $LASTEXITCODE，不看 stderr 是否有内容
- 预防：已写入 Skill 故障降级规则

### 3. GitHub 网络不稳定
- 现象：git push 间歇性失败，报 Connection was reset / Could not resolve host
- 排查：Test-NetConnection 显示 443 端口时通时断，无代理
- 根因：GitHub 在国内访问不稳定，时好时坏
- 解决：网络不通时降级到 MCP 通道（走豆包服务端，不受本地网络影响）
- 预防：已写入 Skill 故障降级规则；建议用户有条件时配置代理

### 4. MCP 推送后本地 git 历史分叉
- 现象：通过 MCP push_files 上传后，本地 git log 和远程 log 的 SHA 不同
- 排查：MCP 上传是通过 GitHub API 直接创建 commit，不经过本地 git，所以 SHA 不同
- 根因：两套提交通道（本地 git 和 MCP API）各自生成 commit，历史不一致
- 解决：git fetch origin && git reset --hard origin/main 对齐远程
- 预防：已写入 Skill 故障降级规则；网络恢复后第一时间同步

### 5. 大文件读取截断
- 现象：用 Get-Content -Raw 读取 app.py（36KB）时输出被截断
- 根因：工具输出有长度限制
- 解决：分段读取或用 Read 工具的 offset/limit 分页
- 预防：大文件上传时分段读取确认完整性

## 数据与成果

- 3 个 GitHub 仓库（全部私有）
- video-downloader：10 次 commit，含完整源码 + 项目文档
- github-collab-skill：3 次功能 commit（初始化 → 故障降级 → 记录钩子）
- dev-journal-skill：1 次初始 commit（SKILL.md + 2 个 references）
- 本地 git 环境：MinGit 2.47.1 + GCM 2.9.1 + OAuth 认证完成
- 2 个可复用 Skill，后续任何开发对话自动加载

## 反思

**做得好的：**
- 从实际操作中发现问题、立即补进 Skill，形成了"使用→发现→完善"的闭环
- 故障降级机制在今天就被验证了多次（网络断了→MCP 顶上），不是纸上谈兵
- Skill 拆分决策正确：github-collab 保持精简，dev-journal 独立演进
- 安全意识到位：.gitignore 排除了 Cookie、config、exe、视频等敏感/大文件

**可以改进的：**
- 视频下载器还有 4 个文件（app.py、style.css、bilibili.js、common.js）因截断问题未上传完，需要补传
- MinGit + GCM 的安装配置流程较长，Skill 中可以写得更具体（或写个自动化脚本）
- MCP 上传大文件时缺少完整性校验机制，应该在 push 后校验文件大小
- 今天网络反复断了多次，如果一开始就配好代理可以省很多时间

**意外收获：**
- "故障降级"本来是事后补充的规则，结果成了今天使用频率最高的部分
- dev-journal 的想法是在做 github-collab 的过程中自然长出来的，不是预先设计的
- Skill 自扩展机制第一次运行就成功了（故障降级规则的补充）

## 下一步

- [ ] 补传 video-downloader 剩余 4 个文件
- [ ] 网络稳定时配置 git 代理（如果有代理软件）
- [ ] 在实际开发中试用 dev-journal，验证记录习惯
- [ ] github-collab Skill 后续扩展方向：多系统环境（macOS/Linux）、Git LFS、CI/CD、项目模板
- [ ] dev-journal Skill 后续扩展方向：更多平台内容模板、内容风格学习、素材自动提取
