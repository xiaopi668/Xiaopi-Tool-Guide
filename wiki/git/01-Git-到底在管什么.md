# Git 到底在管什么

> 提交是快照不是差异，三个区域决定改动去了哪里

`Git 与版本控制` · 建议 12 分钟 · 入门

## 学完这一节，你会

- 说清工作区、暂存区、本地仓库的差别
- 理解提交是完整快照而不是差异
- 知道 HEAD 和分支指针各自在指什么
- 能用一条命令看清项目当前状态

## 先看一个所有人都干过的事

交稿前一天，你的桌面大概是这样的：`论文.docx`、`论文-最终版.docx`、`论文-最终版2.docx`、`论文-最终版-真的最后一版.docx`。

这不是懒。你只是需要回答三个问题：**上周三那版和现在差在哪、哪一版是老师看过的、不满意能不能退回去**。文件名回答不了。它只记录一个时间点，顺序全得靠你脑子里记着。

版本控制（version control）就是把这三个问题变成一条可查询的历史。Git 是目前最常用的实现。

## 复制文件夹 vs 版本控制

| 复制文件夹的做法 | 用 Git 的做法 |
| --- | --- |
| 论文-最终版.docx、论文-最终版2.docx 越堆越多 | 只有一个 论文.docx，改完存盘 |
| 靠文件名和修改时间猜哪份最新 | 每次改动是一条提交，带作者、时间、说明 |
| 想知道改了什么，只能把两份并排肉眼比对 | git diff 随时算出两份快照的差别 |
| 两个人同时改，只能微信传来传去 | 两个人各改一个分支，最后合并 |
| 手滑删掉文件夹，历史当场清零 | 远程仓库还留着一份完整历史 |

> 文件名记录的是时间点，提交记录的是因果关系：谁在什么基础上改了什么、为什么改。

## 提交是快照，不是差异

很多人以为 Git 存的是每次改动的补丁。不是。每条提交（commit）记录的是**整个项目在那一刻的完整样子**：有哪些文件、每个文件什么内容、目录怎么排。

你看到的 diff 是现算的：Git 拿出两个快照一比，把不一样的行标出来。所以你能直达任何一个提交，因为那个提交本身是完整的，不需要从头重放补丁。

快照也不等于把磁盘复制一遍。Git 按内容算哈希（hash），内容相同的文件只存一份。一个 200MB 的仓库塞一万条提交，通常也不会变成 200GB。

## 三个区域，三条命令

| 区域 | 里面放什么 | 谁把东西放进去 | git status 里的说法 |
| --- | --- | --- | --- |
| 工作区（working directory） | 你在编辑器里打开、正在改的文件 | 你自己按保存 | Changes not staged for commit |
| 暂存区（staging area） | 下一次提交要包含的那些改动 | git add | Changes to be committed |
| 本地仓库（repository） | .git 目录里的提交历史 | git commit | nothing to commit, working tree clean |

## 一条命令看清三个区域

```bash
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	modified:   article.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
	modified:   config.json

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	notes.txt
```

> 三段分别对应暂存区、工作区、以及还没被 Git 管起来的新文件。输出里括号中的提示语，就是 Git 在告诉你下一步该敲哪条命令。

## HEAD 和分支只是指针

HEAD 是**你现在站在哪**的记号。正常状态下它指向一个分支名，分支名再指向某条提交。你在哪个分支上提交，HEAD 就带着那个分支一起往前挪一格。

分支（branch）本身轻得离谱：它就是一个存着提交哈希的文本文件，通常 41 个字节。`git switch -c feature/login` 不会复制你的代码，只是多写了一个指针。

提交则相反，它是内容寻址（content-addressable）的：哈希由内容算出来，改一个字符哈希就变。所以历史只能往后追加。所谓改历史，其实是造一批新提交，再让分支指过去。

## 历史长什么样

```bash
$ git log --oneline --decorate --graph --all
* 4d8e2f1 (HEAD -> main) docs: 补充第二节的例子
* 2b7d9e4 feat: 增加草稿自动保存
| * 9a1c3f7 (origin/feature/search) feat(search): 支持按标签过滤
|/
* 8c1d0a7 chore: 初始化项目
```

> HEAD -> main 说明你现在在 main 上。每行开头一个星号就是一条提交，竖线表示分支的分叉与合流，all 表示连远程分支和别的分支一起画出来。

## 提交里到底存了什么

```bash
$ git cat-file -p HEAD
tree 8f3c1a0d9e2b4c5a6f7d8e9a0b1c2d3e4f5a6b7c
parent 2b7d9e4a1c3f5d7e9b0a2c4e6f8d1b3a5c7e9f0d
author Lin <lin@example.com> 1730451600 +0800
committer Lin <lin@example.com> 1730451600 +0800

docs: 补充第二节的例子
```

> 一条提交里存的是指向整棵目录树的 tree 哈希、父提交的哈希、作者与时间、说明文字，没有一行 diff。这就是快照的实物样子。

> **提示：不可变历史带来的两个好处**
>
> 第一，**破坏会被发现**：如果有人悄悄改了旧提交，哈希对不上，`git fsck` 会报错。第二，**回退永远可靠**：每个提交都完整，回到三个月前不需要重放任何东西。
>
> 代价是你没法真正改掉一条已推送的提交。`git commit --amend`、`git rebase` 都是造新提交，旧的那条还在 reflog 里躺着，第 13 课讲怎么靠它救命。

> **注意：别把 .git 当成可有可无的隐藏文件夹**
>
> 删掉 `.git`，你手里的文件还是最新的，但**全部历史当场消失**：分支、提交、作者、说明、reflog 一起没，回收站也救不回来。
>
> 另一个误解是把 Git 当网盘。本地仓库只在本机，硬盘坏了就全没了，备份来自远程仓库（remote），第 8 课配地址，第 11 课细讲推送。
>
> 还有个会毁掉别人工作的操作：对整个仓库跑 `git clean -xfd`，它会连 .gitignore 里忽略的本地配置文件一起删。

## 一次改动要穿过三个区域

### 1. 改文件

在编辑器里改完保存。此时只有工作区变了，`git status` 里显示的是红字 modified，暂存区还是旧内容。

### 2. git add 挑出这次要提交的部分

把文件从工作区放进暂存区。这一步真正的价值是**让你先挑**：改了两个功能，可以只提交其中一个。

### 3. git commit 写进历史

暂存区的内容被打包成一条快照，HEAD 和当前分支一起往前挪一格，工作区不受影响。

### 4. git push 送到远程

本地仓库有了，别人还看不到。推送之后远程仓库才有这条提交，同事才拉得到。

## 动手练一练

建一个空仓库，制造一次「改了但没暂存」和一次「暂存了没提交」，用 status 分别看清它们停在哪一层。

1. 新建目录并初始化仓库 —— `mkdir git-lab && cd git-lab && git init`
2. 写一个文件，看看 status 怎么描述未跟踪文件 —— `echo hello > notes.txt && git status`
3. 暂存它，再跑一次 status，对比两段输出的标题差异 —— `git add notes.txt && git status`
4. 提交，然后查看这条提交的 tree 与父提交 —— `git commit -m "docs: 第一条提交" && git cat-file -p HEAD`

**完成标准**：你能指着 status 的三段输出，说出每段对应哪个区域，这一课就通了。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. git diff 显示的差异是怎么来的？**

- A. Git 保存了每次改动的一行行补丁，diff 就是把补丁读出来
- B. Git 保存完整快照，diff 是拿两个快照现场比较算出来的
- C. Git 向远程服务器请求两版文件再比对
- D. Git 记录了文件的修改时间，按时间推算改了什么

<details><summary>答案</summary>

**B**。提交存的是整棵目录树的快照，diff 只是展示层的计算结果。所以任意两个提交都能比，不依赖中间有没有记录过补丁。

</details>

**2. 你改完一个文件，执行 git commit 后提示 nothing to commit。最可能的原因是什么？**

- A. 文件被 .gitignore 忽略了，需要先 git add -f
- B. 改动没有 git add，还停在工作区，没进暂存区
- C. Git 仓库损坏，需要重新 git init
- D. 当前分支是 main，main 上不允许提交

<details><summary>答案</summary>

**B**。commit 只把暂存区里的内容打包成提交。工作区的改动没有 add 就不会被带走，status 里会以红字 modified 单独列出来，等你自己处理。

</details>

**3. git switch -c feature/login 这条命令在磁盘上做了什么？**

- A. 把你的代码完整复制一份到新目录
- B. 新增一个指向当前提交的指针文件，并把 HEAD 切过去
- C. 在远程仓库创建同名分支并立即推送
- D. 为分支单独建一个 .git 目录，历史从零开始

<details><summary>答案</summary>

**B**。分支就是一个记录提交哈希的小文件，创建成本接近零。代码目录不动，变的只是 HEAD 指向哪个分支名。

</details>

</details>

---

[← 进程、端口与网络](../terminal/06-进程-端口与网络.md) · [目录](../Home.md) · [安装与第一次配置 →](02-安装与第一次配置.md)
