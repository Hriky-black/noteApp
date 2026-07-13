# Git 常用指令手册

> 一份适合日常开发、复习与查阅的 Git 速查笔记。命令默认在**项目根目录**执行。

---

## 快速导航

- [三个区域](#0-先理解三个区域)
- [日常保存与上传](#1-每天最常用保存并上传代码)
- [查看历史与理解 `HEAD`](#2-查看历史与改动)
- [撤销与回溯](#3-撤销与回溯按安全程度使用)
- [分支开发](#4-分支独立开发功能)
- [GitHub 远程仓库](#5-github-远程仓库)
- [常见问题](#6-常见问题速查)

---

## 0. 先理解三个区域

```text
工作区（正在编辑的文件）
        │ git add
        ▼
暂存区（准备提交的改动）
        │ git commit
        ▼
本地仓库（本机的版本历史）
        │ git push
        ▼
远程仓库（GitHub / GitLab）
```

| 概念 | 含义 |
| --- | --- |
| 工作区 | 电脑中正在修改的项目文件。 |
| 暂存区 | 下一次提交准备包含哪些改动的清单。 |
| 提交（commit） | 一个可回溯的版本快照。 |
| 分支（branch） | 独立开发线，例如 `main`、`feature/login`。 |
| 远程仓库（remote） | 托管在 GitHub 等平台上的仓库副本。 |

---

## 1. 每天最常用：保存并上传代码

### 查看当前状态

```powershell
git status
```

**作用：** 查看改了哪些文件、哪些已暂存、当前所在分支。

### 暂存所有改动

```powershell
git add .
```

**作用：** 将当前目录下的新增、修改、删除文件放入暂存区。

> 想只提交某个文件：`git add index.html`

### 创建一个版本

```powershell
git commit -m "完成笔记搜索功能"
```

**作用：** 将暂存区内容保存为本地版本。

**提交信息建议：** 用动词开头，描述“做了什么”，例如：

```text
新增：添加笔记标签功能
修复：解决移动端菜单遮挡问题
优化：调整编辑器排版与间距
文档：补充 Git 使用说明
```

### 上传到 GitHub

```powershell
git push
```

**作用：** 将本地提交上传到已关联的远程分支。

### 一次完成日常流程

```powershell
git add .
git commit -m "描述本次修改"
git push
```

---

## 2. 查看历史与改动

### 精简查看版本历史

```powershell
git log --oneline --decorate
```

**作用：** 每行显示一个提交；`HEAD` 表示当前版本，分支名会一并显示。

示例：

```text
6758800 (HEAD -> main, origin/main) 第一个可用版本
```

### 读懂 `HEAD -> main, origin/main`

这段文字看起来复杂，其实是在说明“你在哪里”以及“GitHub 记录在哪里”。

```text
你的电脑（本地仓库）                         GitHub（远程仓库）

HEAD ──► main ──► 6758800  ◄────────────  origin/main
        当前本地分支                            GitHub 上 main 的本地记录
```

| 标记 | 含义 | 可以这样记 |
| --- | --- | --- |
| `HEAD` | 当前正在查看、编辑的版本位置。通常它指向当前分支。 | “我现在站在这里”。 |
| `HEAD -> main` | 当前检出的是本地 `main` 分支；新提交会添加在 `main` 后面。 | “我正在 main 上工作”。 |
| `main` | 本机上的主分支。可以编辑、提交、切换。 | “我电脑里的主线”。 |
| `origin` | 远程仓库的默认别名；本项目中它指向 GitHub。 | “GitHub 仓库的简称”。 |
| `origin/main` | Git 记录的 GitHub `main` 分支最新位置。它会在 `git fetch`、`git pull`、`git push` 后更新。 | “GitHub 主线在我本机的路标”。 |

#### 为什么 `main` 与 `origin/main` 有时不在同一个提交？

这是正常现象，说明本地与 GitHub 尚未同步：

| 看到的情况 | 含义 | 下一步 |
| --- | --- | --- |
| `main` 在前，`origin/main` 在后 | 本地有新提交，但还没有上传。 | 执行 `git push`。 |
| `origin/main` 在前，`main` 在后 | GitHub 有新提交，但本地还没有。 | 执行 `git pull`。 |
| 两者都指向同一个提交 | 本地与 GitHub 已同步。 | 可以继续开发。 |

可用下面的命令快速确认：

```powershell
git status --branch
```

若输出出现 `ahead 1`，表示本地领先 1 个提交，执行 `git push`；若出现 `behind 1`，表示本地落后 1 个提交，执行 `git pull`。

> `origin/main` 不是一个可直接编辑的普通分支。要修改内容，请切换并提交到 `main`，再用 `git push` 上传。

### 查看某次提交的详细内容

```powershell
git show 6758800
```

**作用：** 显示该提交修改了哪些文件，以及具体代码差异。

> `6758800` 是提交 ID，可用 `git log --oneline` 查询。

### 查看尚未暂存的改动

```powershell
git diff
```

**作用：** 对比工作区与暂存区，查看还没有 `git add` 的修改。

### 查看已经暂存的改动

```powershell
git diff --staged
```

**作用：** 检查下一次 `git commit` 会提交什么内容。

### 查看单个文件的修改记录

```powershell
git log --oneline -- index.html
```

**作用：** 只显示指定文件的提交历史。

---

## 3. 撤销与回溯（按安全程度使用）

### 撤销工作区的单个文件修改

```powershell
git restore index.html
```

**作用：** 丢弃 `index.html` 尚未暂存的修改，恢复到最近一次提交状态。

> ⚠️ 此操作会丢失该文件未提交的内容；请先用 `git diff` 确认。

### 取消暂存，但保留文件修改

```powershell
git restore --staged index.html
```

**作用：** 把文件从暂存区移回工作区，文件编辑内容不会丢失。

### 恢复某文件到指定旧版本

```powershell
git restore --source=6758800 -- index.html
```

**作用：** 用指定提交中的 `index.html` 覆盖当前文件。

恢复后需要重新提交：

```powershell
git add index.html
git commit -m "恢复 index.html 到第一个可用版本"
git push
```

### 临时查看旧版本

```powershell
git switch --detach 6758800
```

**作用：** 将项目切换到旧提交，仅用于查看、运行或对比。

返回当前分支：

```powershell
git switch main
```

> 此方式不会改写历史，是查看旧版本的首选。

### 用一个“反向提交”撤销已推送内容

```powershell
git revert 6758800
git push
```

**作用：** 新建一次提交来抵消目标提交的改动。

> 推荐用于已经推送到 GitHub、且可能被他人使用过的提交；它不会重写历史。

### 重置到旧版本（谨慎）

```powershell
git reset --hard 6758800
```

**作用：** 本地分支、暂存区和工作区都直接回到指定提交。

> ⚠️ 高风险：未保存的改动会丢失，已推送版本通常还需强制推送。多人协作时请优先使用 `git revert`。

---

## 4. 分支：独立开发功能

### 查看本地和远程分支

```powershell
git branch -avv
```

**作用：** 显示本地分支、远程分支，以及各分支最后一次提交。

### 新建并切换到功能分支

```powershell
git switch -c feature/tags
```

**作用：** 创建 `feature/tags` 分支，并立即切换过去开发。

### 切换分支

```powershell
git switch main
```

**作用：** 返回主分支。

### 首次上传新分支

```powershell
git push -u origin feature/tags
```

**作用：** 将新分支推送到 GitHub，并建立后续 `git push` 的默认关联。

### 合并功能分支到主分支

```powershell
git switch main
git merge feature/tags
git push
```

**作用：** 将功能分支中的已提交修改合并到 `main`。

### 删除已合并的本地分支

```powershell
git branch -d feature/tags
```

**作用：** 删除本地功能分支（仅在 Git 确认它已合并后执行）。

---

## 5. GitHub 远程仓库

### 本地与 GitHub 的同步关系

```text
git commit                    git push
工作区 ── git add ──► 暂存区 ─────────► main ─────────► origin/main（GitHub）
                                                     │
                                                     └── git pull：从 GitHub 获取并合并更新
```

**核心原则：** `commit` 只保存到电脑；`push` 才会上传到 GitHub；`pull` 用来获取 GitHub 上的新版本。

### 查看远程仓库地址

```powershell
git remote -v
```

**作用：** 显示 `origin` 的拉取（fetch）与推送（push）地址。

本项目目前的远程仓库：

```text
https://github.com/Hriky-black/noteApp.git
```

### 关联新的 GitHub 仓库

```powershell
git remote add origin https://github.com/用户名/仓库名.git
```

**作用：** 为本地仓库添加名为 `origin` 的远程地址。

> 如果已有关联地址，使用 `git remote set-url origin 新地址` 更新，不要重复 `add`。

### 拉取远程最新提交并合并

```powershell
git pull
```

**作用：** 下载远程分支的新提交，并合并到当前分支。

### 只下载、不合并

```powershell
git fetch
```

**作用：** 更新远程信息，但不改动当前文件；适合先检查再决定如何处理。

### 首次上传主分支

```powershell
git push -u origin main
```

**作用：** 将本地 `main` 上传并建立追踪关系；之后通常只需 `git push`。

---

## 6. 常见问题速查

| 情况 | 建议命令 | 说明 |
| --- | --- | --- |
| 忘记刚才改了什么 | `git status`、`git diff` | 先看状态和差异。 |
| 提交漏了文件 | `git add 文件名` → `git commit --amend --no-edit` | 将文件补进**最后一次、尚未推送**的提交。 |
| 提交信息写错 | `git commit --amend -m "新说明"` | 修改最后一次、尚未推送的提交说明。 |
| `push` 被拒绝 | `git pull` 后解决冲突，再 `git push` | 通常是远程有本地没有的提交。 |
| 文件不应提交 | 写入 `.gitignore`，再执行 `git rm --cached 文件名` | `.gitignore` 只影响未被跟踪的文件。 |
| 想找回误删内容 | `git log --oneline` → `git show 提交ID` | Git 已提交的内容通常可找回。 |

---

## 7. 推荐工作习惯

1. 开始工作前：`git pull`
2. 完成功能后：`git status` 与 `git diff` 检查内容
3. 提交前：`git add .` 后用 `git diff --staged` 再确认一次
4. 提交并上传：`git commit -m "清晰的说明"` → `git push`
5. 每完成一个可运行的小功能就提交一次，提交应当小而清晰

---

## 8. 最小日常命令卡片

```powershell
# 1. 查看状态
git status

# 2. 暂存全部改动
git add .

# 3. 创建版本
git commit -m "描述本次修改"

# 4. 上传 GitHub
git push
```

**记忆口诀：** `status` 看状态 → `add` 放入暂存区 → `commit` 存版本 → `push` 上传。
