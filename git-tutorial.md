# Git 入门到日常使用教程

> **适用读者**：零基础、希望学习 Git 并能独立完成日常开发协作的开发者。
> **环境说明**：本教程命令适用于 Windows / macOS / Linux 终端，建议使用 Git 2.30 及以上版本（`git switch`、`git restore` 等命令需要 Git 2.23+）。
> **阅读建议**：第 1~4 章建立概念，第 5 章作为命令手册随用随查，第 10~12 章用于排错、规范与速查。

## 目录

- [1. Git 是什么](#1-git-是什么)
- [2. Git 与 GitHub 的区别](#2-git-与-github-的区别)
- [3. 安装与初始配置](#3-安装与初始配置)
- [4. Git 核心工作流](#4-git-核心工作流)
- [5. 常用命令解析](#5-常用命令解析)
- [6. 分支与合并](#6-分支与合并)
- [7. 远程仓库协作](#7-远程仓库协作)
- [8. 撤销与回退](#8-撤销与回退)
- [9. 忽略文件](#9-忽略文件)
- [10. 常见问题](#10-常见问题)
- [11. 最佳实践](#11-最佳实践)
- [12. 命令速查表](#12-命令速查表)

---

## 1. Git 是什么

Git 是一个**分布式版本控制系统**（Distributed Version Control System），由 Linus Torvalds 在 2005 年创建。它用来记录文件的每一次修改历史，让你可以：

- 随时回退到任意历史版本；
- 安全地并行开发多个功能（分支）；
- 多人协作时合并彼此的代码；
- 追溯"这行代码是谁、在什么时候、为什么改的"。

### 1.1 四个核心区域

理解 Git 的关键，是理解代码在四个区域之间的流动：

```
工作区            暂存区              本地仓库            远程仓库
(Working        (Staging Area /     (Local Repository,  (Remote Repository,
 Directory)       Index)              .git 目录)           如 GitHub)
   │  git add       │  git commit       │  git push          │
   └───────────────>└──────────────────>└───────────────────>│
   ^                                                        │
   └──────────── git clone / git pull (fetch + merge) ──────┘
```

| 区域 | 是什么 | 位置 |
|------|--------|------|
| 工作区 | 你正在编辑的文件目录，能直接看到和修改 | 项目文件夹 |
| 暂存区 | 下一次提交的"购物车"，`git add` 把改动放进来 | `.git` 目录内的 index 文件 |
| 本地仓库 | 已经提交（commit）的完整历史 | 项目下的 `.git` 目录 |
| 远程仓库 | 团队共享的仓库，用于同步代码 | GitHub / GitLab / 公司 Git 服务器等 |

**一句话总结**：改动先 `add` 进暂存区，再 `commit` 进本地仓库，最后 `push` 到远程仓库；别人推上去的代码用 `pull` 拉下来。即使没有网络，前三步也完全可以离线完成——这就是"分布式"的含义。

---

## 2. Git 与 GitHub 的区别

初学者最容易混淆的两个概念。**Git 是工具，GitHub 是托管 Git 仓库的网站。**

| 对比项 | Git | GitHub |
|--------|-----|--------|
| 本质 | 分布式版本控制**工具（软件）** | 基于 Git 的**代码托管与协作平台（网站）** |
| 运行位置 | 你的本地电脑 | 云端服务器 |
| 核心功能 | 记录版本、分支、合并、历史管理 | 托管仓库、Pull Request、Issue、Code Review、CI/CD |
| 是否联网 | 不需要，完全离线可用 | 需要 |
| 关系 | Git 是基础，没有 Git 就没有仓库 | GitHub 是使用 Git 的平台之一 |

类似的平台还有 **GitLab**、**Bitbucket**、Gitee（码云）等，它们与 GitHub 定位相似。你也可以把公司内网的 GitLab 作为远程仓库——用法完全一致，只是远程地址不同。

---

## 3. 安装与初始配置

### 3.1 安装 Git

- **Windows**：从官网 <https://git-scm.com/download/win> 下载 `Git for Windows` 安装包，一路下一步即可；也可使用包管理器 `winget install Git.Git`。
- **macOS**：`brew install git`（需已安装 Homebrew），或安装 Xcode Command Line Tools。
- **Linux（Ubuntu/Debian）**：`sudo apt install git`。

安装完成后，打开终端（Windows 上是 Git Bash），验证：

```bash
git --version
```

输出示例（版本号可能不同）：

```bash
git version 2.43.0
```

### 3.2 初始配置（必做）

Git 每次提交都会记录作者身份，所以装好后第一件事是配置用户名和邮箱（**邮箱建议与你注册 GitHub 用的邮箱一致**，这样提交才能正确关联到你的账号）：

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

> `--global` 表示对当前用户的所有仓库生效，只需配置一次。如果某个项目需要用另一身份提交，进入该项目目录后去掉 `--global` 再执行一次即可（仅对该仓库生效）。

其他推荐的初始配置：

```bash
# 新仓库的默认分支名设为 main（Git 2.28+，与 GitHub 默认一致）
git config --global init.defaultBranch main

# Windows 下让 Git 正确显示中文文件名（可选）
git config --global core.quotepath false
```

### 3.3 查看配置

```bash
# 查看所有配置（三个级别合并后的结果）
git config --list

# 只看全局配置
git config --global --list

# 查看某一项配置的值
git config user.name
```

输出示例：

```bash
user.name=Your Name
user.email=you@example.com
init.defaultbranch=main
```

---

## 4. Git 核心工作流

日常开发 90% 的场景，就是下面这个循环：

```
修改文件 → git add（暂存）→ git commit（提交到本地仓库）→ git push（推送到远程）
    ^                                                          |
    └──────────── 开始新一天工作前先 git pull ─────────────────┘
```

对应的完整过程：

```bash
# 1. 修改了文件后，先看看改了什么
git status
git diff

# 2. 把改动放入暂存区
git add index.html

# 3. 把暂存区的内容提交为一个版本（快照）
git commit -m "feat: add login page"

# 4. 把本地提交推送到远程仓库
git push
```

记住三个核心动作与三个区域的对应关系：

| 命令 | 改动去向 |
|------|----------|
| `git add` | 工作区 → 暂存区 |
| `git commit` | 暂存区 → 本地仓库 |
| `git push` | 本地仓库 → 远程仓库 |

---

## 5. 常用命令解析

> 每个命令按"作用 → 基本语法 → 常用参数 → 示例 → 执行结果说明 → 注意事项"的模板讲解。本章可以当作手册，用到哪条查哪条。

### 5.1 `git init` — 初始化一个新仓库

**作用**：把当前目录（或指定目录）变成一个 Git 仓库，即创建 `.git` 目录，用于保存所有版本历史。

**基本语法**：

```bash
git init [<目录名>] [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `<目录名>` | 在当前路径下新建该目录并初始化，省略则在当前目录初始化 |
| `-b <分支名>` | 指定初始分支名（Git 2.28+），如 `-b main` |
| `--bare` | 创建"裸仓库"（无工作区），仅用于搭建中央服务器，日常开发用不到 |

**示例**：

```bash
# 在当前目录初始化仓库
git init

# 新建 my-project 目录并初始化，初始分支为 main
git init my-project -b main
```

**执行结果 / 输出说明**：

```bash
Initialized empty Git repository in C:/Users/you/my-project/.git/
```

表示已生成空的 `.git` 目录。此时仓库里没有任何提交，处于"空仓库"状态。

**注意事项 / 常见误区**：

- `git init` 是**安全操作**，不会移动、修改或删除你已有的任何文件，只是在目录下多了个 `.git`。
- `.git` 目录存放全部历史，**不要手动修改或删除它**；删掉它 = 丢掉所有版本历史，只剩下当前文件。
- 在一个已经包含文件的目录里执行 `git init` 完全没问题，之后正常 `add` + `commit` 即可。
- 只应在**项目根目录**初始化，不要在子目录里嵌套初始化。

### 5.2 `git clone` — 克隆远程仓库

**作用**：把一个远程仓库完整下载到本地，包括全部提交历史，并自动建立与远程的关联（默认别名为 `origin`）。

**基本语法**：

```bash
git clone <仓库地址> [<本地目录名>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `<仓库地址>` | HTTPS 或 SSH 地址，如 `https://github.com/user/repo.git` |
| `<本地目录名>` | 自定义本地文件夹名，省略则用仓库名 |
| `-b <分支名>` | 克隆后直接检出指定分支 |
| `--depth 1` | 浅克隆，只拉最近一次提交，历史巨大的仓库可大幅提速 |
| `--recursive` | 同时克隆子模块（submodule） |

**示例**：

```bash
# 基本用法（会在当前目录创建 repo 文件夹）
git clone https://github.com/octocat/Hello-World.git

# 指定本地目录名
git clone https://github.com/user/repo.git my-repo

# 只克隆 dev 分支的最新一次提交
git clone -b dev --depth 1 https://github.com/user/repo.git
```

**执行结果 / 输出说明**：

```bash
Cloning into 'repo'...
remote: Enumerating objects: 1234, done.
remote: Counting objects: 100% (1234/1234), done.
remote: Compressing objects: 100% (567/567), done.
remote: Total 1234 (delta 100), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (1234/1234), 2.31 MiB | 1.20 MiB/s, done.
Resolving deltas: 100% (100/100), done.
```

克隆完成后自动进入默认分支（通常是 `main`），且 `origin` 已配置好，可直接 `git pull` / `git push`。

**注意事项 / 常见误区**：

- 克隆**不需要**再执行 `git init` 或 `git remote add`，这些 `clone` 都帮你做了。
- HTTPS 方式每次推送可能要求输入账号密码（可配置凭证管理器）；SSH 方式配好密钥后免密，适合长期使用。
- 如果克隆失败，先检查网络、仓库地址是否正确、以及该仓库是否需要权限（私有仓库）。

### 5.3 `git status` — 查看仓库状态

**作用**：查看当前分支、哪些文件被修改、哪些已暂存、哪些还未跟踪。它是你使用频率最高、最应该随时执行的命令。

**基本语法**：

```bash
git status [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-s` / `--short` | 精简格式输出，一行一个文件 |
| `-b` | 与 `-s` 连用，同时显示分支信息（`-sb`） |

**示例**：

```bash
git status
git status -s
git status -sb
```

**执行结果 / 输出说明**：

完整格式：

```bash
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   index.html

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   style.css

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        login.js
```

精简格式（`git status -s`）：

```bash
M  index.html     # 已暂存（M 在第一列）
 M style.css      # 已修改未暂存（M 在第二列）
?? login.js       # 未跟踪的新文件
```

Git 的提示信息非常友好，会直接告诉你下一步该用什么命令，务必阅读括号里的 `(use ...)` 提示。

**注意事项 / 常见误区**：

- 养成习惯：`commit` 前、`pull` 前、下班前都看一眼 `git status`，避免漏提交或带着未提交改动切分支。
- "Untracked files" 是 Git 完全不认识的新文件，必须先 `git add` 才会被纳入版本管理。

### 5.4 `git add` — 把改动放入暂存区

**作用**：把工作区的改动（新增、修改、删除）放入暂存区，为下一次提交做准备。

**基本语法**：

```bash
git add <路径> [<路径>...]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `<文件>` | 暂存指定文件，如 `git add index.html` |
| `<目录>/` | 暂存整个目录下的改动 |
| `.` | 暂存**当前目录及子目录**下所有改动（Git 2.x 含新增、修改、删除） |
| `-A` / `--all` | 暂存**整个仓库**所有改动（不受当前所在目录影响） |
| `-u` / `--update` | 只暂存**已跟踪文件**的修改和删除，不含新文件 |
| `-p` / `--patch` | 交互式逐块（hunk）选择要暂存的内容，可把一个文件的部分改动暂存 |

**示例**：

```bash
# 暂存单个文件
git add index.html

# 暂存 src 目录下所有改动
git add src/

# 暂存当前目录下全部改动（最常用）
git add .

# 一个文件里只暂存一部分改动
git add -p index.html
```

**执行结果 / 输出说明**：

`git add` 本身没有输出（Linux 惯例：没有消息就是好消息）。验证方式：

```bash
git status
# index.html 从 "Changes not staged" 移动到 "Changes to be committed"
```

**注意事项 / 常见误区**：

- **常见误区**：以为 `add` 过一次的文件以后改动会自动进暂存区。错——每次修改后都要重新 `git add`，否则 `commit` 提交的是上次 `add` 时的内容。
- `git add .` 和 `git add -A` 在 Git 2.x 效果接近，区别是：`.` 只作用于当前目录及子目录，`-A` 作用于整个仓库。在项目根目录下执行时两者等价。
- `add` 只是"放入购物车"，**没有**写入版本历史，随时可以用 `git restore --staged <file>` 取消暂存。

### 5.5 `git commit` — 提交到本地仓库

**作用**：把暂存区的内容作为一个永久快照（提交）写入本地仓库，每次提交附带唯一的 SHA-1 哈希值（如 `a1b2c3d`）、作者、时间、说明信息。

**基本语法**：

```bash
git commit [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `-m <信息>` | 直接在命令行写提交信息（message），不用打开编辑器 |
| `-a` | 自动暂存**所有已跟踪文件**的修改并提交（不含新文件），相当于 `git add -u` + `git commit` |
| `--amend` | 修补上一次提交：可修改提交信息或补入漏掉的文件（**会改写该提交**） |
| `--allow-empty` | 允许创建空提交（极少用） |

**示例**：

```bash
# 标准提交
git commit -m "feat: add login page"

# 修改过的老文件一键暂存并提交（新文件仍需先 git add）
git commit -am "fix: correct login redirect"

# 上次提交信息写错了，还没推送，改一下
git commit --amend -m "feat: add login page with validation"

# 发现漏了一个文件，补进上一次提交（不弹编辑器）
git add forgotten.js
git commit --amend --no-edit
```

**执行结果 / 输出说明**：

```bash
[main a1b2c3d] feat: add login page
 2 files changed, 25 insertions(+)
 create mode 100644 login.js
```

`[分支名 提交哈希前7位]` + 提交信息 + 变更统计。

**注意事项 / 常见误区**：

- 提交信息要能说明"这次改动做了什么"，避免 `update`、`fix bug`、`111` 这类无意义信息（规范见第 11 章）。
- `git commit -am` 只对**已跟踪**文件生效，新建的文件不会被包含，这是新手最常踩的坑。
- **`--amend` 会生成新的提交哈希替换旧提交，属于改写历史**。如果该提交已经推送到远程，不要再 amend（否则推送会被拒绝，强行推送会给协作者造成麻烦）。

### 5.6 `git log` — 查看提交历史

**作用**：查看提交历史，包含哈希、作者、日期、提交信息，是回溯和定位问题的基础命令。

**基本语法**：

```bash
git log [<选项>] [<分支名>] [-- <路径>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `--oneline` | 每个提交压缩成一行（只显示哈希前 7 位 + 信息） |
| `-<n>` | 只显示最近 n 条，如 `-5` |
| `--graph` | 用 ASCII 图形展示分支与合并结构 |
| `--all` | 显示所有分支的提交（默认只显示当前分支可达的提交） |
| `-p` | 显示每条提交的完整差异（patch） |
| `--stat` | 显示每条提交的文件变更统计 |
| `--author=<名字>` | 按作者过滤 |
| `--grep=<关键词>` | 按提交信息关键词过滤 |
| `-- <路径>` | 只看某个文件/目录的历史 |

**示例**：

```bash
# 完整历史
git log

# 最常用：一行一条，带分支拓扑图
git log --oneline --graph --all

# 只看最近 5 条
git log -5 --oneline

# 看 index.html 的修改历史
git log --oneline -- index.html

# 找出包含 "login" 的提交
git log --oneline --grep="login"
```

**执行结果 / 输出说明**：

```bash
e4f5a6b (HEAD -> main, origin/main) docs: update README
a1b2c3d feat: add login page
1234567 (tag: v0.1.0) chore: project init
```

括号里的 `HEAD -> main` 表示当前所在位置；`(tag: v0.1.0)` 表示该提交打了标签。

**注意事项 / 常见误区**：

- 输出默认使用分页器（less），按 **`q` 退出**，空格翻页，`/关键词` 可搜索。
- 看某个历史提交的完整内容：`git show <哈希>`；只想看哈希：`git log --oneline`。

### 5.7 `git diff` — 查看差异

**作用**：对比两个区域或两个版本之间的具体差异，逐行显示增删内容。

**基本语法**：

```bash
git diff [<选项>] [<比较对象>]
```

**常用参数 / 用法**：

| 用法 | 比较的对象 | 含义 |
|------|-----------|------|
| `git diff` | 工作区 vs 暂存区 | 看尚未暂存的改动 |
| `git diff --staged`（或 `--cached`） | 暂存区 vs 最近一次提交 | 看**已暂存、即将提交**的改动 |
| `git diff HEAD` | 工作区 vs 最近一次提交 | 看所有未提交的改动（暂存 + 未暂存） |
| `git diff <分支1> <分支2>` | 两个分支最新提交 | 看分支之间差异 |
| `git diff HEAD~1 HEAD` | 上次提交 vs 本次提交 | 看最近一次提交改了什么 |
| `git diff <文件>` | 限定某个文件 | 只看该文件差异 |

**示例**：

```bash
# 我刚才改了什么（还没 add）
git diff

# 我 add 进去的都是些什么（commit 前最后检查）
git diff --staged

# main 和功能分支差了什么
git diff main feature-login

# 只看 index.html 的未暂存改动
git diff index.html
```

**执行结果 / 输出说明**：

```bash
diff --git a/index.html b/index.html
index 3a1c2b4..9d8e7f6 100644
--- a/index.html
+++ b/index.html
@@ -1,5 +1,6 @@
 <html>
 <body>
-<h1>Hello</h1>
+<h1>Hello Git</h1>
+<p>new paragraph</p>
 </body>
 </html>
```

`-` 开头（红色）表示删除的行，`+` 开头（绿色）表示新增的行，`@@` 区域显示改动所在的行号范围。

**注意事项 / 常见误区**：

- **最常见困惑**："我明明改了文件，为什么 `git diff` 没输出？"——因为改动已经 `add` 进暂存区了，此时应使用 `git diff --staged`。
- 记住口诀：**不加参数看"没 add 的"，`--staged` 看"add 了还没 commit 的"**。

### 5.8 `git branch` — 分支管理

**作用**：列出、创建、删除、重命名分支（只管理分支本身，不切换；切换用 `git switch` / `git checkout`）。

**基本语法**：

```bash
git branch [<分支名>] [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| （无参数） | 列出本地分支，当前分支前有 `*` |
| `<分支名>` | 基于当前提交创建新分支（不切换过去） |
| `-v` | 列出分支并显示每个分支最新提交 |
| `-a` | 列出本地 + 远程跟踪分支 |
| `-d <分支名>` | 删除分支（安全模式：未合并的分支会拒绝删除） |
| `-D <分支名>` | 强制删除分支（**丢弃未合并的提交**，谨慎） |
| `-m <新名>` | 重命名当前分支；`-m <旧名> <新名>` 重命名指定分支 |

**示例**：

```bash
# 查看所有本地分支
git branch

# 本地 + 远程分支
git branch -a

# 创建功能分支（创建后仍停留在当前分支）
git branch feature-login

# 把当前分支重命名为 main
git branch -m main

# 删除已合并的分支
git branch -d feature-login
```

**执行结果 / 输出说明**：

```bash
  feature-login
* main
  remotes/origin/main
  remotes/origin/feature-login
```

**注意事项 / 常见误区**：

- 不能删除当前所在的分支，先切走再删。
- `-d` 删除被拒绝（`error: the branch 'xxx' is not fully merged`）是**保护机制**，说明该分支还有没合并的提交；确认真的不要了再用 `-D`。
- 远程分支删除用 `git push origin --delete <分支名>`（见 5.13），本地 `git branch -d` 删不掉远程。

### 5.9 `git checkout` — 切换分支 / 检出（传统命令）

**作用**：老牌"万能"命令，身兼两职：**切换分支/检出提交**，以及**丢弃工作区改动**。因为职责太多容易误用，Git 2.23 起拆分出更安全的 `git switch`（只管切换）和 `git restore`（只管恢复文件），但 `checkout` 依然广泛使用，必须认识。

**基本语法**：

```bash
git checkout <分支名 | 提交哈希> [-- <文件>]
```

**常用用法**：

| 用法 | 说明 |
|------|------|
| `git checkout <分支名>` | 切换到该分支 |
| `git checkout -b <新分支名>` | 创建新分支并立即切换（最常用） |
| `git checkout -` | 切回上一个分支 |
| `git checkout -- <文件>` | **丢弃该文件未暂存的修改**（旧写法，等价于 `git restore <文件>`） |
| `git checkout <提交哈希>` | 进入"分离头指针"（detached HEAD）状态查看历史版本 |

**示例**：

```bash
# 创建并切换到功能分支
git checkout -b feature-login

# 切回主分支
git checkout main

# 丢弃 index.html 未暂存的修改（不可恢复！）
git checkout -- index.html
```

**执行结果 / 输出说明**：

```bash
Switched to a new branch 'feature-login'
```

**注意事项 / 常见误区**：

- **危险点**：`git checkout -- <文件>` 会把未暂存的修改**直接丢弃、无法找回**，敲命令前确认清楚。
- `git checkout <哈希>` 会提示 `You are in 'detached HEAD' state`（分离头指针），此时只是"时光旅行"去看旧版本，任何新提交不属于任何分支、切走后容易被丢弃；若想基于旧版本开发，应创建分支：`git checkout -b fix-branch <哈希>`。
- 新项目建议优先使用 `git switch` / `git restore`，语义更清晰、更不容易误伤。

### 5.10 `git switch` — 切换分支（现代命令，Git 2.23+）

**作用**：专管分支切换，是 `git checkout` 切换功能的替代品，职责单一更安全。

**基本语法**：

```bash
git switch <分支名> [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `<分支名>` | 切换到已有分支 |
| `-c <新分支名>` | 创建并切换（create） |
| `-` | 切回上一个分支 |

**示例**：

```bash
# 等价于 git checkout -b feature-login
git switch -c feature-login

# 切换到 main
git switch main

# 在两个分支间来回跳
git switch -
```

**执行结果 / 输出说明**：

```bash
Switched to branch 'feature-login'
```

**注意事项 / 常见误区**：

- `git switch` **不会**像 checkout 那样兼任"恢复文件"，所以不存在误写 `git switch -- file` 丢改动的风险——这正是它存在的意义。
- 如果工作区有未提交改动且与目标分支冲突，switch 会拒绝并提示你先提交或暂存（stash），这是保护而不是故障。

### 5.11 `git merge` — 合并分支

**作用**：把指定分支的提交合并到**当前所在分支**，是把功能分支的成果带回主分支的核心命令。

**基本语法**：

```bash
git merge <分支名> [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `<分支名>` | 把该分支合并进当前分支（方向：**别的 → 当前**） |
| `--no-ff` | 禁止快进合并，强制生成一个合并提交，保留分支存在过的拓扑 |
| `--abort` | 合并出冲突后放弃本次合并，回到合并前状态 |
| `--continue` | 解决冲突并 `add` 后，完成合并提交 |

**示例**：

```bash
# 标准流程：先切到目标分支，再把功能分支合进来
git switch main
git merge feature-login

# 保留分支历史，强制生成合并提交
git merge --no-ff feature-login

# 冲突太乱，不想解决了，回到合并前
git merge --abort
```

**执行结果 / 输出说明**：

快进合并（当前分支没有新提交，直接前移指针）：

```bash
Updating a1b2c3d..e4f5a6b
Fast-forward
 index.html | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```

三方合并（两边的提交汇聚成一个新的合并提交，会弹出编辑器让你确认合并信息）：

```bash
Merge made by the 'ort' strategy.
 index.html | 3 ++-
 1 file changed, 2 insertions(+), 1 deletion(-)
```

产生冲突时：

```bash
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

**注意事项 / 常见误区**：

- **方向最容易搞反**：`git merge feature` 是把 `feature` 合入"我当前所在的分支"。所以合并前先 `git status` 确认自己在哪个分支。
- 冲突不是错误，是 Git 无法替你决定保留哪边代码时的正常询问，解决步骤见 6.4 节。
- 养成"合并前先把工作区收拾干净（提交或 stash）"的习惯。

### 5.12 `git remote` — 管理远程仓库

**作用**：查看、添加、修改、删除远程仓库的关联配置。远程仓库在本地用一个**别名**表示，惯例叫 `origin`。

**基本语法**：

```bash
git remote [<子命令>] [<参数>]
```

**常用用法**：

| 用法 | 说明 |
|------|------|
| `git remote` | 列出所有远程仓库别名 |
| `git remote -v` | 列出别名 + 完整地址（fetch/push 各一条） |
| `git remote add <别名> <地址>` | 添加远程仓库关联 |
| `git remote set-url <别名> <新地址>` | 修改远程地址（如从 HTTPS 换 SSH） |
| `git remote remove <别名>` | 删除关联（不影响远程仓库本身） |
| `git remote show <别名>` | 查看该远程的详细信息（分支、跟踪关系） |

**示例**：

```bash
# 给本地仓库关联 GitHub 上的远程仓库
git remote add origin https://github.com/user/repo.git

# 查看当前关联的远程地址
git remote -v

# 把远程地址从 HTTPS 换成 SSH
git remote set-url origin git@github.com:user/repo.git
```

**执行结果 / 输出说明**：

```bash
$ git remote -v
origin  https://github.com/user/repo.git (fetch)
origin  https://github.com/user/repo.git (push)
```

**注意事项 / 常见误区**：

- `origin` 只是**默认别名**，不是关键字，你可以叫它任何名字；一个本地仓库可以关联多个远程（例如公司内网 + 开源上游 `upstream`）。
- 最常见报错 `fatal: remote origin already exists.`：说明已经关联过，需要的话用 `set-url` 修改而不是重复 `add`。
- 推送/拉取时写的 `origin` 就是指向这里的别名。

### 5.13 `git push` — 推送到远程仓库

**作用**：把本地仓库的提交上传到远程仓库，与团队共享你的成果。

**基本语法**：

```bash
git push [<远程别名>] [<本地分支>:<远程分支>]
```

**常用用法**：

| 用法 | 说明 |
|------|------|
| `git push` | 推送当前分支（需已设置上游跟踪） |
| `git push origin main` | 把本地 `main` 推到远程 `main` |
| `git push -u origin main` | 推送**并建立跟踪关系**（`-u` = `--set-upstream`），之后可直接 `git push` / `git pull` |
| `git push origin feature-login` | 推送功能分支到远程（首次推送会在远程创建该分支） |
| `git push origin --delete feature-login` | 删除远程分支 |
| `git push --tags` | 推送所有本地标签（标签默认不随 push 推送） |
| `git push --force-with-lease` | 较安全的强制推送（见下方警告） |

**示例**：

```bash
# 第一次推送本地 main 到远程，并记住对应关系
git push -u origin main

# 之后每次同步只需
git push

# 把功能分支推上去发 Pull Request
git push origin feature-login
```

**执行结果 / 输出说明**：

```bash
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Writing objects: 100% (3/3), 312 bytes | 312.00 KiB/s, done.
To https://github.com/user/repo.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

**注意事项 / 常见误区**：

- 被拒绝（`! [rejected] ... (fetch first)`）通常是因为**远程有别人推的新提交**，先 `git pull` 合并再推（详见 10.3）。
- **`git push --force`（强制推送）会覆盖远程历史、可能抹掉别人的提交，属于危险操作**，严禁对公共分支（`main` 等）使用。确有需要（如整理自己未共享的功能分支）时，使用 `git push --force-with-lease`，它会在远程被别人更新过时拒绝执行，相对安全。

### 5.14 `git pull` — 拉取并合并远程更新

**作用**：从远程仓库获取最新提交并**合并到当前分支**。它等于两步的组合：`git fetch` + `git merge`。

**基本语法**：

```bash
git pull [<远程别名>] [<远程分支>] [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| （无参数） | 拉取当前分支的上游（origin/当前分支）并合并 |
| `origin main` | 拉取远程 `main` 并合并到当前分支 |
| `--rebase` | 拉取后用 rebase 方式并入（历史更线性，详见说明） |

**示例**：

```bash
# 每天开工 / 推送前先拉取
git pull

# 显式指定
git pull origin main

# 团队偏好线性历史时
git pull --rebase
```

**执行结果 / 输出说明**：

```bash
remote: Enumerating objects: 4, done.
Unpacking objects: 100% (4/4), done.
From https://github.com/user/repo
   a1b2c3d..e4f5a6b  main       -> origin/main
Updating a1b2c3d..e4f5a6b
Fast-forward
 style.css | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

**注意事项 / 常见误区**：

- `pull` 包含 merge，所以**可能产生冲突**，出现冲突时按 6.4 的步骤处理。
- `git pull` 与 `git fetch` 的区别：`pull = fetch + merge`，`pull` 会直接动你的工作区和当前分支；`fetch` 只更新远程跟踪分支，更安全可控（对比见 7.3）。
- 如果拉取时本地有未提交改动，Git 可能拒绝合并，先提交或 stash。

### 5.15 `git fetch` — 仅获取远程更新（不合并）

**作用**：把远程仓库的最新提交、分支信息下载到本地的**远程跟踪分支**（如 `origin/main`），但**不改动**你的工作区和当前分支。

**基本语法**：

```bash
git fetch [<远程别名>] [<分支名>] [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| （无参数） | 抓取所有远程分支的更新 |
| `origin` | 抓取 `origin` 的全部更新 |
| `origin main` | 只抓取远程 `main` |
| `--prune` | 顺便清理本地缓存中已被远程删除的分支记录 |

**示例**：

```bash
# 先安全地看看远程发生了什么
git fetch origin

# 比较本地与远程的差异（本命令不会改动任何文件）
git log HEAD..origin/main --oneline
git diff HEAD origin/main

# 确认没问题后，再合并
git merge origin/main
```

**执行结果 / 输出说明**：

```bash
remote: Enumerating objects: 4, done.
Unpacking objects: 100% (4/4), done.
From https://github.com/user/repo
   a1b2c3d..e4f5a6b  main       -> origin/main
```

表示"远程 main 有新提交，已下载到 `origin/main`，等你决定要不要合并"。

**注意事项 / 常见误区**：

- `fetch` 是**只读、绝对安全**的操作，任何时候都可以放心执行。
- fetch 之后你的代码没有任何变化，想看"远程比我多了什么"用 `git log HEAD..origin/main`，想看具体差异用 `git diff HEAD origin/main`。
- 不确定该不该直接 `pull`（怕冲突弄乱工作区）时，先 `fetch` 再决定，是更稳妥的工作习惯。

### 5.16 `git stash` — 临时保存工作现场

**作用**：把工作区未提交的改动（含已暂存的）临时存进"栈"里，让工作区瞬间回到干净状态，方便切分支处理紧急任务，回来再恢复。

**基本语法**：

```bash
git stash [push | list | pop | apply | drop] [<选项>]
```

**常用用法**：

| 用法 | 说明 |
|------|------|
| `git stash` | 保存现场（默认只含已跟踪文件的改动） |
| `git stash push -m "说明"` | 保存现场并加说明，推荐 |
| `git stash -u` | 连未跟踪的新文件一起保存 |
| `git stash list` | 查看栈里所有现场 |
| `git stash pop` | 恢复最近一次现场并从栈中删除 |
| `git stash apply stash@{1}` | 恢复指定现场但**保留**在栈中 |
| `git stash drop stash@{0}` | 删除栈中指定现场 |
| `git stash clear` | **清空整个 stash 栈（不可轻易恢复，谨慎）** |

**示例**：

```bash
# 正在开发，突然要去修紧急 bug
git stash push -m "wip: login page half done"

# 工作区干净了，放心切分支
git switch hotfix/crash
# ...修复并提交...

# 回来继续
git switch feature-login
git stash pop
```

**执行结果 / 输出说明**：

保存时：

```bash
Saved working directory and index state WIP on feature-login: a1b2c3d feat: add login page
```

恢复时：

```bash
On branch feature-login
Changes not staged for commit:
        modified:   login.js

Dropped refs/stash@{0} (9d8e7f6a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e)
```

`git stash list`：

```bash
stash@{0}: WIP on feature-login: a1b2c3d feat: add login page
stash@{1}: On feature-x: wip: fix nav
```

`stash@{0}` 是最新保存的现场，编号从 0 开始。

**注意事项 / 常见误区**：

- 默认**不含未跟踪的新文件**，新文件想一起存要加 `-u`。
- `pop` 时如果当前分支的代码与现场冲突，Git 会像合并冲突一样标记出来，解决后现场不会自动出栈，需要手动 `git stash drop`。
- stash 不是长期存储，尽快 `pop` 回来；栈太深容易忘记每个现场是什么，所以保存时务必 `-m` 写说明。

### 5.17 `git reset` — 移动分支指针回退（危险区）

**作用**：把当前分支的指针（HEAD 指向的提交）**移动**到指定位置，按模式决定是否保留改动。用于回退本地尚未共享的提交。

**基本语法**：

```bash
git reset [--soft | --mixed | --hard] [<提交哈希> | HEAD~n]
```

**三种模式（重点）**：

| 模式 | HEAD 指针 | 暂存区 | 工作区（你的文件） | 典型用途 |
|------|-----------|--------|--------|----------|
| `--soft` | 回退 | **保留**（改动仍在暂存区） | 保留 | 撤销提交但想重新组织提交内容 |
| `--mixed`（默认） | 回退 | 重置（改动退回未暂存） | 保留 | 撤销提交，重新挑选要提交的内容 |
| `--hard` | 回退 | 重置 | **全部丢弃** | 彻底回到某个历史状态 |

**常用用法**：

| 用法 | 说明 |
|------|------|
| `git reset --soft HEAD~1` | 撤销最近一次提交，改动保留在暂存区 |
| `git reset HEAD~1` | 撤销最近一次提交，改动退回工作区（未暂存） |
| `git reset --hard HEAD~1` | **彻底删除最近一次提交及其所有改动** |
| `git reset --hard origin/main` | **把本地 main 强制对齐到远程状态（丢弃本地全部未推送提交）** |
| `git reset <文件>` | 旧式写法：把文件移出暂存区（等价 `git restore --staged <文件>`） |

> `HEAD~1` 表示"HEAD 的上一个提交"，`HEAD~2` 表示再往前一个，以此类推；也可以用 `git log` 查到的具体哈希，如 `git reset --mixed a1b2c3d`。

**示例场景：刚提交了一次错误的 commit，还没推送**

```bash
git log --oneline        # 确认要退到哪
# e4f5a6b (HEAD -> main) 这次提交写错了
# a1b2c3d feat: add login page

git reset --soft HEAD~1  # 退回去，改动仍在暂存区，可重新提交
git status               # 看到"Changes to be committed"
```

**执行结果 / 输出说明**：

```bash
Unstaged changes after reset:
M       index.html
```

（`--mixed` 模式会列出被退回未暂存状态的文件；`--soft` 与 `--hard` 通常无输出。）

**注意事项 / 常见误区**：

- **`git reset --hard` 会丢弃工作区所有未提交修改与被回退的提交，且常规手段无法找回，是 Git 中最危险的命令之一。执行前务必：① `git status` 确认没有未提交的重要改动；② 记下当前 HEAD 哈希（万一误操作可用 `git reflog` 抢救）。**
- `reset` 会**改写当前分支的历史**。对**已经推送到远程、他人可能基于其工作**的分支做 reset，等于强迫所有人混乱——公共分支请改用 `git revert`。
- `git reset --hard origin/main` 常用于"本地改乱了，想完全与远程保持一致"，执行前确认本地真的没有需要保留的东西。

### 5.18 `git revert` — 用一个新提交撤销旧提交（安全）

**作用**：生成一个与指定提交**内容相反**的新提交（"反做"），从而抵消该提交的影响。历史只会增加、不会改写，因此适合撤销**已经推送**的提交。

**基本语法**：

```bash
git revert <提交哈希 | HEAD> [<选项>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| `HEAD` | 撤销最近一次提交 |
| `<哈希>` | 撤销指定提交 |
| `--no-edit` | 使用默认的提交信息，不弹编辑器 |
| `-n` / `--no-commit` | 只把反向改动放入暂存区，不自动提交（想和其他撤销合并成一个提交时用） |

**示例**：

```bash
# 撤销最近一次提交（弹编辑器确认信息）
git revert HEAD

# 撤销指定提交
git revert a1b2c3d --no-edit
```

**执行结果 / 输出说明**：

```bash
[main 9988776] Revert "feat: add login page"
 2 files changed, 25 deletions(-)
 delete mode 100644 login.js
```

历史中会**同时保留**原提交和 Revert 提交，一眼可追溯"撤销过什么"。

**注意事项 / 常见误区**：

- `revert` 与 `reset` 的本质区别：**revert 新增提交、不改写历史（安全，可用于公共分支）；reset 移动指针、改写历史（只用于本地/未共享提交）**。
- 如果被撤销的提交之后，同一处代码又被别人改过，revert 可能产生冲突，正常解决即可。
- revert 的是"该提交引入的改动"，不是"把整个项目变回那时的样子"——后者是 reset 或 checkout 历史版本干的事。

### 5.19 `git tag` — 打标签（版本标记）

**作用**：给某个提交打上语义化标记（如 `v1.0.0`），常用于发布版本、永久标识里程碑。

**基本语法**：

```bash
git tag [-a | -d | -l] [<标签名>] [-m <说明>] [<提交哈希>]
```

**常用参数**：

| 参数 | 说明 |
|------|------|
| （无参数） | 列出所有标签 |
| `<标签名>` | 创建轻量标签（lightweight，仅是个指针） |
| `-a <标签名> -m <说明>` | 创建附注标签（annotated，**推荐**：含作者、日期、说明，可签名） |
| `<标签名> <哈希>` | 给历史某次提交补打标签 |
| `-d <标签名>` | 删除本地标签 |
| `-l "v1.*"` | 按通配符筛选标签 |

**示例**：

```bash
# 给当前提交打发布标签（推荐用附注标签）
git tag -a v1.0.0 -m "Release 1.0.0"

# 查看标签内容
git show v1.0.0

# 推送单个标签到远程（push 默认不带标签！）
git push origin v1.0.0

# 推送全部标签
git push --tags

# 删掉本地 / 远程标签
git tag -d v1.0.0
git push origin --delete v1.0.0
```

**执行结果 / 输出说明**：

```bash
$ git tag
v0.1.0
v1.0.0

$ git show v1.0.0
tag v1.0.0
Tagger: Your Name <you@example.com>
Date:   Mon Jan 1 12:00:00 2025 +0800

Release 1.0.0

commit a1b2c3d... (HEAD -> main, tag: v1.0.0)
```

**注意事项 / 常见误区**：

- **标签不会随普通 `git push` 推送**，必须显式 `git push origin <标签>` 或 `git push --tags`，这是新手发布"找不到版本"的头号原因。
- 建议版本号遵循语义化版本（SemVer）：`主版本.次版本.修订号`，如 `v2.1.3`。
- 正式发布用附注标签（`-a`），临时本地记号才用轻量标签。

### 5.20 `git config` — 读写配置

**作用**：查看和修改 Git 的各项配置（身份、编辑器、别名、行为开关等）。

**基本语法**：

```bash
git config [--global | --system | --local] <配置项> [值]
```

**配置的三个级别**（优先级从低到高，高的覆盖低的）：

| 级别 | 作用范围 | 存储位置 |
|------|----------|----------|
| `--system` | 本机所有用户 | Git 安装目录 |
| `--global` | 当前用户的所有仓库 | `~/.gitconfig`（Windows: `C:\Users\你\.gitconfig`） |
| （默认）local | 仅当前仓库 | 项目下 `.git/config` |

**常用用法**：

| 用法 | 说明 |
|------|------|
| `git config --global user.name "名字"` | 设置提交署名 |
| `git config --global user.email "邮箱"` | 设置提交邮箱 |
| `git config --list` | 查看全部生效配置 |
| `git config user.name` | 查看某一项 |
| `git config --global init.defaultBranch main` | 新仓库默认分支为 main |
| `git config --global alias.co checkout` | 设置别名：以后 `git co` = `git checkout` |
| `git config --global --unset user.email` | 删除某项全局配置 |

**示例**：

```bash
# 给常用命令起短别名（进阶提效）
git config --global alias.st "status -s"
git config --global alias.lg "log --oneline --graph --all"

git st     # 等价于 git status -s
git lg     # 等价于 git log --oneline --graph --all
```

**执行结果 / 输出说明**：

```bash
$ git config user.name
Your Name
```

**注意事项 / 常见误区**：

- 身份配置错误会导致 GitHub 上提交不关联你的账号头像/贡献图，改完后**只影响之后的提交**，历史提交需另行处理。
- 想查看某配置最终生效于哪个文件、哪一行：`git config --list --show-origin`。

### 5.21 `.gitignore` — 忽略文件的写法与作用

**作用**：项目根目录下的 `.gitignore` 是一个纯文本文件，列出**不应纳入版本管理**的文件模式（如临时文件、密钥、构建产物、依赖目录），Git 会自动忽略它们，`git status` 不再显示、`git add .` 也不会误提交。

**基本语法规则**：

| 写法 | 含义 |
|------|------|
| `# 注释` | 注释行 |
| `node_modules/` | 忽略名为 `node_modules` 的**目录**（末尾 `/` 限定目录） |
| `*.log` | 忽略所有 `.log` 结尾的文件（`*` 匹配任意字符） |
| `/build` | 只忽略**根目录**下的 `build`（开头 `/` 限定根目录） |
| `doc/**/*.pdf` | 忽略 `doc` 目录下任意层级中的所有 pdf |
| `!important.log` | **例外**：前面的规则忽略了它，但这个文件不忽略（`!` 取反） |
| `temp?.txt` | `?` 匹配单个任意字符 |

**典型示例**（一个通用 `.gitignore`）：

```bash
# 依赖目录
node_modules/

# 构建产物
/dist
/build
*.class

# 日志与临时文件
*.log
*.tmp
.DS_Store

# 环境变量与密钥（绝不入库！）
.env
*.pem

# IDE 配置
.idea/
.vscode/

# 例外：允许提交 VSCode 的项目级编辑器配置
!.vscode/settings.json
```

**执行结果 / 输出说明**：

创建 `.gitignore` 后，匹配的文件将从 `git status` 的 Untracked 列表消失；想验证某个文件为什么被忽略：

```bash
git check-ignore -v .env
# 输出：.gitignore:9:.env      .env   ← 命中 .gitignore 第 9 行的规则
```

**注意事项 / 常见误区**：

- **`.gitignore` 只对"未被跟踪"的文件生效**。如果一个文件已经被提交过，之后再把它加进 `.gitignore` 是**不会**生效的，需要先取消跟踪：

  ```bash
  git rm --cached .env          # 从版本管理中移除，但保留本地文件
  git commit -m "chore: stop tracking .env"
  ```

  `--cached` 的意思就是"只动仓库记录、不删本地文件"；**不带** `--cached` 的 `git rm` 会把本地文件一起删掉，注意区分。
- `.gitignore` 本身应当**提交到仓库**，让整个团队共享忽略规则；个人特有的忽略（如自己的编辑器临时文件）可放全局忽略文件：

  ```bash
  git config --global core.excludesFile ~/.gitignore_global
  ```

### 5.22 综合场景实战

以下六个场景把前面的命令串成完整流程，建议逐行理解。

#### 场景一：新建仓库并完成第一次提交

```bash
mkdir my-project && cd my-project   # 新建并进入项目目录
git init -b main                    # 初始化仓库，默认分支 main

echo "# My Project" > README.md     # 创建第一个文件
git status                          # README.md 显示为 Untracked

git add README.md                   # 放入暂存区
git commit -m "chore: initial commit"   # 完成第一次提交
git log --oneline                   # 查看历史：能看到这条提交
```

#### 场景二：从 GitHub 克隆项目并修改后推送

```bash
git clone https://github.com/user/repo.git   # 克隆（自动关联 origin）
cd repo

# 修改代码后……
git pull                        # 推送前先同步远程，避免被拒绝
git status                      # 确认改动清单
git diff                        # 复查改动内容
git add .                       # 暂存全部改动
git commit -m "fix: correct typo in README"
git push origin main            # 推送到远程 main
```

#### 场景三：创建功能分支、开发、合并回主分支

```bash
git switch -c feature-login     # 从 main 创建并切换到功能分支

# ……开发登录功能，期间多次小步提交……
git add login.js
git commit -m "feat: add login form"
git add auth.js
git commit -m "feat: add auth logic"

# 开发完毕，合并回 main
git switch main                 # 先回到主分支
git pull                        # 确保 main 是最新的
git merge feature-login         # 把功能分支合进来
git push                        # 推送合并结果

git branch -d feature-login     # 删除已合并的本地功能分支
```

#### 场景四：多人修改同一文件产生冲突时如何解决

你和朋友都改了 `index.html` 的同一行，你合并/拉取时会看到：

```bash
$ git pull
Auto-merging index.html
CONFLICT (content): Merge conflict in index.html
Automatic merge failed; fix conflicts and then commit the result.
```

打开 `index.html`，找到冲突标记：

```bash
<<<<<<< HEAD
<h1>欢迎来到我的网站</h1>        ← 你本地的版本
=======
<h1>Welcome to My Site</h1>      <<<<<<< / ======= / >>>>>>> 之间是双方的代码
>>>>>>> origin/main
```

解决步骤：

```bash
# 1. 手动编辑文件：决定保留哪一方（或融合双方），然后删除 <<<<<<< ======= >>>>>>> 三行标记
#    （也可用 git mergetool 或编辑器的冲突解决界面）

# 2. 标记冲突已解决
git add index.html

# 3. 完成合并提交
git commit -m "merge: resolve conflict in index.html"

# 4. 正常推送
git push
```

中途反悔（不想解决了）：`git merge --abort` 回到合并前。

#### 场景五：提交错了如何用 `git revert` 或 `git reset` 处理

```bash
# 情况 A：提交还没推送，直接退回，改动保留
git log --oneline
git reset --soft HEAD~1       # 撤销提交，改动仍在暂存区，可改好再提交

# 情况 B：提交已推送到远程（别人可能看到了），用 revert 安全撤销
git revert HEAD               # 生成一个"反提交"抵消它
git push                      # 把 revert 推上去，历史清晰且不打扰他人
```

选择口诀：**没推送用 reset（退回重做），已推送用 revert（反做抵消）。**

#### 场景六：临时切换任务如何用 `git stash` 保存现场

```bash
# 登录功能写到一半，线上出了紧急 bug
git stash push -m "wip: login page half done"   # 收起手头的改动
git switch main                                 # 工作区已干净，放心切走

# ……切到 hotfix 分支修 bug、提交、推送……
git switch -c hotfix/navbar
git commit -am "fix: navbar overflow on mobile"
git push origin hotfix/navbar

# 修完回到登录功能，恢复现场继续
git switch feature-login
git stash list                # stash@{0}: wip: login page half done
git stash pop                 # 改动原样回来，栈中记录被删除
```

---

## 6. 分支与合并

### 6.1 为什么需要分支

没有分支时，所有人改同一份代码，功能写到一半无法提交（提交了会影响别人），发版时无法区分"完成了的"和"没完成的"。分支让每个人/每个功能拥有一条**独立的开发线**，互不干扰，完成后再汇合。

Git 的分支本质上只是**指向某个提交的可移动指针**，创建分支几乎零成本（这也是它相对老一代版本控制系统最大的优势）。

### 6.2 `main` 与 `master` 的差异

两者**只是默认分支的名字，功能完全相同**，没有任何技术区别：

- Git 早期的默认名字是 `master`；
- 2020 年起，社区（包括 GitHub）逐步改用 `main`，GitHub/GitLab 新仓库默认 `main`；
- 你遇到的仓库可能是其中任何一个（甚至别的名字），**以 `git branch` 看到的为准**，不要想当然。

让本地默认也用 `main`：

```bash
git config --global init.defaultBranch main
```

把本地仓库的 `master` 改名为 `main`：

```bash
git branch -m master main
git push -u origin main
git push origin --delete master   # 确认无误后删除远程旧分支（需仓库设置允许）
```

### 6.3 分支的完整生命周期

```bash
git branch feature-login           # 1. 创建
git switch feature-login           # 2. 切换（或一步到位：git switch -c feature-login）
# ……开发、add、commit……
git switch main                    # 3. 回到主分支准备合并
git merge feature-login            # 4. 合并
git branch -d feature-login        # 5. 删除本地功能分支
git push origin --delete feature-login   # 6. 删除远程功能分支（如已推送过）
```

### 6.4 合并冲突的解决步骤（通用流程）

冲突的本质：两条分支改了**同一处**代码，Git 无法替你决定保留谁。解决流程固定为五步：

1. **触发**：`git merge <分支>` 或 `git pull` 时看到 `CONFLICT (content): Merge conflict in <文件>`；
2. **定位**：`git status` 列出所有 `both modified` 的文件；打开文件找 `<<<<<<<`、`=======`、`>>>>>>>` 标记；
3. **裁决**：手动编辑，保留正确内容（可取一方、可融合双方），**删掉三行标记**；
4. **标记**：`git add <文件>` 告诉 Git 冲突已解决；
5. **收尾**：`git commit` 完成合并（`git pull` 产生的冲突同样如此收尾），然后 `git push`。

> 提示：想放弃这次合并回到合并前，执行 `git merge --abort`。文件较多的复杂冲突可以用 `git mergetool` 调用图形化对比工具。

### 6.5 两种合并形态

| 形态 | 触发条件 | 结果 |
|------|----------|------|
| 快进合并（fast-forward） | 当前分支在分叉后**没有新提交** | 指针直接前移，历史是一条直线 |
| 三方合并（merge commit） | 两条分支**各有新提交** | 生成一个合并提交，历史出现汇合点 |

团队若希望"每个功能在历史上留下一个汇聚点"，使用 `git merge --no-ff <分支>` 强制生成合并提交。

---

## 7. 远程仓库协作

### 7.1 `origin`、`main` 与远程跟踪分支

- **`origin`**：远程仓库的**默认别名**。`clone` 时自动创建，指向你克隆的地址。`git push origin main` 读作"把本地 main 推到别名叫 origin 的那个远程仓库"。
- **`main`**：分支名。远程的 main 在本地缓存里记作 `origin/main`，称为**远程跟踪分支**——它代表"我上次联系远程时，远程 main 长这样"，本身只读，靠 `fetch`/`pull` 更新。
- `git status` 里的 `Your branch is up to date with 'origin/main'` 或 `ahead by 2 commits` 描述的就是本地 main 与 `origin/main` 的领先/落后关系。

### 7.2 关联远程并首次推送

```bash
# 本地已有仓库，想推到 GitHub 上新建的空仓库
git remote add origin git@github.com:user/repo.git
git push -u origin main
```

**`-u`（`--set-upstream`）的作用**：把本地 `main` 与 `origin/main` 建立**跟踪关系**，之后在 main 分支上直接敲 `git push` / `git pull` 即可，不必再写全名。只需在**每个分支首次推送时**设置一次。

### 7.3 `git pull` 与 `git fetch` 的区别

| 对比项 | `git fetch` | `git pull` |
|--------|-------------|------------|
| 下载远程更新 | 是 | 是 |
| 是否改动你的工作区/当前分支 | **否**（只更新 `origin/*`） | **是**（自动合并进当前分支） |
| 可能产生冲突 | 不会 | 可能（合并阶段） |
| 适用场景 | 先看看远程发生了什么再决定；排查问题 | 日常同步（等价 `fetch` + `merge`） |

推荐习惯：**心里没底时先 `git fetch` + `git diff` 看一眼，再合并；日常熟悉后直接 `git pull`。**

### 7.4 多人协作的日常循环

```bash
git switch main              # 1. 每天开工：切到主分支
git pull                     # 2. 同步最新代码
git switch -c feature-x      # 3. 从最新代码切出自己的功能分支
# ……开发、多次 add + commit……
git fetch origin              # 4. 收工：先看远程动态
git push -u origin feature-x  # 5. 推送功能分支，发起 Pull Request / Merge Request
# 6. 代码评审通过后在平台合并，或由维护者 merge 回 main
git branch -d feature-x       # 7. 本地清理已合并分支
git remote prune origin       # （可选）清理远程已删除分支的本地残留记录
```

---

## 8. 撤销与回退

撤销是新手最容易出事的区域。先问自己两个问题：**改动提交了没有？推送了没有？** 然后对号入座。

### 8.1 三员大将的分工

| 命令 | 作用对象 | 典型场景 |
|------|----------|----------|
| `git restore <文件>` | 工作区 | 文件改乱了，想恢复成暂存区/上次提交的样子 |
| `git restore --staged <文件>` | 暂存区 | `add` 错了，想取消暂存（改动保留） |
| `git reset [--soft/--mixed/--hard]` | 本地提交 | 退回本地**未推送**的提交 |
| `git revert <提交>` | 已有提交 | 撤销**已推送**的提交（安全，不改写历史） |

### 8.2 按场景对号入座

**场景 1：文件改乱了，还没 add，想丢弃修改**

```bash
git restore index.html            # 恢复该文件（丢弃未暂存修改，不可恢复！）
# 旧写法等价于：git checkout -- index.html
```

**场景 2：add 了但还没 commit，想取消暂存**

```bash
git restore --staged index.html   # 移出暂存区，改动仍保留在工作区
# 旧写法等价于：git reset HEAD index.html
```

**场景 3：commit 了但没 push，想撤销这次提交**

```bash
git reset --soft HEAD~1     # 撤销提交，改动留在暂存区（想改改再提交）
git reset HEAD~1            # 同上，但改动退回未暂存状态
git reset --hard HEAD~1     # 彻底丢弃这次提交和改动（见下方警告）
```

**场景 4：已经 push 了，想撤销这次提交**

```bash
git revert HEAD             # 用反向提交抵消，然后 git push
```

### 8.3 风险等级速记

- **`git reset --hard`（含 `git reset --hard origin/main`）：会不可逆地丢弃工作区未提交修改和被回退的提交，是最高危操作。执行前确认 `git status` 干净、记下当前哈希；只用于本地未共享的提交。**
- `git restore <文件>` / `git checkout -- <文件>`：丢弃工作区未暂存改动，**同样找不回**，次高危。
- `git reset --soft` / `--mixed`、`git restore --staged`、`git revert`：不丢内容、不改写已共享历史，安全。
- 挽救手段：误操作后先冷静执行 `git reflog` 查看 HEAD 的全部移动记录，找到操作前的哈希，`git reset --hard <哈希>` 或 `git branch recovered <哈希>` 抢救（前提是相关对象未被清理回收）。

### 8.4 `reset` 三模式再对比（结合文件去向）

| 模式 | 提交历史 | 你的改动去哪了 |
|------|----------|----------------|
| `--soft` | 回退 | 原封不动待在**暂存区** |
| `--mixed`（默认） | 回退 | 退回**工作区**（未暂存状态） |
| `--hard` | 回退 | **彻底消失** |

---

## 9. 忽略文件

### 9.1 为什么要忽略

仓库应该保存**源代码与必要资源**，而不是：

- 可重新生成的产物：`node_modules/`、`dist/`、`*.class`、`__pycache__/`；
- 环境相关配置与日志：`.env`、`*.log`、`.DS_Store`；
- **密钥与凭据**：`*.pem`、`id_rsa`、`.env` 中的 API key——一旦推到公共仓库即视为泄露；
- IDE 自动生成的目录：`.idea/`、`.vscode/`。

好处：仓库更小更干净、`git status` 输出清爽、**避免泄露密钥**、避免无意义的合并冲突。

### 9.2 常见模板

```bash
# ---- Node.js ----
node_modules/
npm-debug.log*
.env

# ---- Python ----
__pycache__/
*.py[cod]
.venv/

# ---- Java ----
target/
*.class
*.jar

# ---- 构建产物 ----
dist/
build/

# ---- 操作系统 ----
.DS_Store
Thumbs.db

# ---- IDE ----
.idea/
*.iml
```

各大语言/框架的完整模板可参考 GitHub 官方仓库 <https://github.com/github/gitignore>。

### 9.3 已跟踪文件的注意点（重要）

`.gitignore` **只防患于未然**——对**已经被跟踪**（提交过）的文件不生效：

```bash
# .env 已经被提交过，现在想停止跟踪但保留本地文件
git rm --cached .env
git commit -m "chore: stop tracking .env"

# 此后 .gitignore 中的 .env 规则才真正接管它
```

注意区分：

| 命令 | 效果 |
|------|------|
| `git rm --cached <文件>` | 仅取消跟踪，**保留本地文件**（从下次提交起不再包含它） |
| `git rm <文件>` | 取消跟踪**并删除本地文件** |

调试忽略规则是否生效：

```bash
git check-ignore -v path/to/file
```

另外，`git add -f <文件>` 可以**强行添加**被忽略的文件（`-f` = force），仅在你明确知道为什么时使用。

---

## 10. 常见问题

### 10.1 提交前 / 提交后发现漏了文件

```bash
# 还没 commit：补上一起提交即可
git add forgotten.js

# 已经 commit 但还没 push：并入上一次提交
git add forgotten.js
git commit --amend --no-edit      # 保持原提交信息不变
```

若已推送则**不要 amend**，直接补一个新提交。

### 10.2 提交信息写错了

```bash
# 未推送：修改最近一次提交信息
git commit --amend -m "feat: add login page with validation"

# 已推送：最稳妥是补一个修正提交（或与维护者沟通后强制覆盖个人分支）
git commit -m "docs: fix typo in previous message"
```

### 10.3 推送被拒绝（`! [rejected] ... (fetch first)`）

原因：远程分支上有你本地没有的提交（别人先推了）。解决：

```bash
git pull --rebase          # 拉取远程提交并把你的提交"接"在其后
# 如有冲突：解决 → git add → git rebase --continue
git push
```

避免方案：养成**推送前先 pull** 的习惯。

### 10.4 合并冲突怎么办

见 [6.4 节](#64-合并冲突的解决步骤通用流程) 的五步流程。核心：手改文件 → 删标记 → `git add` → `git commit`。

### 10.5 误删了分支 / 丢失了提交

只要提交对象还没被 Git 清理回收，都能救回来：

```bash
git reflog                 # 查看 HEAD 的全部移动轨迹
# 输出示例：
# a1b2c3d HEAD@{0}: checkout: moving from feature-x to main
# e4f5a6b HEAD@{1}: commit: feat: half-done work   ← 被误删分支的最新提交

git branch recovered-x e4f5a6b   # 基于该哈希重建分支
```

> `git reflog` 记录本仓库 90 天内（默认）的 HEAD 移动，是"后悔药"的主要来源；但它**救不回从未提交过的改动**——重要工作请勤 commit。

### 10.6 如何查看远程仓库地址

```bash
git remote -v

# 更详细的信息（分支跟踪关系等）
git remote show origin
```

想更换地址：

```bash
git remote set-url origin git@github.com:user/new-repo.git
```

---

## 11. 最佳实践

### 11.1 小步提交

一次提交只做一件事（一个功能、一个修复）。提交粒度小，回退、定位问题、代码评审都更容易。忌讳"一周攒一个大提交"。

### 11.2 提交信息规范

推荐 **Conventional Commits** 风格（`类型: 描述`），标题用一句话讲清"做了什么"：

```bash
feat: add user login page
fix: correct token expiration check
docs: update API examples in README
refactor: extract login form component
test: add cases for token refresh
chore: upgrade webpack to v5
```

要点：

- 标题用祈使句、不超过 50 字符左右、结尾不加句号；
- 中文团队用中文描述同样可以（如 `feat: 新增登录页`），关键是全团队统一；
- 较大改动在正文补充"为什么改"（`git commit` 不带 `-m` 会打开编辑器写多行信息）。

### 11.3 先 pull 再 push

推送前先 `git pull`（或 `git pull --rebase`）同步远程，能避免绝大多数"推送被拒绝"。每天开工时也先 pull 一次，基于最新代码开发。

### 11.4 谨慎 rebase / 谨慎强制推送

黄金法则：**不要对已经推送、且他人可能基于其工作的分支执行 rebase（或 amend、reset 后强推）**——这些操作改写历史，会让协作者的仓库陷入混乱。rebase 适合整理**只属于你自己、还没共享**的提交；确需覆盖远程时用 `git push --force-with-lease` 而不是 `--force`。

### 11.5 不要提交密钥与大文件

- 密钥、口令、`.env` 一律进 `.gitignore`；**一旦误推，视为已泄露，立即更换密钥**（仅删除文件不够，历史里仍可看到）；
- 大文件让仓库膨胀且难恢复，媒体/二进制产物用对象存储或 [Git LFS](https://git-lfs.com/) 管理；
- 克隆大历史仓库可用 `git clone --depth 1`。

### 11.6 常用分支策略简述

| 策略 | 模式 | 适用 |
|------|------|------|
| 单主干（trunk-based） | 所有人直接提交 `main`，靠小步提交 + CI 保障质量 | 小团队、个人项目 |
| GitHub Flow | 每个需求从 `main` 切出功能分支 → Push → Pull Request → 评审 → 合并 → 部署 | 绝大多数中小团队（推荐入门） |
| Git Flow | `main`（发版）+ `develop`（开发）+ `feature/*` + `release/*` + `hotfix/*` 多类分支 | 版本发布节奏严格的大型项目 |

新手建议：**主分支保持可用，功能都在短生命周期的功能分支上开发，通过 Pull Request 合并**——简单、清晰、够用。

---

## 12. 命令速查表

### 配置与初始化

| 命令 | 作用 | 常用示例 |
|------|------|----------|
| `git config` | 读写配置 | `git config --global user.name "Your Name"` |
| `git init` | 初始化仓库 | `git init -b main` |
| `git clone` | 克隆远程仓库 | `git clone https://github.com/user/repo.git` |

### 日常三步

| 命令 | 作用 | 常用示例 |
|------|------|----------|
| `git status` | 查看状态 | `git status -s` |
| `git add` | 暂存改动 | `git add .` |
| `git commit` | 提交 | `git commit -m "feat: xxx"` |
| `git diff` | 查看差异 | `git diff --staged` |

### 历史与标签

| 命令 | 作用 | 常用示例 |
|------|------|----------|
| `git log` | 查看历史 | `git log --oneline --graph --all` |
| `git show` | 查看某提交详情 | `git show a1b2c3d` |
| `git tag` | 管理标签 | `git tag -a v1.0.0 -m "Release 1.0.0"` |

### 分支

| 命令 | 作用 | 常用示例 |
|------|------|----------|
| `git branch` | 管理/列出分支 | `git branch -a` |
| `git switch` | 切换分支（新） | `git switch -c feature-x` |
| `git checkout` | 切换/检出（旧） | `git checkout -b feature-x` |
| `git merge` | 合并分支 | `git merge feature-x` |

### 远程

| 命令 | 作用 | 常用示例 |
|------|------|----------|
| `git remote` | 管理远程关联 | `git remote -v` |
| `git push` | 推送 | `git push -u origin main` |
| `git pull` | 拉取并合并 | `git pull --rebase` |
| `git fetch` | 仅拉取不合并 | `git fetch --prune` |

### 撤销与恢复

| 命令 | 作用 | 常用示例 | 风险 |
|------|------|----------|------|
| `git restore` | 恢复文件/取消暂存 | `git restore --staged file` | 丢弃工作区改动时不可逆 |
| `git reset` | 回退本地提交 | `git reset --soft HEAD~1` | **`--hard` 极高危** |
| `git revert` | 反做已推送提交 | `git revert HEAD` | 安全 |
| `git stash` | 临时保存现场 | `git stash push -m "wip"` | `clear` 需谨慎 |
| `git reflog` | 查看 HEAD 轨迹（后悔药） | `git reflog` | 只读，安全 |

---

> **学完自测**：如果你能独立完成——初始化仓库、配置身份、添加并提交、查看历史、创建与合并分支、关联远程并推送/拉取、解决一次冲突、撤销一次错误提交、正确使用 `.gitignore`——说明你已经具备日常 Git 协作能力。之后遇到问题，回到第 5 章查命令、第 10 章查排错即可。
