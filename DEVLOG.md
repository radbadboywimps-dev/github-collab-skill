# DEVLOG

## 2026-08-27 [decision] 收窄Skill触发边界，不在纯编码时打扰用户

- 原 description 包含"开发、写代码、改代码、bug"等泛词，导致用户在 vibe coding 时误触发，自动跑 git status/pull 打扰开发节奏
- 改为只在明确涉及 git/GitHub 操作时触发（提交、推送、建仓库、PR等），纯编码/调试/重构时保持沉默
- 启动检查标题从"每次触发时自动执行"改为"执行 git/GitHub 操作前"
- 记录钩子也加了限制：仅在 Skill 已激活时生效

## 2026-08-27 [decision] 大文件上传逐批探测网络恢复

- 原规则降级 MCP 后一口气传完，不检测网络是否恢复，浪费了更快的本地上传通道
- 改为每批上传后快速探测 github.com:443（3秒超时），通了立即切回本地 git push 剩余文件
- 不通只提醒一次"检查VPN/代理"，不重复唠叨

## 2026-08-27 [decision] 首次上传后询问公开+license推荐

- 新项目初始化流程末尾加了公开询问：告知仓库地址，问要不要公开
- 公开时根据项目类型推荐 license（个人项目默认 MIT），只说一句人话解释，用户想了解再展开
- 新增 License 章节，含推荐逻辑和标准话术
- 给 github-collab-skill 和 dev-journal-skill 都加了 MIT LICENSE

## 2026-08-27 [pitfall] Skill规则不能导致agent降智

- 故障降级章节新增核心原则："Skill 规则是指导不是枷锁"
- 网络失败时要主动诊断原因（DNS/端口/代理）并给建议，不能机械重试或干等
- 任务必须完成不留尾巴，本地分叉同步是 agent 的事不是用户的待办

## 2026-08-27 [milestone] github-collab-skill 首次公开 + gh CLI 第三通道打通

- 仓库从私有切换为公开：https://github.com/radbadboywimps-dev/github-collab-skill
- MCP 不支持改仓库可见性，安装了 gh CLI v2.98.0 并通过 OAuth Device Flow 完成认证
- 踩坑：`gh auth login --web` 在 agent 后台环境中会卡死（等待交互式 Enter），改用直接调用 GitHub Device Flow API 获取验证码，轮询拿 token 后 `gh auth login --with-token` 写入 keyring
- 新增 references/gh-cli-setup.md：安装（winget+便携版降级）、Device Flow 认证脚本、常用命令、三通道分工
- 三通道体系成型：本地 git（提交/分支）→ MCP（建仓库/PR/Issue/搜索）→ gh CLI（仓库设置/Release附件/CI/API直通）

## 2026-08-27 [decision] gh CLI改为懒加载，不在启动时检查安装

- 原设计把 gh CLI 作为第三通道，但没有明确什么时候安装和认证，可能导致 skill 启动时就检查 gh，打扰不需要的用户
- 改为懒加载：只有 MCP 不支持某操作时才自动下载安装 gh，安装对用户透明；认证是唯一需要用户介入的环节，且验证码与操作目标直接关联（如"公开仓库需要授权"）
- 用户拒绝认证时回退到手动操作指引，不阻塞任务
- gh-cli-setup.md 新增完整的按需引导流程图

## 2026-08-28 [decision] 借鉴Claude Code插件，整合PR规模警告和自动清理分支

- 拆解了 Dee-0503/git-collaboration-workflow 项目（Claude Code插件，13个Hook+12个Skill+2个Agent）
- 值得借鉴的：PR超20文件警告拆分、合并后自动清理分支、确定性脚本校验、模式开关
- 不适合照搬的：系统级Hook拦截（豆包无此API）、integration分支层（个人项目太重）、云端审查（需Actions+中继）
- 本次整合P0两项：提交前检查加`git diff --stat`规模警告、合并后自动清理本地+远程分支
- 密钥白名单（.secretsignore）暂不做——AI上下文判断本身就能区分真密钥和文档示例，不像正则扫描会误报

## 2026-08-28 [decision] 流程分级：简单流程为默认，完整流程按需触发

- 原设计强制所有项目走分支+PR流程，对个人vibe coding项目太重
- 改为两档：简单流程（默认，直接在main上提交推送，只做静默安全检查）和完整流程（建分支+PR+自查+清理）
- 判断优先级：用户本次语气 > AGENTS.md配置 > 多作者检测建议 > 默认简单
- 用户说"直接上传"走简单流程，但安全检查（密钥/大文件/.env）始终静默执行——"直接"是少废话不是少安全
- 简单流程下不主动生成AGENTS.md，不强制pull（失败不阻塞）
- 踩坑：MCP上传后本地分叉，git reset --hard同步远程时覆盖了未提交的P1改动，只能重新应用——教训：同步前先stash或commit本地改动
