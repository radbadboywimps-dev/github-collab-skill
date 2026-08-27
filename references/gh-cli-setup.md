# GitHub CLI (gh) 安装与认证

## 定位

gh CLI 是 MCP 的兜底通道，用于 MCP 不支持的操作：
- 修改仓库设置（可见性、描述、默认分支等）
- Release 附件上传
- CI/CD（GitHub Actions）查询与触发
- 仓库克隆（带认证）
- 其他 GitHub API 操作（`gh api` 万能直通）

**不主动安装，懒加载**：只有当 MCP 无法完成某个操作时才引导安装和认证，不在 skill 启动时检查，不打扰不需要的用户。

## 按需引导流程（MCP 不支持某操作时执行）

```
1. 检测 gh 是否已安装（不能只试 PATH，要检查非标准路径）
   ├─ 先试 gh --version
   ├─ 不可用则按平台查找常见路径：
   │   Windows: ~/gh-cli/bin/gh.exe、~/.gh-cli/bin/gh.exe、C:\Program Files\GitHub CLI\gh.exe
   │   macOS:   /opt/homebrew/bin/gh、/usr/local/bin/gh
   │   Linux:   /usr/bin/gh、/usr/local/bin/gh
   ├─ 在非标准路径找到 → 加入当前会话 PATH，跳到步骤 3
   └─ 完全找不到 → 步骤 2

2. 自动下载安装（对用户透明，无需手动操作）
   ├─ Windows：优先 winget，失败则下载便携版到 ~/gh-cli/ 并加入 PATH
   ├─ macOS：brew install gh
   └─ Linux：apt/官方仓库安装
   安装耗时约 10-30 秒，告知用户"正在安装 GitHub CLI..."

3. gh auth status 检查是否已认证
   ├─ 已认证 → 直接执行原操作
   └─ 未认证 → 步骤 4

4. 启动 OAuth Device Flow 认证
   ├─ 请求设备码，拿到 user_code
   ├─ 告知用户："需要授权一下 GitHub，打开 https://github.com/login/device 输入验证码 XXXX-XXXX"
   ├─ 后台轮询等待用户完成（最多 15 分钟）
   └─ 认证成功后自动继续执行原操作

5. 操作完成，告知结果
   认证一次后长期有效，以后不再打扰
```

**关键原则**：
- 安装过程全自动，用户不需要打开终端或手动下载
- 认证是唯一需要用户介入的环节，且只在第一次需要 gh 时发生
- 验证码和用户目标直接关联（如"公开仓库需要授权"），不突兀
- 如果用户拒绝认证，回退到告知手动操作步骤（给 GitHub 设置页链接），不阻塞任务

## 安装

### 优先：winget
```powershell
winget install --id GitHub.cli --silent --accept-source-agreements --accept-package-agreements
```

### 降级：便携版（winget 不可用时）
```powershell
$ghDir = "$env:USERPROFILE\gh-cli"
New-Item -ItemType Directory -Path $ghDir -Force | Out-Null
$release = Invoke-RestMethod -Uri "https://api.github.com/repos/cli/cli/releases/latest" -UseBasicParsing
$asset = $release.assets | Where-Object { $_.name -match "windows_amd64.zip$" } | Select-Object -First 1
Invoke-WebRequest -Uri $asset.browser_download_url -OutFile "$ghDir\gh.zip" -UseBasicParsing
Expand-Archive -Path "$ghDir\gh.zip" -DestinationPath $ghDir -Force
Remove-Item "$ghDir\gh.zip"
# 加入用户 PATH（永久）和当前会话 PATH
$binDir = (Get-ChildItem -Path $ghDir -Recurse -Filter "gh.exe").Directory.FullName
$currentPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($currentPath -notlike "*$binDir*") {
    [Environment]::SetEnvironmentVariable("Path", "$currentPath;$binDir", "User")
}
$env:Path += ";$binDir"
```

### macOS / Linux
```bash
# macOS
brew install gh
# Linux (Debian/Ubuntu)
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list
sudo apt update && sudo apt install gh
```

## 认证（OAuth Device Flow）

**不要用 `gh auth login --web`**——它是交互式的，在 agent 环境中会卡住（等待 Enter 和浏览器回调）。
直接走 GitHub OAuth Device Flow：

### PowerShell 实现
```powershell
$clientId = "178c6fc778ccc68e1d6a"  # gh CLI 公开 OAuth client ID

# 1. 请求设备码
$body = "client_id=$clientId&scope=repo+read:org+gist+workflow"
$raw = Invoke-WebRequest -Uri "https://github.com/login/device/code" -Method POST -Body $body -ContentType "application/x-www-form-urlencoded" -UseBasicParsing
$text = [System.Text.Encoding]::ASCII.GetString($raw.Content)
# 解析 device_code、user_code、verification_uri、interval
$deviceCode = [regex]::Match($text, "device_code=([^&]+)").Groups[1].Value
$userCode = [regex]::Match($text, "user_code=([^&]+)").Groups[1].Value
$interval = [regex]::Match($text, "interval=(\d+)").Groups[1].Value

# 2. 告诉用户去验证
Write-Host "请打开 https://github.com/login/device 输入验证码：$userCode"

# 3. 轮询等待授权
$token = $null
for ($i = 0; $i -lt 30; $i++) {
    Start-Sleep -Seconds $interval
    $pollBody = "client_id=$clientId&device_code=$deviceCode&grant_type=urn:ietf:params:oauth:grant-type:device_code"
    $pollRaw = Invoke-WebRequest -Uri "https://github.com/login/oauth/access_token" -Method POST -Body $pollBody -ContentType "application/x-www-form-urlencoded" -UseBasicParsing
    $pollText = [System.Text.Encoding]::ASCII.GetString($pollRaw.Content)
    if ($pollText -match "access_token=([^&]+)") {
        $token = $Matches[1]
        break
    }
    if ($pollText -match "error=expired_token") { break }
}

# 4. 用 token 登录 gh（token 存入系统 keyring，后续不可见）
if ($token) {
    $token | gh auth login --with-token
    gh auth status
}
```

### 注意事项
- 验证码有效期约 15 分钟，轮询间隔按返回的 `interval`（通常 5 秒）
- token 存入系统 keyring（Windows Credential Manager / macOS Keychain / libsecret），agent 无法再次读取
- scope 说明：`repo`（仓库读写）、`read:org`（组织读取）、`gist`（gist）、`workflow`（Actions）
- 认证一次即可长期使用，除非手动 `gh auth logout` 或 token 过期

## 常用命令

### 仓库管理
```bash
gh repo view <owner>/<repo> --json visibility,url    # 查看仓库信息
gh repo edit <owner>/<repo> --visibility public      # 公开仓库
gh repo edit <owner>/<repo> --visibility private     # 私有仓库
gh repo edit <owner>/<repo> --description "xxx"      # 改描述
gh repo clone <owner>/<repo>                          # 克隆
gh repo list                                          # 列出自己的仓库
gh repo delete <owner>/<repo>                         # 删除（需确认）
```

### Release
```bash
gh release create v1.0.0 --title "v1.0.0" --notes "更新内容"
gh release create v1.0.0 ./dist/*.exe ./dist/*.zip   # 带附件
gh release list
gh release download v1.0.0
gh release upload v1.0.0 ./extra-file.zip            # 追加附件
```

### Pull Request
```bash
gh pr list
gh pr create --title "xxx" --body "xxx"
gh pr checkout 123
gh pr merge 123 --squash
gh pr checks 123                                     # 查看CI状态
```

### CI/CD (Actions)
```bash
gh run list
gh run view <run-id>
gh run watch <run-id>
gh workflow list
gh workflow run <workflow-id>
```

### Issue
```bash
gh issue list
gh issue create --title "xxx" --body "xxx"
gh issue close 123
```

### API 直通（任何 MCP 和 gh 都没有封装的操作）
```bash
gh api /repos/<owner>/<repo> --method PATCH -f visibility=public
gh api /user
```

## 三通道分工

| 通道 | 用途 | 认证 |
|------|------|------|
| 本地 git | commit、branch、stash、merge、rebase、push、pull | GCM / SSH key |
| MCP | 建仓库、PR、Issue、代码搜索、文件上传 | OAuth（内置） |
| gh CLI | 仓库设置、Release 附件、CI/CD、API 直通 | Device Flow → keyring |

优先用最简单的通道：本地 git 能做的用 git，MCP 能做的用 MCP，两者都不行的用 gh。
