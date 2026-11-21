# 日常三件套：status / add / commit / log

> 每天重复几十次的四条命令，把每一步看清

`Git 与版本控制` · 建议 15 分钟 · 基础

## 学完这一节，你会

- 用 status 和 diff 判断改动停在哪一层
- 说清 git add 文件与 git add . 的边界
- 写出别人看得懂的提交信息
- 用 log 的图形模式读懂分支历史

## 开工第一条命令永远是 status

不知道现在什么状态时，第一条命令永远是 `git status`。它不改任何东西，只回答三件事：你在哪个分支、哪些改动已经进了暂存区、哪些还留在工作区。

输出很长但结构固定：上面 Changes to be committed 是暂存区，下面 Changes not staged 是工作区，最后 Untracked files 是还没被 Git 管的新文件。看不懂英文也没关系，括号里 use git add... 那句就是它建议你敲的命令。

想看得紧凑，加 `-s`（short）和 `-b`（branch）。

## 缩短版状态

```bash
$ git status -sb
## main...origin/main [ahead 2]
 M article.md
M  src/app.js
A  postcss.config.js
 D old-notes.md
?? notes.txt
```

> 第一行是分支与跟踪关系：本地 main 比 origin/main 多两条提交。两个字符位最关键，左列是暂存区，右列是工作区。同样是 M，在左列表示已暂存，在右列表示只改了没加。

## 短标记怎么读

| 标记 | 出现在哪一列 | 含义 | 下一步 |
| --- | --- | --- | --- |
| M | 右列 | 文件改过，还没进暂存区 | git add 文件名 |
| M | 左列 | 改动已暂存，等提交 | git commit |
| A | 左列 | 新文件已加入暂存区 | git commit |
| D | 任一列 | 文件被删除，在左列表示删除已暂存 | git add 或 git restore |
| ?? | 两列 | 未跟踪的新文件 | git add 文件名 |
| !! | 两列 | 被 .gitignore 忽略，一般不显示 | 不用管 |

## add 到哪一步算够

`git add 文件名` 只处理你点名的那个文件；`git add .` 处理当前目录下的所有变化，包括新文件、修改和删除。

风险很具体：`git add .` 会把 `.env`、`node_modules`、编译产物、你顺手拷进来的导出文件一起推进去。加错了还能救，`git restore --staged 文件名` 把它撤下，文件内容一个字不丢。

不想整块提交时用 `git add -p`：它把改动切成小段逐段问 y/n，只把这一段放进暂存区，很适合「一次改了两个功能，只想提交其中一个」。

## add 的五种写法

```bash
git add article.md                 # 只加一个文件
git add src/                       # 加整个目录下的变化
git add .                          # 当前目录下全部变化，含新文件、修改、删除
git add -u                         # 只处理已跟踪文件的修改和删除，不含新文件
git add -p src/app.js              # 分块挑选，逐段确认 y / n / s
git restore --staged article.md    # 加错了撤下来，文件内容不变
```

> add 只动暂存区，磁盘上的文件不会变。所以 add 本身不危险，危险的是 add 之后又 commit 又 push。

## diff 的两个方向

`git diff` 比的是**工作区 vs 暂存区**，也就是你还没打算提交的改动。`git diff --staged` 比的是**暂存区 vs 最后一次提交**，也就是这次提交真正要带走的东西。

养成一个习惯：commit 之前先跑 `git diff --staged`。输出为空说明暂存区是空的，commit 会直接失败；看到不该出现的行，立刻用 restore --staged 撤下。

review 别人的分支用 `git diff main...feature`，三个点表示从两条分支的分叉点开始比。

## diff 输出怎么读

```bash
$ git diff
diff --git a/article.md b/article.md
index 3f9a1c2..8c1d0a7 100644
--- a/article.md
+++ b/article.md
@@ -12,6 +12,8 @@
 
 ## 第二节
 
-旧的一行
+新的一行
+再加一行
```

> 减号是被删掉的行，加号是新增的行，@@ 那行说明从第 12 行开始取 6 行上下文。这一份是工作区相对暂存区的差别，如果不 add，提交时带不走。

## commit 信息写三件事

`git commit -m "说明"` 把暂存区打包成一条提交。第一行控制在 50 个字符以内，写**做了什么**，不要写「更新」「修改」「改一下」这类空话。

一次提交只做一件事。把「加登录页」和「修页脚错位」塞进同一条提交，以后想单独回滚其中一个就做不到了。

还有个细节：-m 后面的引号用英文引号。中文引号会被当成普通字符写进提交信息，在 git log 里很刺眼。

## 提交成功的样子

```bash
$ git commit -m "docs: 补充第二节的例子"
[main 4d8e2f1] docs: 补充第二节的例子
 1 file changed, 2 insertions(+), 1 deletion(-)
```

> 方括号里是分支名和这条提交的短哈希，第二行统计改了几个文件、加了几行、删了几行。短哈希留着，回滚和查看都要用。

> **注意：不带 -m 会掉进 Vim**
>
> 忘了 -m，Git 会打开默认编辑器。如果默认是 Vim，你会看到底部一行 `~`，按键全是响铃或乱跳——那是普通模式，不是卡死。
>
> 退出方法：按 `i` 进入编辑，写完说明按 `Esc`，输入 `:wq` 回车保存退出；不想提交就输入 `:q!` 回车放弃。直接关窗口会留下 .git/COMMIT_EDITMSG 和锁文件，下次 commit 报 index.lock 存在。
>
> 想换成熟悉的编辑器：`git config --global core.editor "code --wait"`（VS Code），或把值改成 `nano`。

## 一次巨型提交 vs 小步提交

| 一次巨型提交 | 小步提交 |
| --- | --- |
| 一条提交改了 14 个文件、900 多行 | 一条提交只做一件事，标题一句话讲得清 |
| 提交信息写的是 update | git blame 一眼定位是谁哪次改的 |
| 线上出问题，不知道是哪一行引起的 | git revert 只退那一条，其他功能不受影响 |
| 想回滚只能整条退，把别的功能一起退掉 | review 十分钟看完，敢认真提意见 |
| review 的人看到一半就放弃，等于没 review | git bisect 几次就能找到引入问题的提交 |

> 判断标准很土但好用：这条提交能不能用一句话说清。说不清，就说明它做了太多事。

## 读历史的四个开关

```bash
$ git log --oneline --graph --all
* 7b2e9d4 (HEAD -> main) fix: 修复移动端标题换行
* 4d8e2f1 docs: 补充第二节的例子
* 2b7d9e4 feat: 增加草稿自动保存
* 8c1d0a7 chore: 初始化项目
```

> --oneline 一行一条，--graph 画出分叉与合流，--all 连远程分支一起看。再加 --author=名字 只看某人，git show 短哈希 看某次提交具体改了什么。

## 动手练一练

在一个练习仓库里改两个文件，只暂存其中一个并提交，再从 diff 和 log 里确认自己只提交了想要的那部分。

1. 新建仓库，写两个文件并提交一次作为起点 —— `mkdir daily-lab && cd daily-lab && git init && echo A > a.txt && echo B > b.txt && git add . && git commit -m "chore: 初始化两个文件"`
2. 两个文件都改，然后只暂存 a.txt，用 -sb 看两列的区别 —— `echo A2 >> a.txt && echo B2 >> b.txt && git add a.txt && git status -sb`
3. 对比工作区与暂存区，确认 b.txt 的改动没被带走 —— `git diff && git diff --staged`
4. 提交并读一遍图形化历史 —— `git commit -m "feat: 给 a.txt 追加一行" && git log --oneline --graph --all`

**完成标准**：status -sb 里 a.txt 的 M 在左列、b.txt 的 M 在右列，且 diff --staged 只列出 a.txt，说明两份改动被干净地分开了。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. git status -sb 输出里的「M 」和「 M」有什么区别？**

- A. 没有区别，只是对齐方式不同
- B. 前者改动已进暂存区，后者只改了工作区还没 add
- C. 前者是新增文件，后者是删除文件
- D. 前者表示文件被锁定，后者表示文件可编辑

<details><summary>答案</summary>

**B**。两个字符位左列代表暂存区、右列代表工作区。M 在左列说明改动已 add 待提交，在右列说明还留在工作区，commit 不会带走。记住左列管提交、右列管 add。

</details>

**2. 你执行了 git add .，发现把 .env 也加进了暂存区。正确的补救命令是什么？**

- A. git restore .env 把文件内容也恢复回去
- B. git restore --staged .env 只从暂存区撤下，文件本身不动
- C. git reset --hard 清空所有改动
- D. git rm .env 从磁盘删除这个文件

<details><summary>答案</summary>

**B**。restore --staged 只把条目从暂存区撤回工作区，文件内容原样保留，改动还在。不带 --staged 的 restore 才会用仓库里的版本覆盖文件，那一步不可撤销。

</details>

**3. 执行 git commit 后提示 nothing added to commit but untracked files present，问题出在哪？**

- A. 新文件还没被 git add，暂存区里没有它们
- B. 仓库初始化失败，需要重新 git init
- C. 提交信息太短，被 Git 拒绝
- D. 当前分支受保护，不允许提交

<details><summary>答案</summary>

**A**。未跟踪文件不会自动进入提交。Git 已经明说 untracked files present，先 git add 想提交的那些文件，再 commit。

</details>

</details>

---

[← 安装与第一次配置](02-安装与第一次配置.md) · [目录](../Home.md) · [分支：并行工作的安全区 →](04-分支-并行工作的安全区.md)
