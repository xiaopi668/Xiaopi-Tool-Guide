# 终端、Shell 与一条命令的解剖

> 从提示符开始，把一条命令拆成零件，顺手记住几个救命按键。

`终端与系统` · 建议 10 分钟 · 入门

## 学完这一节，你会

- 分清终端、Shell 和命令行这三个词
- 看懂提示符，知道自己站在哪个目录
- 把一条命令拆成程序名、选项、参数
- 遇到报错时先按哪几个键，心里有数

## 三个词，先分清

**终端**（terminal）是你眼前这个能打字的窗口，它本身不执行任何命令，只负责把你的按键交给 Shell、把结果画回屏幕。macOS 自带 Terminal.app，也有 iTerm2；Windows 上是 Windows Terminal；VS Code 下面那个面板叫集成终端，本质一样。

**Shell** 是躲在终端后面的解释器：读你这一行，解析成程序加参数，执行，再把结果送回来。macOS 默认 zsh，多数 Linux 默认 bash，Windows 的 PowerShell 是另一套语法。

**命令行**（command line）不是某个软件，是一种「一行文字换一个动作」的交互方式。你用鼠标点十下能完成的事，这里一行写完。

## 谁坏了，你就看到什么

| 概念 | 它是什么 | 坏掉时的症状 |
| --- | --- | --- |
| 终端 | 承载输入输出的窗口程序 | 字打不出来、中文乱码、窗口卡死 |
| Shell | 解析并执行命令的解释器 | 提示符变了、报 syntax error |
| 命令 | 一个可执行程序加它的选项和参数 | command not found、illegal option |

## 提示符在告诉你什么

行首那串东西叫**提示符**（prompt），通常由用户名、主机名、当前目录拼成，家目录会缩写成 `~`。

结尾的符号最关键：`$` 表示你是普通用户，`#` 表示你已经是 root（超级用户）。zsh 的默认提示符长这样：`xiaopi@MacBook study %`，bash 长这样：`xiaopi@ubuntu:~/study$`。末尾的 `%` 也是普通用户，别被它吓到。

看到 `#` 就当作「这一行敲错，没有回收站」来对待。

## 先确认自己在哪、用的是什么 Shell

```bash
$ whoami; echo $SHELL; echo $ZSH_VERSION; pwd
xiaopi
/bin/zsh
5.9
/Users/xiaopi/study
```

> 用分号可以把几条命令写在一行，按顺序执行。$SHELL 是登录时记下的默认 Shell，$ZSH_VERSION 是当前这个 zsh 的版本；写 bash 的人用 echo $BASH_VERSION 看版本。

## 一条命令的三个零件

拆开看，永远是三块：**程序名**（跑哪个程序）、**选项**（怎么跑，通常以横线开头）、**参数**（对谁跑，通常是文件或目录）。

`ls -la /tmp` 里，`ls` 是程序名，`-la` 是两个短选项合写（`-l` 长格式、`-a` 含隐藏文件），`/tmp` 是参数。短选项能合并，长选项不能：`--all` 只能单独写。

## 拆一条真实命令

```bash
$ ls -la /tmp | head -n 4
total 0
drwxrwxrwt  12 root    wheel  384  3 14 09:21 .
drwxr-xr-x   6 root    wheel  192  3 14 09:20 ..
-rw-r--r--   1 xiaopi  wheel  128  3 14 09:19 note.txt
```

> 程序名 ls，选项 -la，参数 /tmp。竖线后面是第二条命令，这一课先不管它，第四课专门拆。

## 命令的四个部位

| 部位 | 例子 | 规矩 |
| --- | --- | --- |
| 程序名 | ls、grep、node、git | 必须能在 PATH 里找到 |
| 短选项 | -l、-a、-n 5 | 单横线，可以合并成 -la |
| 长选项 | --help、--version | 双横线，不能合并 |
| 参数 | /tmp、app.js、"my file.txt" | 空格是分隔符，有空格就加引号 |

## 终端里的救命键，先记五个

| 操作 | macOS | Windows |
| --- | --- | --- |
| 补全命令或路径 | <kbd>⇥</kbd> | <kbd>Tab</kbd> |
| 调出上一条命令 | <kbd>↑</kbd> | <kbd>↑</kbd> |
| 中断当前程序 | <kbd>⌃</kbd><kbd>C</kbd> | <kbd>Ctrl</kbd><kbd>C</kbd> |
| 清屏 | <kbd>⌃</kbd><kbd>L</kbd> | <kbd>Ctrl</kbd><kbd>L</kbd> |
| 反向搜索历史 | <kbd>⌃</kbd><kbd>R</kbd> | <kbd>Ctrl</kbd><kbd>R</kbd> |
| 退出当前 Shell | <kbd>⌃</kbd><kbd>D</kbd> | <kbd>Ctrl</kbd><kbd>D</kbd> |

> **注意：command not found 不是命令写错，是找不到**
>
> 报错原文：`zsh: command not found: node`，bash 下是 `bash: node: command not found`。含义是 Shell 沿着 `PATH` 里列的每个目录找了一遍，没有找到叫 `node` 的可执行文件。
>
> 三种常见原因：真没装；装了但目录不在 PATH 里（nvm、Homebrew 最常踩）；装的是另一个名字（`python3` 而不是 `python`）。
>
> 排查顺序：`command -v node` 看能不能找到，`echo $PATH` 看找过哪些目录。第五课会拆开讲 PATH。

## 五分钟自己走一遍

### 1. 看清单前状态

敲 `pwd` 和 `echo $SHELL`，把路径和 Shell 名字记下来，后面出错时有用。

### 2. 用 Tab 代替手打

敲 `cd ~/Des` 然后按 Tab，路径会自动补全成 `~/Desktop`。补不上就停手看拼写，不要直接回车。

### 3. 故意触发一次报错

敲 `nodee --version`，看 command not found 的原文；再敲 `node --version` 对比成功的输出。

### 4. 用 Ctrl+C 逃生

敲 `ping 127.0.0.1`，它每秒刷一行；按 ⌃C（Windows 为 Ctrl+C）停下，看提示符回来。

> **提示：man 和 --help 怎么选**
>
> `man ls` 打开完整手册：`/` 搜索、`n` 下一个匹配、`q` 退出。输出太长想慢慢翻，`man ls | less` 效果一样。
>
> 不想离开当前屏幕就用 `ls --help` 打一份简表。注意 macOS 自带的部分命令（比如 `ls`）没有 `--help`，会回你 `ls: illegal option -- -`，这时只能 man。记不住选项先 `man 命令名`，比搜索快得多。

## 三个新手误会

| 以为是这样 | 实际是这样 |
| --- | --- |
| 终端就是命令行 | 终端是窗口，命令行是交互方式，Shell 才是执行者 |
| 关掉窗口只是关掉显示 | 窗口关掉，前台进程收到挂断信号一起结束 |
| 按 Ctrl+C 会把电脑弄坏 | Ctrl+C 只是给前台进程发中断信号，系统毫发无损 |

> 这三个误会搞清楚了，你才敢在终端里随便试。

## 动手练一练

打开终端，按顺序做完下面四步。每步都先看清输出再往下走，卡住了就按 ⌃C 重来一次。

1. 确认当前目录与默认 Shell —— `pwd; echo $SHELL`
2. 用 Tab 补全进入你的项目目录 —— `cd ~/Projects`
3. 翻一遍 ls 的手册，用 q 退出 —— `man ls`
4. 故意触发一次 command not found 并解释原因 —— `nodee --version`

**完成标准**：你能说出 $ 与 # 的区别，也知道报错时该先查 command -v 还是先看 PATH。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. 提示符结尾是 # 而不是 $，说明什么？**

- A. Shell 换成了 bash
- B. 当前是 root 用户，命令会直接生效
- C. 终端进入了只读模式
- D. 上一条命令还没结束

<details><summary>答案</summary>

**B**。井号是 root 的提示符标记。此时每条命令都以超级用户身份执行，删文件、改系统配置都不再问密码，所以回车前务必先看清路径。

</details>

**2. 敲 ls -la /tmp 时，哪一部分是参数？**

- A. ls
- B. -la
- C. /tmp
- D. 整条命令都是参数

<details><summary>答案</summary>

**C**。ls 是程序名，-la 是合并写的两个短选项（-l 长格式、-a 含隐藏文件），/tmp 才是这条命令作用的对象。选项可以合并，参数不能当选项用。

</details>

**3. 命令卡在不停刷屏的状态，最干净的停止方式？**

- A. 关掉终端窗口
- B. 按 ⌃C（Windows 为 Ctrl+C）
- C. 按 ⌃L 清屏
- D. 再开一个新终端窗口

<details><summary>答案</summary>

**B**。⌃L 只清屏，进程还在后台刷。关窗口会连带杀掉正在跑的任务。⌃C 给前台进程发中断信号，提示符马上回来，副作用最小。

</details>

</details>

---

[目录](../Home.md) · [文件与目录的十个命令 →](02-文件与目录的十个命令.md)
