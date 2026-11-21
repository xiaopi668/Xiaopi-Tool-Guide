# Claude Code 上手

> 安装、会话、CLAUDE.md 与权限

`AI 编程助手` · 建议 14 分钟 · 基础

## 学完这一节，你会

- 装好并登录 Claude Code
- 用 -p、-c、-r 管理一次性与连续会话
- 写一份能被 agent 用起来的 CLAUDE.md

## 安装

Claude Code 是 Anthropic 的终端 agent。官方推荐用安装脚本，不需要预先装 Node。

## 安装与检查

```bash
$ curl -fsSL https://claude.ai/install.sh | bash
$ claude --version
Claude Code installed successfully.
2.1.118 (Claude Code)
```

> Windows 用 PowerShell：`irm https://claude.ai/install.ps1 | iex`。装完用 `claude auth login` 登录，`claude auth status` 查看状态。

## 四种启动方式

```bash
$ claude
$ claude "解释这个项目的目录结构"
$ claude -p "列出所有 TODO"
$ claude -c
$ claude -r "auth-refactor" "继续上次的重构"
```

> `-p` 是一次性提问、输出后退出，适合脚本；`-c` 继续当前目录最近一次会话；`-r` 按会话名恢复。会话里的 `/resume` 也能切换历史会话。

## CLAUDE.md：项目给它的说明书

每次开始会话，Claude Code 都会读项目根目录的 `CLAUDE.md`（也支持 `AGENTS.md`）。这里应该写**它猜不到、但每次都要用**的信息：怎么构建、怎么跑测试、目录约定、禁区。

## CLAUDE.md 示例

```text
# 项目说明

## 常用命令
- 安装依赖：pnpm install
- 开发：pnpm dev（端口 5173）
- 测试：pnpm test --run
- 类型检查：pnpm typecheck

## 约定
- 组件放 src/components，一个文件一个组件
- 不引入新的 UI 库，样式用现有 tokens
- 提交信息用 Conventional Commits

## 不要做
- 不要修改 .env、package-lock.json
- 不要动 legacy/ 目录
- 不要执行数据库迁移
```

> 写得越具体，它越少问、越少猜。命令部分尤其重要：它需要知道「怎么验证自己改对了」。

## .claude 目录里有什么

| 路径 | 作用 |
| --- | --- |
| CLAUDE.md | 项目级长期指令，每次会话自动加载 |
| .claude/settings.json | 权限规则、环境变量、模型设置 |
| .claude/skills/<名字>/SKILL.md | 可复用的技能，按需加载 |
| .claude/agents/ | 子代理定义，把专项任务拆出去 |
| .claude/hooks/ | 在工具调用前后自动执行的脚本 |

## 权限：默认每次确认

读文件通常不需要批准；**执行命令、改文件、联网**默认都会弹确认。提示里能看到它准备做什么，并可以选择「本次允许」「本次会话都允许」。请把它当成代码评审的一部分，而不是点掉的弹窗。

> **提示：上下文快满时用 /compact**
>
> 长会话会占满上下文窗口。用 `/compact` 让它把前文压缩成摘要再继续；任务切换时干脆新开一个会话，比在旧会话里翻旧账更省事。

> **注意：不要为了省事关掉全部权限**
>
> 权限确认是最后一道防线。在真实项目里全开，意味着一条错误命令就能删掉未提交的工作。要提效，应该写清楚 CLAUDE.md、缩小任务范围，而不是关掉确认。

## 第一次使用的完整流程

### 1. 进入项目目录

`cd your-project &amp;&amp; claude`。它会以当前目录为工作区。

### 2. 先让它读

「先读项目结构，说明构建和测试怎么跑，不要改文件。」

### 3. 补一份 CLAUDE.md

「根据你刚才的观察，起草一份 CLAUDE.md，包含命令、约定、禁区。」确认后提交。

### 4. 做一个小任务

从修一个 bug 或加一个小功能开始，让它先给计划，你确认后再动手。

## 权限模式与团队设置

`除了逐个批准，Claude Code 还支持按模式控制：只读、按需询问、自动接受编辑等。建议从默认模式开始，熟悉它的行为后再调整。团队可以把权限规则写进 `.claude/settings.json` 提交到仓库，让所有人的边界一致，而不是各点各的。注意：放宽的是「少问几次」，不是「不用看」。编辑与命令的结果仍然要自己验收。`

## 子代理与技能

`当任务可以拆开时，让它把独立部分交给子代理并行处理：一个读前端、一个读后端，最后汇总。重复出现的流程（发布检查、周报生成）适合写成技能，用 `/技能名` 直接调用，比每次重新描述稳定得多。技能的好处是按需加载：不用把整套流程塞进 CLAUDE.md，只在用到时展开，上下文不会被撑爆。`

## 动手练一练

在自己的项目里跑通一次「读结构 → 写 CLAUDE.md → 做小改动」的完整流程。

1. 安装并登录 —— `claude auth login`
2. 让 Claude Code 读项目并起草 CLAUDE.md
3. 提交 CLAUDE.md，再让它改一个小问题
4. 用 git diff 检查它的改动

**完成标准**：CLAUDE.md 能被提交，且下一次会话里它不再问「测试怎么跑」。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. claude -p "..." 和直接运行 claude 有什么区别？**

- A. 没有区别
- B. -p 输出一次结果后退出，适合脚本
- C. -p 只能读文件
- D. -p 会关闭权限

<details><summary>答案</summary>

**B**。-p 是非交互模式：给一个提示，打印结果就退出，适合放进脚本或管道。

</details>

**2. CLAUDE.md 最该写什么？**

- A. 完整的项目源码
- B. 构建、测试命令与项目约定
- C. 所有的历史提交
- D. 模型参数

<details><summary>答案</summary>

**B**。它每次会话都会加载，适合放「每次都要用、但 agent 猜不到」的信息：命令、目录约定、禁区。

</details>

**3. 长会话上下文快满时，最合适的做法是？**

- A. 继续堆问题
- B. 删掉项目文件
- C. 用 /compact 压缩，或新开会话
- D. 关掉权限确认

<details><summary>答案</summary>

**C**。/compact 把前文压成摘要继续；任务切换时新开会话更干净。

</details>

</details>

---

[← 什么是 AI 编程助手](01-什么是-AI-编程助手.md) · [目录](../Home.md) · [Codex CLI 上手 →](03-Codex-CLI-上手.md)
