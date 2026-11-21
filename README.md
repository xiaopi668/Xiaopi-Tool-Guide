<div align="center">
  <img src="pictures/banner.svg" alt="Xiaopi 工具指南" width="100%" />
  <br><br>

  <img src="https://img.shields.io/badge/%E7%AB%A0%E8%8A%82-39-000?style=flat-square" alt="39 节">
  <img src="https://img.shields.io/badge/%E5%91%BD%E4%BB%A4-136-000?style=flat-square" alt="136 条命令">
  <img src="https://img.shields.io/badge/%E5%BF%AB%E6%8D%B7%E9%94%AE-58-000?style=flat-square" alt="58 个快捷键">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-000?style=flat-square" alt="MIT License"></a>

  <br><br>
  <strong>从终端到 Git，从编辑器到开发环境：把每天要用的计算机工具讲明白。</strong>
</div>

# 项目简介

这是一份写给普通人的计算机工具指南。

网上关于这些工具的资料并不少，但大多有两个问题：要么假设你已经会了，直接从「执行 xxx 命令」开始；要么把工具当成考试科目，讲一堆概念却不告诉你明天上班该怎么用。

这份指南想做的事情很朴素：**把每个工具讲到你能立刻用起来**。每一节都从真实场景出发，给出可以直接复制的命令、真实会遇到的报错，以及出错之后怎么查。全篇 39 节，每节 10 到 18 分钟，可以按顺序读，也可以当成手册随时翻。

内容覆盖七条主线：

- **终端与系统**：命令是怎么回事、文件和目录、查找、管道、权限、进程与端口
- **Git**：心智模型、第一次配置、日常三件套、分支、远程协作、冲突、撤销、提交规范
- **VS Code**：界面与命令面板、编辑快捷键、settings.json、插件、断点调试、远程开发
- **开发环境**：Node 与 npm、Python 虚拟环境、浏览器开发者工具、Docker、SSH
- **效率与排错**：别名与历史、9 类常见报错的解法、安全习惯、学习路线
- **AI 编程助手**：Claude Code、Codex CLI、dsh，以及让 agent 干活的流程与安全边界
- **AI 时代**：工具地图、提示词、核查方法、本地模型与 API 成本

# 这份指南适合谁

- 刚接触命令行，看到黑色窗口就发怵的人
- 会一点 Git，但每次出问题都靠搜索和复制粘贴的人
- 每天用 VS Code，却一直在用鼠标点菜单的人
- 想把项目在本机跑起来，却被 Node、Python、环境变量绕晕的人
- 想给自己或团队留一份「新人第一天就能看」的材料的人

不需要任何前置知识。所有命令都标注了 macOS 和 Windows 的差异；快捷键一律分两列给出。

# 怎么用

**路线一：从零开始，按顺序读。** 39 节大概需要 8 到 10 小时，建议每天 3 节。从 [第 1 节](wiki/terminal/01-终端-Shell-与一条命令的解剖.md) 开始，遇到不懂的术语先记下来，读到后面自然会懂。

**路线二：带着问题查。** 直接跳到 [命令速查](wiki/appendix/命令速查.md) 或 [报错排查手册](wiki/efficiency/02-报错排查手册.md)，找到对应的条目。每节末尾都有「动手练一练」，建议真的在自己的电脑上跑一遍。

**路线三：当团队手册。** 新人入职时，让他读「终端 + Git + 开发环境」这三块，再配一份自己项目的 README。

> **一个建议**：不要等「学完」再动手。每读完一节，立刻在终端里把这一节的命令跑一遍，哪怕只是 `pwd` 和 `ls`。肌肉记忆比笔记可靠。

# 目录

| # | 章节 | 时长 | 难度 |
| --- | --- | --- | --- |
| 1 | [终端、Shell 与一条命令的解剖](wiki/terminal/01-终端-Shell-与一条命令的解剖.md) | 10 分钟 | 入门 |
| 2 | [文件与目录的十个命令](wiki/terminal/02-文件与目录的十个命令.md) | 12 分钟 | 入门 |
| 3 | [看文件与在文件里找东西](wiki/terminal/03-看文件与在文件里找东西.md) | 14 分钟 | 基础 |
| 4 | [管道与重定向：把命令拼成流水线](wiki/terminal/04-管道与重定向-把命令拼成流水线.md) | 14 分钟 | 基础 |
| 5 | [权限与环境变量](wiki/terminal/05-权限与环境变量.md) | 15 分钟 | 基础 |
| 6 | [进程、端口与网络](wiki/terminal/06-进程-端口与网络.md) | 15 分钟 | 进阶 |
| 7 | [Git 到底在管什么](wiki/git/01-Git-到底在管什么.md) | 12 分钟 | 入门 |
| 8 | [安装与第一次配置](wiki/git/02-安装与第一次配置.md) | 12 分钟 | 入门 |
| 9 | [日常三件套：status / add / commit / log](wiki/git/03-日常三件套-status-add-commit-log.md) | 15 分钟 | 基础 |
| 10 | [分支：并行工作的安全区](wiki/git/04-分支-并行工作的安全区.md) | 14 分钟 | 基础 |
| 11 | [远程仓库与协作](wiki/git/05-远程仓库与协作.md) | 15 分钟 | 基础 |
| 12 | [冲突：为什么发生、怎么解决](wiki/git/06-冲突-为什么发生-怎么解决.md) | 15 分钟 | 进阶 |
| 13 | [后悔药：撤销与回滚](wiki/git/07-后悔药-撤销与回滚.md) | 18 分钟 | 进阶 |
| 14 | [提交规范与团队工作流](wiki/git/08-提交规范与团队工作流.md) | 15 分钟 | 进阶 |
| 15 | [先认识这个编辑器](wiki/vscode/01-先认识这个编辑器.md) | 10 分钟 | 入门 |
| 16 | [编辑快捷键：多光标、行操作、快速跳转](wiki/vscode/02-编辑快捷键-多光标-行操作-快速跳转.md) | 14 分钟 | 基础 |
| 17 | [settings.json：把编辑器调成你的](wiki/vscode/03-settings.json-把编辑器调成你的.md) | 12 分钟 | 基础 |
| 18 | [插件：少而精](wiki/vscode/04-插件-少而精.md) | 14 分钟 | 基础 |
| 19 | [断点调试：比 console.log 快十倍](wiki/vscode/05-断点调试-比-console.log-快十倍.md) | 16 分钟 | 进阶 |
| 20 | [远程开发：在本机编辑服务器上的代码](wiki/vscode/06-远程开发-在本机编辑服务器上的代码.md) | 15 分钟 | 进阶 |
| 21 | [Node.js 与 npm/pnpm](wiki/dev/01-Node.js-与-npm-pnpm.md) | 15 分钟 | 基础 |
| 22 | [Python 与虚拟环境](wiki/dev/02-Python-与虚拟环境.md) | 14 分钟 | 基础 |
| 23 | [浏览器开发者工具](wiki/dev/03-浏览器开发者工具.md) | 15 分钟 | 基础 |
| 24 | [Docker 基础：把环境装进盒子](wiki/dev/04-Docker-基础-把环境装进盒子.md) | 16 分钟 | 进阶 |
| 25 | [SSH 与密钥：免密登录服务器](wiki/dev/05-SSH-与密钥-免密登录服务器.md) | 14 分钟 | 进阶 |
| 26 | [终端效率：别名、历史与补全](wiki/efficiency/01-终端效率-别名-历史与补全.md) | 12 分钟 | 基础 |
| 27 | [报错排查手册](wiki/efficiency/02-报错排查手册.md) | 16 分钟 | 基础 |
| 28 | [安全习惯：别把钥匙贴在门上](wiki/efficiency/03-安全习惯-别把钥匙贴在门上.md) | 12 分钟 | 进阶 |
| 29 | [学习路线图与下一步](wiki/efficiency/04-学习路线图与下一步.md) | 10 分钟 | 进阶 |
| 30 | [什么是 AI 编程助手](wiki/agents/01-什么是-AI-编程助手.md) | 10 分钟 | 入门 |
| 31 | [Claude Code 上手](wiki/agents/02-Claude-Code-上手.md) | 14 分钟 | 基础 |
| 32 | [Codex CLI 上手](wiki/agents/03-Codex-CLI-上手.md) | 14 分钟 | 基础 |
| 33 | [dsh：DeepSeek Harness](wiki/agents/04-dsh-DeepSeek-Harness.md) | 12 分钟 | 基础 |
| 34 | [让 agent 真正干活的流程](wiki/agents/05-让-agent-真正干活的流程.md) | 14 分钟 | 进阶 |
| 35 | [权限、沙箱与安全](wiki/agents/06-权限-沙箱与安全.md) | 12 分钟 | 进阶 |
| 36 | [AI 工具地图](wiki/ai/01-AI-工具地图.md) | 10 分钟 | 入门 |
| 37 | [提示词：把需求说清楚](wiki/ai/02-提示词-把需求说清楚.md) | 12 分钟 | 基础 |
| 38 | [用 AI 学习与查证](wiki/ai/03-用-AI-学习与查证.md) | 12 分钟 | 进阶 |
| 39 | [本地模型与 API](wiki/ai/04-本地模型与-API.md) | 12 分钟 | 进阶 |

## 附录

| 内容 | 说明 |
| --- | --- |
| [命令速查](wiki/appendix/命令速查.md) | 136 条常用命令，按工具分组，含 Windows 对应写法与「仅 Linux / 仅 macOS」标注 |
| [快捷键速查](wiki/appendix/快捷键.md) | 58 个快捷键，macOS / Windows / Linux 对照 |
| [术语表](wiki/appendix/术语表.md) | 28 个术语的大白话解释 |
| [开始之前](wiki/开始之前.md) | 环境准备、三个习惯、怎么读报错、怎么提问 |

# 网页版

同一套内容还有一个可以交互的网页版：[Xiaopi-Tool-Guide-Web](https://github.com/xiaopi668/Xiaopi-Tool-Guide-Web)。带命令筛选、快捷键随机练习、⌘K 搜索和学习进度，适合边做边查。

<a href="https://github.com/xiaopi668/Xiaopi-Tool-Guide-Web"><img src="pictures/web-home.jpg" alt="网页版首页" width="100%" /></a>

# 本地阅读

```bash
git clone https://github.com/xiaopi668/Xiaopi-Tool-Guide.git
cd Xiaopi-Tool-Guide
```

用任何 Markdown 编辑器打开 `README.md`，或者用 VS Code 的 Markdown 预览：

```bash
code README.md
```

# 约定

- 命令一律用 ```bash``` 代码块，行首的 `$` 表示提示符，不要一起复制
- 快捷键分 macOS 和 Windows 两列，符号 ⌘ ⌥ ⇧ ⌃ 仅用于 macOS
- 标着「危险」的命令会写明后果，练习请在空目录或测试仓库里做
- 中文术语首次出现时附英文，例如 暂存区（staging area）

# 反馈

内容有错误、或者你想补充一节，欢迎提 Issue 或 PR。这个仓库会持续更新，命令和快捷键会随工具版本校正。

# 许可

[MIT](LICENSE)。转载、修改、用于内部培训都可以，保留版权声明即可。
