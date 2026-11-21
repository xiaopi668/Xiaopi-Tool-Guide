# 断点调试：比 console.log 快十倍

> 让程序停在你想看的那一行，变量就摆在旁边

`VS Code` · 建议 16 分钟 · 进阶

## 学完这一节，你会

- 会打行断点和条件断点
- 看懂变量、监视、调用栈、断点四个面板
- 写出一份能跑的 launch.json
- 断点不生效时知道从哪查

## console.log 的问题不是丑

console.log 能回答「这一行执行了吗、变量是多少」，但要问「它在第几次循环里开始变错、谁把参数改成了 undefined」，就得反复加打印、重跑、删打印。异步代码里更糟：刷屏的输出根本分不清哪条属于哪次请求。

断点（breakpoint）让程序停在你想看的那一行，变量就摆在旁边的面板里，还能一层层往回看调用栈。它不修改代码，也不用重跑。

## 第一次打断点，四步

### 1. 在行号左边点一下

点行号左侧的空白处，出现红点就是行断点（line breakpoint）。只有可执行语句才能打，空行和纯注释行打不上。

### 2. 按 F5 启动

第一次按 F5，编辑器让你选环境（Select environment），选 Node.js。程序跑起来后会停在红点那一行，整行高亮。

### 3. 看左边的面板

变量（VARIABLES）里是当前作用域的值，调用栈（CALL STACK）显示谁调用了这里，断点（BREAKPOINTS）列出所有断点。

### 4. 按 F10 一行一行走

F10 单步跳过（Step Over），不进函数；F11 单步进入（Step Into）会钻进被调用的函数。走几步就知道从哪一行开始不对。

## 调试面板四个区块

| 区块 | 看什么 | 怎么用 |
| --- | --- | --- |
| 变量 VARIABLES | 当前作用域的局部变量、闭包、this | 点三角展开对象；双击值可以临时改，只在这次运行有效 |
| 监视 WATCH | 你手写的表达式 | 点加号输入 user.items.length，每走一步都会重新求值 |
| 调用栈 CALL STACK | 谁调用了当前函数 | 点上一层栈帧，能看到那一层的局部变量 |
| 断点 BREAKPOINTS | 所有断点的清单 | 取消勾选可以临时屏蔽，不用删掉再打一遍 |

## 调试键

| 操作 | macOS | Windows |
| --- | --- | --- |
| 启动 / 继续 | <kbd>F5</kbd> | <kbd>F5</kbd> |
| 单步跳过，不进函数 | <kbd>F10</kbd> | <kbd>F10</kbd> |
| 单步进入函数 | <kbd>F11</kbd> | <kbd>F11</kbd> |
| 单步跳出当前函数 | <kbd>⇧</kbd><kbd>F11</kbd> | <kbd>Shift</kbd><kbd>F11</kbd> |
| 停止调试 | <kbd>⇧</kbd><kbd>F5</kbd> | <kbd>Shift</kbd><kbd>F5</kbd> |
| 重启调试 | <kbd>⌘</kbd><kbd>⇧</kbd><kbd>F5</kbd> | <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>F5</kbd> |
| 在当前行切换断点 | <kbd>F9</kbd> | <kbd>F9</kbd> |
| 打开运行和调试面板 | <kbd>⌘</kbd><kbd>⇧</kbd><kbd>D</kbd> | <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>D</kbd> |
| 打开调试控制台 | <kbd>⌘</kbd><kbd>⇧</kbd><kbd>Y</kbd> | <kbd>Ctrl</kbd><kbd>Shift</kbd><kbd>Y</kbd> |

## 最小的 Node 调试配置 launch.json

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "启动 src/index.js",
      "program": "src/index.js",
      "cwd": ".",
      "console": "integratedTerminal",
      "skipFiles": ["<node_internals>/**"]
    }
  ]
}
```

> 放在项目根目录的 .vscode/launch.json。program 和 cwd 都按相对路径写，换机器也能用；如果团队里每个人的目录结构不同，也可以把入口写成绝对的 workspace 变量，VS Code 支持一组内置变量，在 launch.json 的输入框里按 ⌃ 空格 能看到候选。

## launch.json 里每行在说什么

| 字段 | 含义 |
| --- | --- |
| version | 配置文件版本，固定写 0.2.0 |
| type | 调试器类型。node 是内置的 Node.js 调试器；python、go、pwa-chrome 要装对应插件才有 |
| request | launch 表示由 VS Code 启动程序；attach 表示连上一个已经在跑的进程 |
| name | 下拉框里显示的名字，随便起，有多套配置时靠它区分 |
| program | 入口文件，相对路径按 cwd 解析。写错会报 Cannot find module |
| cwd | 程序运行时的工作目录，填 . 就是当前工作区根目录；读配置文件、找相对路径资源全靠它 |
| console | integratedTerminal 让程序输出走集成终端，能接受键盘输入；默认的 internalConsole 是只读的 |
| skipFiles | F11 单步时跳过这些文件，避免钻进 Node 内部源码 |

## 条件断点：只在出问题的那一次停下

右键行号左边的红点，选 Edit Breakpoint，再选 Expression，填一个条件：只有条件为真时才停。循环里排查「第几次开始错」，这一条能省掉几百次按 F5。

同一个菜单里还有 Hit Count，填 500 表示第 500 次经过这里才停，适合复现「跑一阵子才出问题」的 bug。还有 Log Message 模式，只打印不暂停，等于一个不用删的 console.log。

> **注意：断点不生效时先看这四种**
>
> **断点是空心灰圈。**悬停显示 `Breakpoint set but not yet bound`，意思是调试器还没把这段代码加载进来：路径映射不对、跑的是编译后的文件，或者那段代码根本没被执行到。
>
> **报找不到模块。**终端里出现 `Error: Cannot find module` 和 `MODULE_NOT_FOUND`，就是 program 写错了，先确认相对路径是相对 cwd 还是相对工作区根目录。
>
> **端口被占。**`Error: listen EADDRINUSE: address already in use :::3000` 说明上一次调试没停干净，或者本地 dev server 还开着。
>
> **TypeScript 断点位置偏移。**要打开 sourceMap 并设置 outFiles，否则断点会落在编译产物上。

> **提示：attach：连上已经在跑的进程**
>
> 有些进程不归你启动：PM2 托管的服务、容器里的 Node、已经跑了一小时的脚本。这时用 attach：先让进程带 `--inspect` 启动，再用 request 为 attach 的配置连上去。
>
> 连接配置最少只要三项：type 为 node、request 为 attach、port 为 9229。连上之后断点、变量、调用栈和 launch 模式完全一样。

## 带调试端口启动一个脚本

```bash
$ node --inspect-brk src/index.js
Debugger listening on ws://127.0.0.1:9229/3f1c2d9a-6b1e-4f0a-9c2e-7d5b8a1f0c33
For help, see: https://nodejs.org/en/docs/inspector
```

> 加 -brk 会在第一行就停下来等你连，不加则直接跑，只在你打断点的地方停。执行后进程会一直等着，按 Ctrl+C 结束。端口 9229 默认只监听本机，别改成 0.0.0.0，那等于把调试口开给整个内网。

## 两种查 bug 的方式

| 到处塞 console.log | 断点加监视 |
| --- | --- |
| 改代码、重跑、再看输出 | 不改一行代码，停在你关心的位置 |
| 循环里刷出几百行，找不到关键那条 | 变量、调用栈、闭包全在面板里 |
| 异步请求交错，日志对不上号 | 条件断点直接跳到出错那一次 |
| 查完忘了删，一起提交进仓库 | 关掉调试，代码一行没动 |

> console.log 不是不能用：跑批处理、看时序、排查只在生产出现的问题时，它仍然最快。区别在于**本地能复现的 bug，先用断点**。

## 动手练一练

写一个十来行的小脚本，让它在循环里算错一个数，然后用断点把它找出来。

1. 写一个 for 循环累加数组，故意把 += 写成 =
2. 在循环体里打行断点，按 F5 启动
3. 编辑断点加条件 i === 3，只在第四次循环停下
4. 在监视里加 arr.length，走几步看值怎么变

**完成标准**：能在监视面板里指出从第三次循环开始不对，说明你真的抓住它了。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. 想让程序停在某一行，最直接的做法是什么？**

- A. 在那行写一句 console.log
- B. 在行号左侧点一下打红点，然后按 F5
- C. 把文件保存一次
- D. 在终端里执行 node

<details><summary>答案</summary>

**B**。行断点在编辑器里点出来，F5 启动后程序会在那一行暂停，变量和调用栈都在左边的面板里，完全不用改代码。

</details>

**2. 启动调试后报 Cannot find module，最可能是什么问题？**

- A. Node 没有安装
- B. launch.json 里的 program 或 cwd 写错
- C. 断点打得太多了
- D. 编辑器版本太旧

<details><summary>答案</summary>

**B**。program 是相对 cwd 解析的入口文件路径。cwd 指错目录或文件名拼错，Node 找不到文件就会抛 MODULE_NOT_FOUND。

</details>

**3. 断点显示成一个空心灰圈，说明什么？**

- A. 断点已经命中过了
- B. 调试器还没把这段代码加载进来
- C. 文件被删掉了
- D. 代码里有语法错误

<details><summary>答案</summary>

**B**。空心圈表示断点已设置但还没绑定（not yet bound）：路径映射不对、代码没被执行，或者跑的是编译产物而源映射没配好。

</details>

</details>

---

[← 插件：少而精](04-插件-少而精.md) · [目录](../Home.md) · [远程开发：在本机编辑服务器上的代码 →](06-远程开发-在本机编辑服务器上的代码.md)
