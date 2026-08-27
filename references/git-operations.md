# Git 操作详解

## 目录
- [撤销与回退](#撤销与回退)
- [Stash 暂存](#stash-暂存)
- [冲突处理](#冲突处理)
- [分支管理](#分支管理)
- [提交历史整理](#提交历史整理)
- [日志查看](#日志查看)
- [Git LFS（大文件）](#git-lfs大文件)

## 撤销与回退

### 丢弃工作区改动（未 git add）
```bash
# 丢弃单个文件
git restore <file>
git checkout -- <file>

# 丢弃所有改动
git checkout -- .
```

### 撤销暂存（已 git add，未 commit）
```bash
# 取消暂存，保留工作区改动
git restore --staged <file>
git reset HEAD <file>
```

### 撤销最后一次 commit（未 push）
```bash
# 保留改动在工作区（最常用）
git reset --soft HEAD~1

# 保留改动在暂存区
git reset --mixed HEAD~1   # 默认

# 彻底丢弃改动（危险）
git reset --hard HEAD~1
```

### 撤销已 push 的 commit
```bash
# 创建一个反向 commit（安全，推荐）
git revert <commit-sha>
git push

# 不要对已 push 的 commit 用 reset --hard + force push
# 除非是个人项目且确认没有其他人基于该 commit 工作
```

### 回退到某个历史版本
```bash
# 临时查看旧版本（detached HEAD）
git checkout <sha>

# 回退到某个版本并丢弃之后的所有提交（危险）
git reset --hard <sha>
git push --force-with-lease   # 比 --force 安全
```

## Stash 暂存

用于临时保存未完成的改动，切换分支处理其他事情。

```bash
# 暂存当前改动（包括暂存区和工作区）
git stash push -m "正在做xxx功能"

# 暂存并包含未跟踪的新文件
git stash push -u -m "xxx"

# 查看暂存列表
git stash list

# 恢复最近一次暂存并删除
git stash pop

# 恢复指定暂存
git stash pop stash@{2}

# 恢复但不删除暂存记录
git stash apply

# 删除指定暂存
git stash drop stash@{0}

# 清空所有暂存
git stash clear
```

## 冲突处理

当 `git pull`、`git merge`、`git rebase` 发生冲突时：

1. `git status` 查看冲突文件
2. 打开冲突文件，找到标记：
   ```
   <<<<<<< HEAD
   你的改动
   =======
   对方的改动
   >>>>>>> branch-name
   ```
3. 手动编辑，保留正确内容，删除 `<<<`、`===`、`>>>` 标记
4. `git add <冲突文件>`
5. merge 冲突：`git commit`；rebase 冲突：`git rebase --continue`
6. 放弃合并：`git merge --abort` 或 `git rebase --abort`

**原则**：不确定怎么解决时，停下来问用户，不要擅自选择某一方。

## 分支管理

```bash
# 查看所有分支（含远程）
git branch -a

# 创建并切换
git checkout -b feat/xxx
git switch -c feat/xxx

# 切换
git checkout main
git switch main

# 删除已合并的本地分支
git branch -d feat/xxx

# 强制删除未合并分支
git branch -D feat/xxx

# 删除远程分支
git push origin --delete feat/xxx

# 重命名当前分支
git branch -m new-name

# 查看分支合并情况
git branch --merged
git branch --no-merged
```

## 提交历史整理

### 合并多个小 commit（rebase -i）
```bash
# 整理最近3个commit
git rebase -i HEAD~3
```
在编辑器中：
- `pick`：保留
- `squash`：合并到前一个commit，保留message
- `fixup`：合并到前一个commit，丢弃message
- `reword`：保留但修改message

**注意**：只整理未 push 的 commit；已 push 的不要 rebase。

###  cherry-pick（拣选某个commit到当前分支）
```bash
git cherry-pick <sha>
```

## 日志查看

```bash
# 简洁图形化
git log --oneline --graph --all -20

# 查看某个文件的修改历史
git log --oneline -- <file>

# 查看某行是谁改的
git blame <file>

# 查看某次提交的改动
git show <sha>
```

## Git LFS（大文件）

当项目需要跟踪 >50MB 的二进制文件时启用：

```bash
# 安装 LFS（一次性）
git lfs install

# 跟踪大文件类型
git lfs track "*.psd"
git lfs track "*.zip"

# .gitattributes 会被自动创建/更新，需要提交
git add .gitattributes
```

注意：
- GitHub 免费账户 LFS 有流量和存储配额（1GB 存储，1GB/月流量）
- 已提交到普通 git 历史中的大文件不会自动迁移，需用 `git lfs migrate`
- 如果不确定是否需要 LFS，优先考虑把大文件放到外部存储（如 Release 附件）
