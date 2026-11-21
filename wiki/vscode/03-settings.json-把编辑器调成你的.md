# settings.json：把编辑器调成你的

> 先把默认值改对一次，以后每天少点十次鼠标

`VS Code` · 建议 12 分钟 · 基础

## 学完这一节，你会

- 分清用户设置和工作区设置
- 能打开并手写 settings.json
- 抄一份适合自己的配置文件
- 知道写错时去哪看报错

## 两层设置，别混着改

VS Code 的设置分两层。用户设置（User Settings）存在你自己机器上，对你打开的所有项目生效，放个人偏好；工作区设置（Workspace Settings）存在项目里的 `.vscode/settings.json`，只对这个项目生效，跟着仓库走。

优先级很直接：同一个键，工作区设置赢用户设置。所以团队约定写进工作区，个人偏好留在用户设置里，两边都不要越界。

## 两个文件分别在哪

| 层级 | 文件位置 | 影响谁 | 要不要进 Git |
| --- | --- | --- | --- |
| 用户设置 | macOS 在 ~/Library/Application Support/Code/User/settings.json；Windows 在 %APPDATA%\Code\User\settings.json | 你打开的所有项目 | 不要，跟机器绑在一起 |
| 工作区设置 | 项目根目录下的 .vscode/settings.json | 打开这个项目的所有人 | 要，前提是写的是团队约定 |

## 打开 JSON 的三种办法

### 1. 走命令面板

⌘⇧P（Ctrl+Shift+P）打开命令面板，输入 `Preferences: Open User Settings (JSON)` 或 `Preferences: Open Workspace Settings (JSON)`，回车。这条路径最不容易点错。

### 2. 走设置界面右上角

⌘,（Ctrl+,）打开设置界面，标题栏右上角有一个像翻开的文件的图标，点它就是整份 JSON。旁边的图标打开的是当前页面的片段，点错会以为设置丢了。

### 3. 直接当普通文件打开

工作区设置就是一个普通文件。在资源管理器里点 `.vscode/settings.json` 就能编辑；文件不存在时，新建并保存到这个路径即可。

## 一份可以直接用的 settings.json

```json
{
  // 编辑体验
  "editor.fontSize": 14,                // 只影响编辑器里的代码字号，不动侧边栏
  "editor.tabSize": 2,                  // 一个缩进显示成 2 个空格（Go、Java 常用 4）
  "editor.insertSpaces": true,          // 按 Tab 键插入空格而不是制表符
  "editor.rulers": [80, 120],           // 在第 80、120 列画竖线，提醒一行别太长
  "editor.wordWrap": "on",              // 长行自动折行，写 Markdown 时很需要
  "editor.minimap.enabled": false,      // 关掉右侧缩略图，小屏幕能多出两行

  // 保存与格式
  "editor.formatOnSave": true,          // 保存时自动格式化，需要先装格式化插件
  "editor.formatOnPaste": false,        // 粘贴时不自动格式化，很容易打乱原有缩进
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "files.eol": "\n",                    // 行尾统一成 LF，避免 CRLF 混进仓库

  // 文件
  "files.autoSave": "onFocusChange",    // 切走窗口时保存；也有 off、afterDelay、onWindowChange
  "files.autoSaveDelay": 1000,          // autoSave 选 afterDelay 时的延迟毫秒数

  // 文件树里藏掉噪音，文件本身不会删
  "files.exclude": {
    "**/.git": true,
    "**/node_modules": true,
    "**/.DS_Store": true
  }
}
```

> settings.json 是 JSON with Comments（JSONC），可以写 // 注释，编辑器自己认。但把这份文件喂给标准 JSON 解析器（比如某些构建脚本）会报错，所以它只适合放 VS Code 的配置目录。

## 最常用的七项

| 设置键 | 作用 | 建议值 |
| --- | --- | --- |
| editor.fontSize | 编辑器字号，不影响侧边栏 | 13 到 15，看你的屏幕 |
| editor.tabSize | 一个缩进显示成几个空格 | 前端 2，Go 和 Java 4 |
| editor.formatOnSave | 保存时自动格式化 | 打开，但要先装格式化插件 |
| files.exclude | 在文件树里藏掉匹配的路径 | node_modules、.git、dist |
| editor.rulers | 在第 N 列画一条竖线 | [80, 120] 两档 |
| files.autoSave | 自动保存的时机 | onFocusChange，切走窗口时存 |
| editor.defaultFormatter | 默认用哪个格式化器 | 团队统一的那一个 |

> **注意：写错一个逗号，整份设置不生效**
>
> 少一个逗号、多一个括号，保存时会弹出 `Unable to write into user settings. Please open the user settings to correct errors/warnings in it and try again.`。这时界面上的设置会退回默认值，看起来像「改了没用」。
>
> 按提示打开文件，⌘⇧M（Ctrl+Shift+M）打开问题面板，能直接看到第几行第几列。另一种错更安静：键名拼错不会报红，设置界面里显示 `Unknown Configuration Setting`，或者干脆被忽略，你只会觉得这个设置项不存在。

> **提示：设置同步：换机器不用重配**
>
> 左下角齿轮 → Turn on Settings Sync，或者命令面板搜 `Settings Sync: Turn On`，用 GitHub 或 Microsoft 账号登录。同步项包括设置、键盘快捷方式、用户代码片段、扩展、UI 状态和配置档案（Profiles），在 `Settings Sync: Configure` 里逐项勾选。
>
> 注意扩展也在同步范围里：公司机器和个人机器共用一个账号时，你个人的插件会被带过去，启动变慢。这一点在别人的机器上尤其明显。

## 哪些设置该进仓库

| 塞进工作区的个人偏好 | 真正属于团队约定 |
| --- | --- |
| editor.fontSize: 18 | editor.tabSize 与 insertSpaces |
| workbench.colorTheme: 某个深色主题 | files.eol 统一成 LF |
| files.autoSave: off | editor.formatOnSave: true |
| 你自己的字体和缩放比例 | files.exclude 藏掉 dist、node_modules |

> 一条判断标准：这条设置影响**代码长什么样**，进工作区；只影响**你自己看得舒不舒服**，留用户设置。混着写，同事每次拉代码都要改回来。

## 设置界面里的搜索前缀

设置界面顶部的搜索框支持前缀：`@modified` 只看你改过的项，`@ext:esbenp.prettier-vscode` 只看某个扩展贡献的设置，`@lang:javascript` 只看这门语言的设置。找一项记不住名字的设置，从这里搜比翻文档快得多。

搜到之后点齿轮可以直接选「复制设置 ID」，粘到 JSON 里再填值，键名就不用背了。

## 确认用户设置文件真的存在

```bash
$ ls -l "$HOME/Library/Application Support/Code/User/settings.json"
-rw-r--r--  1 you  staff  812 11月 20 10:02 /Users/you/Library/Application Support/Code/User/settings.json
```

> macOS 的默认路径。Linux 在 ~/.config/Code/User/settings.json，Windows 在 PowerShell 里看 $env:APPDATA\Code\User\settings.json。文件不存在，说明你还没保存过任何用户设置，在设置界面改一项就会生成。

## 动手练一练

把上面那份设置抄进你的用户设置里，再为一个真实项目加一份工作区设置。

1. 按 ⌘, 打开设置界面，搜 format on save 并勾上
2. 点标题栏右上角图标切到 JSON，对照抄一遍
3. 在项目根目录建 .vscode/settings.json，只写 tabSize 和 files.exclude
4. 把 .vscode 一起提交，让同事拉到同样的设置

**完成标准**：打开任意文件按 ⌘S，代码自动排好，说明工作区设置已经生效了。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. 同一个设置在用户设置和工作区设置里取值不同，哪个生效？**

- A. 用户设置
- B. 工作区设置
- C. 随机取一个
- D. 弹窗让你当场选

<details><summary>答案</summary>

**B**。工作区设置优先级更高，它压在你自己的用户设置上面。这也是团队约定放 .vscode/settings.json 的原因，谁拉下来都一致。

</details>

**2. 工作区设置文件应该放在哪里？**

- A. 项目根目录的 settings.json
- B. 项目根目录的 .vscode/settings.json
- C. 用户目录下的 .vscode/settings.json
- D. VS Code 安装目录里

<details><summary>答案</summary>

**B**。VS Code 只认项目根下 .vscode 目录里的 settings.json。放错位置不会报错，只是完全不起作用，这是最常见的「设置了没用」原因。

</details>

**3. 改完 settings.json 保存，设置却没变化，先查什么？**

- A. 重装 VS Code
- B. 文件里有没有语法错误，看问题面板
- C. 换一个主题
- D. 把项目目录删掉重开

<details><summary>答案</summary>

**B**。有语法错误时整份文件会被忽略，编辑器退回默认值。保存弹出提示就按提示打开文件，问题面板会直接指出第几行第几列。

</details>

</details>

---

[← 编辑快捷键：多光标、行操作、快速跳转](02-编辑快捷键-多光标-行操作-快速跳转.md) · [目录](../Home.md) · [插件：少而精 →](04-插件-少而精.md)
