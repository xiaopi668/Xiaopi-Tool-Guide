# Node.js 与 npm/pnpm

> 把依赖装进项目文件夹，而不是装进整台电脑

`开发环境与工具链` · 建议 15 分钟 · 基础

## 学完这一节，你会

- 看懂 node -v 和 npm -v 各自在报什么
- 用 nvm 在多个 Node 版本之间来回切
- 分清 npm install 和 npm i -D
- 知道哪些文件该提交、哪些绝对不能

## 先把这三个东西分开

一句话：**Node.js 是让 JavaScript 跑在浏览器外面的运行时（runtime）**；npm 是随 Node 一起装上的包管理器（package manager），版本号也是一起来的；pnpm 是另一个包管理器，读同一份 package.json，但装依赖的方式不一样。

你敲的 `node -v` 看运行时版本，`npm -v` 看包管理器版本。两个数字对不上是正常的，它们本来就不是同一个东西。

版本号是三段式：`20.11.1` 里的 20 是大版本，11 是小版本，1 是修订号。大版本之间会不兼容，小版本只加功能，修订号只修 bug。

## 先确认装没装

```bash
$ node -v && npm -v
v20.11.1
10.2.4
```

> 在任意目录执行，只读不写。如果第一行就报 zsh: command not found: node，说明 Node 没装，或者装了但没进 PATH。

## nvm：装版本、切版本

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 重开终端后
nvm install --lts        # 装当前 LTS，例如 20.x
nvm install 18           # 装指定大版本的最新小版本
nvm use 18               # 只对当前终端生效，关掉就回默认
nvm alias default 20     # 把默认版本设成 20
nvm ls                   # 看装了哪些、当前在用哪个
```

> 为什么需要它：老项目锁在 Node 16，新项目要 20，直接装新版覆盖旧的，老项目当场报 SyntaxError: Unexpected token ? ——问号语法在 16 上还不存在。nvm 把每个版本装进独立目录，用一个软链接决定当前用哪个，切版本不重新下载。安装脚本只写 ~/.nvm 和几行 shell 配置，不动系统目录。项目根目录放一个 .nvmrc 写一行 20，进目录后敲 nvm use 会读它。Windows 用 nvm-windows，命令要写完整版本号：nvm install 20.11.1、nvm use 20.11.1。

## package.json：项目的身份证

```json
{
  "name": "my-app",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint ."
  },
  "dependencies": {
    "react": "^18.2.0"
  },
  "devDependencies": {
    "vite": "^5.2.0",
    "eslint": "^8.57.0"
  },
  "engines": {
    "node": ">=20"
  }
}
```

> 这个文件要提交到 Git。^18.2.0 表示 18.2.0 及以上但不跨到 19，真正锁死的精确版本在 package-lock.json 里。JSON 不允许尾逗号，多一个就报 npm ERR! code EJSONPARSE。

## 四个字段，各管一件事

| 字段 | 写什么 | 生产环境还需要吗 |
| --- | --- | --- |
| dependencies | 代码跑起来必须引入的包，如 react、express | 需要，会一起打包上线 |
| devDependencies | 只在开发、构建、测试时用，如 vite、eslint、typescript | 不需要 |
| scripts | 命令别名，用 npm run 名字 执行 | 需要，部署命令常写在这 |
| engines | 声明 Node 版本范围，如 >=20 | 需要，版本不对会警告 |

## npm install 和 npm i -D 的区别

`npm install` 不带包名时，把 dependencies 和 devDependencies 都装上；带包名时**默认写进 dependencies**。

`npm i -D 包名` 是 `npm install --save-dev 包名` 的缩写，写进 devDependencies。判断标准只有一句：**这东西打包上线之后还要不要运行？要，就 dependencies；只在你自己电脑上干活，就 devDependencies。**

装错了不用删文件重来：再执行一次 `npm i -D 包名`，npm 会把它从 dependencies 搬到 devDependencies，lock 文件一起更新。

> **注意：node_modules 绝对不要提交**
>
> 它动辄几万个文件、几百 MB，里面还有针对你操作系统编译的二进制文件。提交上去，队友在 Windows 上拉下来就是一堆崩溃，Git 本身也会慢到几乎不可用。
>
> 项目根目录建 `.gitignore`，写三行：`node_modules/`、`dist/`、`.env`。已经误提交过的，先 `git rm -r --cached node_modules`（只从 Git 索引里移除，磁盘上的文件不动），再提交一次。
>
> **该提交的是 package.json 和 package-lock.json**：前者说我要什么，后者说我最后装到了什么。

## package-lock.json 到底锁了什么

package.json 写的是范围（`^5.2.0`），意味着今天装到 5.2.1，下个月可能装到 5.3.0。lock 文件记录**每个包的确切版本、下载地址和校验值**，连依赖的依赖也记。

它让「我这能跑」变成「谁都能跑出同一套依赖」。CI 里用 `npm ci`：只认 lock 文件，装之前先删掉 node_modules，版本对不上直接报 `npm ERR! can only install packages when your package.json and package-lock.json are in sync`，比 npm install 严格，也更快。

lock 文件冲突时不要手改。回到主干执行 `npm install` 重新生成，再把你的改动合并进去。

## npm run：把长命令收进 scripts

```bash
npm run dev        # 执行 scripts.dev，也就是 vite
npm run build      # 执行 scripts.build
npm run            # 不带名字，列出所有可用脚本
npm run dev -- --port 5174   # -- 后面的参数原样传给脚本
```

> 报 Missing script: dev 就是 scripts 里没有这个名字，先 npm run 看一眼真实名字。只有 start 和 test 可以省略 run：npm start、npm test。

## npx：用完就走的临时执行

```bash
$ npx serve .
Need to install the following packages:
  serve@14.2.1
Ok to proceed? (y)
```

> npx 把包下到缓存里执行，不写进 package.json，也不装进 node_modules。适合跑一次性的工具：npx create-vite@latest my-app、npx prettier --write .。带 @版本号可以避免每次都拉最新版。

## pnpm 为什么快

npm 给每个项目都往自己的 node_modules 里复制一份完整的包。十个项目用同一个 React，磁盘上就有十份。

pnpm 把所有包按内容存在一个全局仓库（store）里，项目的 node_modules 中放的是**硬链接（hard link）**——同一个文件在磁盘上只有一份，多个路径指向它。所以装第二个项目几乎不下载，磁盘占用小得多，切分支重装也就几秒。

用法和 npm 基本一样：`pnpm install`、`pnpm add lodash`、`pnpm add -D vite`、`pnpm run dev`。注意**一个项目只用一种**，npm 和 pnpm 混用会生成两套 lock 文件，node_modules 结构也会互相打架。

## 国内镜像源：装不动时先换源

```bash
$ npm config set registry https://registry.npmmirror.com
```

> 这条写进用户级 ~/.npmrc，对所有项目生效。查当前源用 npm config get registry，恢复官方源就设回 https://registry.npmjs.org。pnpm 用 pnpm config set registry https://registry.npmmirror.com。只想对单个项目生效，就在项目根目录建 .npmrc 写一行 registry=https://registry.npmmirror.com。

## 动手练一练

在空文件夹里从零建一个 npm 项目，装一个开发依赖和一个运行时依赖，再用脚本把它跑起来。

1. 新建文件夹并生成 package.json —— `mkdir demo-node && cd demo-node && npm init -y`
2. 装一个构建工具，打开 package.json 确认它落在 devDependencies —— `npm i -D vite`
3. 装一个运行时依赖，同样确认它落在 dependencies —— `npm i dayjs`
4. 加一条脚本并执行，确认能打印 Node 版本 —— `npm pkg set scripts.hello="node -v" && npm run hello`

**完成标准**：package.json 里两个依赖各在其位，npm run hello 能打印出 v20 这样的版本号，说明脚本生效了。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. 你在项目里执行 npm i -D typescript，它会被写进哪里？**

- A. dependencies
- B. devDependencies
- C. 只写 package-lock.json，不写 package.json
- D. 全局 node_modules

<details><summary>答案</summary>

**B**。参数 -D 是 --save-dev 的缩写，装完写进 devDependencies 而不是 dependencies。它的含义是这个包只在开发和构建时用，别人执行 npm ci --omit=dev 部署生产环境时不会装它。

</details>

**2. package-lock.json 应该提交到 Git 吗？**

- A. 不该，它体积太大
- B. 该，它锁定每个依赖的精确版本，保证所有人装出同一套
- C. 只在用 pnpm 时才提交
- D. 该，但冲突时可以手动改完再提交

<details><summary>答案</summary>

**B**。它记录了每个包的确切版本和校验值，提交后别人和 CI 才能装出完全相同的依赖。冲突时重新执行 npm install 生成，不要手改。

</details>

**3. 终端报 zsh: command not found: node，最可能的原因是什么？**

- A. 项目里没装依赖
- B. Node 没装，或者装了但没进 PATH
- C. package.json 写错了
- D. 镜像源挂了

<details><summary>答案</summary>

**B**。这个报错来自 shell，意思是它在 PATH 里找不到 node 这个可执行文件，跟项目内容无关。用 nvm 装完记得重开终端或 source 一下配置文件。

</details>

</details>

---

[← 远程开发：在本机编辑服务器上的代码](../vscode/06-远程开发-在本机编辑服务器上的代码.md) · [目录](../Home.md) · [Python 与虚拟环境 →](02-Python-与虚拟环境.md)
