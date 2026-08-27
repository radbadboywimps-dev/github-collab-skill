# 跨平台命令参考

SKILL.md 中只写逻辑，平台相关命令查此表。AI 根据当前操作系统选择对应命令。

## git 可执行文件检测

按优先级查找，找到第一个存在的即可：

| 优先级 | Windows | macOS | Linux |
|--------|---------|-------|-------|
| 1 | PATH 中的 `git`（`git --version`） | 同左 | 同左 |
| 2 | `C:\Program Files\Git\cmd\git.exe` | `/opt/homebrew/bin/git`（Apple Silicon） | `/usr/bin/git` |
| 3 | `C:\Program Files (x86)\Git\cmd\git.exe` | `/usr/local/bin/git`（Intel） | `/usr/local/bin/git` |
| 4 | `%USERPROFILE%\mingit\cmd\git.exe`（便携版） | `/usr/bin/git` | — |

找到便携版 git 时，将其目录加入当前会话 PATH：
- PowerShell: `$env:Path = "$env:Path;<git目录>"`
- bash: `export PATH="$PATH:<git目录>"`

找不到时引导安装：
- Windows: `winget install Git.Git` 或下载便携版 MinGit
- macOS: `brew install git`
- Linux: `sudo apt install git`（Debian/Ubuntu）或 `sudo yum install git`（CentOS/RHEL）

## 网络探测（github.com:443，3 秒超时）

### Windows PowerShell
```powershell
try {
    $client = New-Object System.Net.Sockets.TcpClient
    $task = $client.ConnectAsync("github.com", 443)
    if ($task.Wait(3000)) { $client.Connected } else { $false }
} catch { $false } finally { $client.Close() }
```

### macOS / Linux (bash)
```bash
timeout 3 bash -c 'echo > /dev/tcp/github.com/443' 2>/dev/null && echo "up" || echo "down"
```

### 跨平台 Python 兜底（三平台通用）
```python
python3 -c "import socket; s=socket.socket(); s.settimeout(3); s.connect(('github.com',443)); s.close(); print('up')" 2>/dev/null || echo "down"
```

## 大文件检查（>50MB）

### Windows PowerShell
```powershell
Get-ChildItem -Recurse -File | Where-Object { $_.Length -gt 50MB } | Select-Object FullName, @{N='SizeMB';E={[math]::Round($_.Length/1MB,1)}}
```

### macOS / Linux (bash)
```bash
find . -type f -size +50M -exec ls -lh {} \; | awk '{print $5, $9}'
```

## 未提交改动检查

三平台通用：
```bash
git status --porcelain
```
输出为空 = 工作区干净。

## 本地额外 commit 检查（远程没有的）

三平台通用：
```bash
git log origin/main..HEAD --oneline
```
输出为空 = 本地没有远程缺失的 commit。

## 文件编码写入（UTF-8 无 BOM）

### Windows PowerShell
```powershell
$content = Get-Content $path -Raw
[System.IO.File]::WriteAllText($path, $content, [System.Text.UTF8Encoding]::new($false))
```

### macOS / Linux (bash)
```bash
# 大多数编辑器默认 UTF-8 无 BOM，直接写入即可
cat > file << 'EOF'
内容
EOF
```

## 系统代理检测

### Windows PowerShell
```powershell
# 检查系统代理
Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' | Select-Object ProxyEnable, ProxyServer
# 检查常见代理端口
foreach ($p in 7890,1080,8080,10809) { Test-NetConnection 127.0.0.1 -Port $p -WarningAction SilentlyContinue -InformationLevel Quiet }
```

### macOS
```bash
# 检查系统代理
scutil --proxy | grep -E 'HTTPEnable|HTTPPort|HTTPSEnable|HTTPSPort'
# 检查常见代理端口
for p in 7890 1080 8080 10809; do nc -z -w1 127.0.0.1 $p 2>/dev/null && echo "port $p open"; done
```

### Linux
```bash
# 检查环境变量代理
echo "http_proxy=$http_proxy https_proxy=$https_proxy"
# 检查常见代理端口
for p in 7890 1080 8080 10809; do nc -z -w1 127.0.0.1 $p 2>/dev/null && echo "port $p open"; done
```

## 环境变量设置

| 操作 | Windows PowerShell | macOS/Linux bash |
|------|-------------------|------------------|
| 当前会话 PATH 追加 | `$env:Path = "$env:Path;<目录>"` | `export PATH="$PATH:<目录>"` |
| 当前会话环境变量 | `$env:VAR = "value"` | `export VAR="value"` |
| git stderr 重定向 | `$env:GIT_REDIRECT_STDERR = '2>&1'` | 不需要（bash 不把 stderr 当错误） |

## 平台判断

AI 可通过以下方式判断当前系统：
- Windows: `$PSVersionTable` 存在且 `$IsWindows` 为 true，或 `uname` 返回 `MINGW*`/`MSYS*`/`CYGWIN*`
- macOS: `uname` 返回 `Darwin`
- Linux: `uname` 返回 `Linux`
