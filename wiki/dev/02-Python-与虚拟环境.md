# Python 与虚拟环境

> 每个项目一套独立的包，不再把系统 Python 装乱

`开发环境与工具链` · 建议 14 分钟 · 基础

## 学完这一节，你会

- 看懂 python3 -V 输出的版本和路径
- 用 venv 建出项目专用的环境
- 分清 python 与 python3 的差别
- 知道 .venv 与清单独该提交

## 先确认你手上是哪个 Python

macOS 和大多数 Linux 自带 Python，版本往往偏旧；Windows 默认没有，要去 python.org 装，安装时**一定要勾上 Add python.exe to PATH**，否则后面每条命令都会提示找不到。

两个命令名要分开看：`python3` 在 macOS 和 Linux 上一直存在；`python` 在很多系统上要么不存在，要么指向早就停更的 Python 2。搞不清就用带 3 的那个。

## 三条命令看清现状

```bash
$ python3 -V && which python3 && python -V
Python 3.11.6
/usr/bin/python3
zsh: command not found: python
```

> 都在任意目录执行，只读版本和路径，不改任何文件。最后一行报错不代表环境坏了，它就是在告诉你这个名字没有对应的程序。

## 为什么必须用 venv

直接 `pip install requests` 会把包装进系统 Python 的 site-packages。问题是**你的项目和操作系统共用同一套包**：A 项目要 requests 2.28，B 项目要 2.31，你只能留一个，另一个项目当场报 `ImportError: cannot import name`。

更麻烦的是系统自带的工具（macOS 的 /usr/bin 工具、Linux 发行版里 apt 装的脚本）依赖特定版本。你升级了它，某个命令行工具就坏掉，而且很难查出是谁干的。

虚拟环境（virtual environment）给每个项目开一个独立目录，里面有自己的一份解释器和 site-packages。装什么都只影响这个目录，删掉就等于什么都没发生。

## 新版系统会直接拦住你

```bash
$ pip3 install requests
error: externally-managed-environment

This environment is externally managed
-> To install Python packages system-wide, try apt install
   python3-xyz, where xyz is the package you are trying to
   install.

note: If you believe this is a mistake, please contact your
Python installation or OS distribution provider.
```

> 这是 PEP 668 的保护，不是 bug。它在提醒你别往系统 Python 里装东西。正确反应是建虚拟环境，而不是加 --break-system-packages 硬闯过去。

## 建环境、激活、确认、退出

### 1. 建环境

在项目根目录执行 `python3 -m venv .venv`，会生成一个 .venv 目录，里面有独立的解释器和 pip，几十 MB。**名字前面的点表示隐藏目录**，用 ls -a 才看得到。这条命令只写当前目录。

### 2. 激活

macOS / Linux：`source .venv/bin/activate`。Windows PowerShell：`.\.venv\Scripts\Activate.ps1`；cmd：`.venv\Scripts\activate.bat`。激活只影响当前这个终端窗口。

### 3. 确认装到哪

激活后提示符前面多出 `(.venv)`，此时 `pip -V` 第一行会显示路径在 .venv 里面。**这一步不要跳过**：后面每条 pip install 都靠它判断有没有装对地方。

### 4. 退出

敲 `deactivate`。它是激活脚本临时注册的命令，不在任何目录里，所以只在激活状态下有效，退出后再敲会报 command not found。

## 从零开始的完整流程

```bash
cd ~/projects/my-app
python3 -m venv .venv              # 建环境，只写当前目录
source .venv/bin/activate          # macOS / Linux 激活
# .\.venv\Scripts\Activate.ps1     # Windows PowerShell 用这行
pip install -r requirements.txt    # 有清单就按清单装
pip install requests               # 没有清单就直接装一个新包
pip freeze > requirements.txt      # 把当前精确版本写进清单
deactivate                         # 退出环境
```

> 顺序不能反：先激活，再 pip install。忘了激活就来装，包会进系统 Python，而当时不会有任何提示。

## 激活前后，两个路径的差别

```bash
$ source .venv/bin/activate && which python && pip -V
/Users/you/projects/my-app/.venv/bin/python
pip 24.0 from /Users/you/projects/my-app/.venv/lib/python3.11/site-packages/pip (python 3.11)
```

> 两行输出都带 .venv 路径，说明你确实在虚拟环境里。如果 which python 指向 /usr/bin/python3，激活没生效，先查是不是在另一个终端窗口里操作。

## 四个常见坑，按现象对照查

| 现象 | 原因 | 怎么修 |
| --- | --- | --- |
| 激活了，pip 还是装到全局 | IDE 内置终端没执行激活脚本，或在另一个窗口装的 | which pip 看路径里有没有 .venv |
| PowerShell 提示无法加载脚本 | 执行策略禁止运行 .ps1 | Set-ExecutionPolicy -Scope CurrentUser RemoteSigned |
| python 和 python3 不是同一个 | 旧系统里 python 指向 Python 2 | 统一用 python3，Windows 用 py -3 |
| 程序报 Permission denied | 之前装过一次全局包，写进了系统目录 | 建虚拟环境重装，不要用 sudo pip |

## 哪些东西要提交

| 文件或目录 | 要提交 Git 吗 | 原因 |
| --- | --- | --- |
| requirements.txt | 要 | 只有包名和版本，几行文本，别人靠它还原环境 |
| .venv/ | 不要 | 几十 MB 的平台相关二进制，换台机器就不能用 |
| pyproject.toml | 要 | 现代项目的依赖与构建配置，用来代替 requirements |
| __pycache__/ | 不要 | 编译缓存，运行时会自动重新生成 |

> **提示：pip freeze 把版本写死**
>
> `pip freeze` 输出的是**当前环境里所有包**的精确版本，包括依赖的依赖，所以清单会很长。好处是可复现：别人执行 `pip install -r requirements.txt` 装出来的和你一模一样。
>
> 只想记直接依赖，就手写一份不带版本号的清单（一行 `requests`），代价是每个人装到的版本不同。团队协作选前者，一个人写脚本两者都行。

## 让编辑器用同一个环境

激活只改终端，编辑器要单独选。VS Code 里按 `⇧⌘P`（Windows 是 `Ctrl+Shift+P`），输入 `Python: Select Interpreter`，选中路径带 `.venv` 的那一条，状态栏会显示 `3.11.6 (.venv)`。

选错的表现很典型：终端里 `import requests` 正常，编辑器里满屏黄色波浪线提示找不到模块。**那不是代码的问题，是解释器选错了。**

## 动手练一练

在一个空目录里建虚拟环境，装一个包，导出清单，再退出环境确认这个包没有进系统 Python。

1. 建目录并创建虚拟环境 —— `mkdir demo-venv && cd demo-venv && python3 -m venv .venv`
2. 激活并确认 pip 指向 .venv —— `source .venv/bin/activate && which pip`
3. 装一个包并把精确版本导出成清单 —— `pip install requests && pip freeze > requirements.txt`
4. 退出环境，确认系统 Python 里没有这个包 —— `deactivate && python3 -c "import requests"`

**完成标准**：清单里有 requests，退出环境后同一条命令报 ModuleNotFoundError，隔离就生效了。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. 为什么不该直接 pip install 到系统 Python？**

- A. 会让 pip 变慢
- B. 不同项目要的版本会互相覆盖，还可能弄坏系统工具
- C. pip 不支持这种方式
- D. 系统 Python 装不了第三方包

<details><summary>答案</summary>

**B**。系统 Python 只有一套 site-packages，两个项目要不同版本时必然打架；操作系统工具也依赖它，升级会连带弄坏。虚拟环境把这份风险关进项目目录。

</details>

**2. Windows PowerShell 里激活 .venv 应该执行哪条？**

- A. source .venv/bin/activate
- B. .\.venv\Scripts\Activate.ps1
- C. python -m activate
- D. pip install venv

<details><summary>答案</summary>

**B**。Windows 的虚拟环境把激活脚本放进 Scripts 目录，PowerShell 执行 Activate.ps1，cmd 执行 activate.bat；在 PowerShell 里敲 source 只会提示找不到这个命令。source 加 bin 目录是 macOS 和 Linux 的写法。

</details>

**3. pip freeze > requirements.txt 记录的是什么？**

- A. 只有你手动装的那几个包
- B. 当前环境里所有包及其精确版本
- C. Python 解释器版本
- D. 每个包的下载地址

<details><summary>答案</summary>

**B**。freeze 会把环境里所有已安装的包连同版本号列出来，连依赖的依赖也包含在内。这样别人按清单安装，能得到和你完全一致的组合。

</details>

</details>

---

[← Node.js 与 npm/pnpm](01-Node.js-与-npm-pnpm.md) · [目录](../Home.md) · [浏览器开发者工具 →](03-浏览器开发者工具.md)
