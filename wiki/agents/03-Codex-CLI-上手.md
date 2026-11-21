# Codex CLI 上手

> 安装、交互与 exec、AGENTS.md、沙箱

`AI 编程助手` · 建议 14 分钟 · 基础

## 学完这一节，你会

- 装好 Codex CLI 并完成登录
- 分清交互模式与 codex exec 的场景
- 知道沙箱和审批在保护什么

## 安装

Codex CLI 是 OpenAI 的本地 agent。三种安装方式任选：官方脚本、npm、Homebrew。

## 三种安装方式

```bash
$ curl -fsSL https://chatgpt.com/codex/install.sh | sh
$ npm install -g @openai/codex
$ brew install --cask codex
$ codex --version
```

> Windows 用 `powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"`。装完在项目目录运行 `codex`，首次会让你选择登录方式。

## 两种工作模式

**交互模式**：直接跑 `codex`，像聊天一样，它会持续读文件、改代码、执行命令。**非交互模式**：`codex exec "..."`，给一个任务、跑完退出，适合脚本和 CI。要接着上次的会话，用 `codex resume`；`--last` 取当前目录最近一次。选错模式的代价很直接：交互模式塞进脚本会卡在等待输入，exec 拿来做探索又缺少中途纠偏。

## 常用入口

```bash
$ codex
$ codex exec "把 src/utils.ts 里的日期格式化抽成函数，并补测试"
$ codex resume --last
$ codex -C ~/projects/demo
```

> `-C` 指定工作目录，避免在错误的目录里开跑。任务描述里带上**文件、期望行为、验收方式**，产出会稳定很多。

## AGENTS.md：仓库级说明

Codex 会读取仓库里的 `AGENTS.md`，规则和 Claude Code 的 `CLAUDE.md` 一样：写命令、结构、约定、禁区。多个 agent 共用一份约定时，可以只维护 AGENTS.md。

## AGENTS.md 示例

```text
# AGENTS.md

## 环境
- Node 22，包管理器用 pnpm
- 本地服务：pnpm dev（3000 端口）

## 验证
- 改完必须跑：pnpm test --run
- 类型检查：pnpm typecheck

## 约定
- 新增依赖前先说明理由
- 不要提交 dist/ 与 .env
- API 变更要同步更新 docs/api.md
```

> 把「怎么验证」写在最前面。agent 知道自己会被检查，才会认真跑测试。

## 沙箱与审批

Codex 在不同系统上使用不同的沙箱：macOS 用 Seatbelt，Linux 用 bubblewrap，Windows 有原生沙箱。命令默认在受限环境里执行，越权操作会先问你。`codex sandbox` 可以用同样的策略单独跑一条命令，用来验证权限边界。如果不确定某个操作为什么被拦，先用 `codex sandbox` 复现一次：同样的策略、同样的报错，排查起来比在会话里猜快得多。先跑小任务，再放开复杂操作；先读后改，先验证再合并。

## 交互与 exec 怎么选

| 场景 | 用哪个 | 原因 |
| --- | --- | --- |
| 探索、重构、多轮调整 | codex | 需要你中途确认与纠偏 |
| 一次性的明确任务 | codex exec | 跑完即走，可放进脚本 |
| CI 里做检查 | codex exec | 非交互，输出可收集 |
| 继续昨天的会话 | codex resume --last | 保留上下文 |

> **注意：不要默认全自动**
>
> 把审批和沙箱全部关掉，等于把终端交给一个会自信犯错的模型。先用小任务观察它的行为，再逐步放宽；真实项目目录里保持默认设置。

## 第一次使用

### 1. 装好并登录

运行 `codex`，选择 Sign in with ChatGPT 或配置 API key。

### 2. 在项目里先读

「先读项目，说明入口和测试怎么跑，不要改文件。」

### 3. 补 AGENTS.md

让它根据观察起草 AGENTS.md，你确认后提交。

### 4. 跑一个 exec 任务

用 `codex exec` 做一个小改动，观察它的输出和 diff。

## 配置与 profile

`Codex 的配置放在 `$CODEX_HOME`（通常是 `~/.codex`）。你可以用 profile 给不同项目准备不同的默认值：一个偏保守、只读；一个允许改代码。`--profile` 选择配置层，`-C` 指定工作目录，两者配合能避免「在错误目录、用错误策略」开跑。配置文件里可能出现密钥，记得把它排除在版本控制之外。需要提交给团队共享的，只有项目级的 `AGENTS.md`。`

## 输出怎么看

`exec 模式适合把输出接进脚本：它会把过程与结果写到标准输出。交互模式则会在每步之间停下来等你确认。无论哪种，最后都要回到 `git diff`：agent 的总结是线索，diff 才是事实。在 CI 里用 exec 时，要把退出码当作判据。任务失败就该让流水线红掉，而不是看它打印了一句「已完成」就放行。`

## 动手练一练

装好 Codex CLI，在测试仓库里各跑一次交互模式和 exec。

1. 安装并登录 —— `codex`
2. 交互模式下让它解释项目，不修改文件
3. 用 exec 跑一个小任务 —— `codex exec "..."`
4. 用 git diff 检查它的改动

**完成标准**：能说清两种模式的差别，并看懂它改了什么。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. codex exec 最适合什么场景？**

- A. 多轮探索式重构
- B. 一次性的明确任务或 CI
- C. 调试复杂交互
- D. 需要频繁确认的操作

<details><summary>答案</summary>

**B**。exec 是非交互模式：给任务、跑完退出，适合脚本与流水线；探索式任务用交互模式更合适。

</details>

**2. AGENTS.md 的作用是？**

- A. 存放源码
- B. 给 agent 的仓库级说明：命令、约定、禁区
- C. 替代 package.json
- D. 保存聊天记录

<details><summary>答案</summary>

**B**。它每次会话都会被读取，用来交代 agent 猜不到但必须遵守的信息。

</details>

**3. 关于沙箱与审批，正确做法是？**

- A. 一律关掉以提速
- B. 先用小任务观察，再按需放宽
- C. 只在 CI 里开启
- D. 审批会降低代码质量

<details><summary>答案</summary>

**B**。默认最小权限，观察行为后再放宽；全关等于把终端交给会犯错的模型。

</details>

</details>

---

[← Claude Code 上手](02-Claude-Code-上手.md) · [目录](../Home.md) · [dsh：DeepSeek Harness →](04-dsh-DeepSeek-Harness.md)
