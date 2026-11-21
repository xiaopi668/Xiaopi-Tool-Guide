# Docker 基础：把环境装进盒子

> 同一份镜像，在你电脑和服务器上跑出同样的结果

`开发环境与工具链` · 建议 16 分钟 · 进阶

## 学完这一节，你会

- 说清容器和虚拟机的区别
- 用 docker run 起一个服务并映射端口
- 写四行 Dockerfile 把项目打成镜像
- 知道挂载目录会带来什么权限问题

## 容器解决的是「我这能跑」

你写了个 Node 服务，本机能跑。同事拉下来报 `Error: Cannot find module express`；换到服务器又报 `GLIBC_2.28 not found`。原因不复杂：**代码一样，运行环境不一样**——Node 版本、系统库、环境变量、时区，任何一项不同都会让程序起不来。

容器（container）的思路是把程序加它需要的整个环境打成一个包，到哪都按包里的样子跑。这个包叫镜像（image），跑起来的实例叫容器。

## 直接在电脑上装 vs 用容器

| 直接在电脑上装 | 用容器 |
| --- | --- |
| 项目 A 要 Node 16，项目 B 要 Node 20，只能来回切版本 | 每个项目自带版本，互不干扰 |
| 同事的系统版本和你不同，报错查不出来 | 镜像里就是这个程序需要的全部东西，结果可复现 |
| 换台电脑要重装一遍所有工具，半天起步 | 一条 docker run 就能跑，换机器也一样 |
| 装错了留下的残留文件很难清干净 | 删容器等于全部清掉，不污染系统 |

> 容器不是虚拟机。它和宿主机共用同一个系统内核，所以启动是秒级、开销接近零。代价是 Windows 和 macOS 上跑的是 Linux 容器，中间隔了一层虚拟机，文件读写性能比 Linux 差一截。

## 把镜像拉下来

```bash
$ docker pull node:20-alpine
20-alpine: Pulling from library/node
Status: Downloaded newer image for node:20-alpine
docker.io/library/node:20-alpine
```

> 这条只下载，不运行，不会在你电脑上留下任何可执行程序。alpine 是精简版 Linux，镜像约 130 MB；换成 node:20 会到 1 GB 左右。看本地有哪些镜像用 docker images。

## 跑一条一次性命令

```bash
$ docker run -it --rm node:20-alpine node -v
v20.11.1
```

> -it 让容器接上你的终端，--rm 表示退出后自动删除容器。末尾的 node -v 是在容器里执行的命令，不是你本机的。这条命令不写你的磁盘。

## 不加 --rm，容器退出后会留在那

`docker run` 不加 `--rm` 时，容器退出后仍然留在磁盘上，占空间也占名字。`docker ps` 只看正在跑的，`docker ps -a` 才看得到已退出的那些。

退出的容器不会自动清。攒到几十个，磁盘会明显变少。确认不要了用 `docker rm 容器ID` 删；还在跑的要先 `docker stop`。名字重复导致的 `Conflict. The container name is already in use` 也是这么来的。

## 四条命令管住日常

| 命令 | 做什么 | 注意什么 |
| --- | --- | --- |
| docker images | 列出本地镜像和体积 | 只读，无风险 |
| docker ps -a | 列出所有容器，含已退出 | 只读，用来看残留 |
| docker rm 容器ID | 删除已退出的容器 | 正在跑的删不掉，先 docker stop |
| docker rmi 镜像名 | 删除本地镜像 | 被容器引用时删不掉，先删容器 |

> **注意：挂载目录：最方便，也最容易踩权限**
>
> `-v "$(pwd)":/app` 把当前目录映射进容器的 /app，改代码不用重新构建。它有两个必须知道的后果。
>
> - **挂载会盖住镜像里同名目录**。镜像里 /app 已经装好了依赖，你一挂载，容器看到的是你本机那份目录，缺依赖就直接报找不到模块。
> - **容器默认以 root 运行**，它在挂载目录生成的文件在宿主机上属于 root。之后你删不掉、编辑器也保存不了，只能 sudo。加 `-u "$(id -u):$(id -g)"` 让它用你的身份跑。
>
> 还有一条：macOS 和 Windows 的挂载要经过共享层，几万个文件的 node_modules 读写会明显变慢。

## 四行 Dockerfile

```bash
FROM node:20-alpine
WORKDIR /app
COPY . .
CMD ["node", "server.js"]
```

> FROM 选基础镜像，WORKDIR 设容器里的工作目录（不存在会自动建），COPY 把当前目录复制进去，CMD 是启动时默认执行的命令。在 Dockerfile 所在目录执行 docker build -t my-app:0.1 . ，最后的点表示构建上下文是当前目录，不能省。这四行能跑，但没利用缓存：任何文件一改，COPY 那一层失效，依赖得重装。正式项目会把 COPY package*.json ./ 和 RUN npm ci 提到 COPY . . 之前。

## 端口映射：让浏览器访问到容器里的服务

```bash
$ docker run -d -p 3000:3000 --name web my-app:0.1
9f2c1a7b4e8d
```

> -p 前面是宿主机端口，后面是容器里的端口，写反了浏览器就打不开。-d 表示后台运行，命令立刻返回容器 ID。之后访问 http://localhost:3000，实际连的是容器里监听 3000 的那个进程。

## docker compose：把几个容器编成一组

一个 Web 服务通常还带数据库、缓存。挨个 `docker run` 记参数很痛苦，compose 用一个 `compose.yaml` 描述这些容器：镜像、端口、挂载、环境变量、启动顺序，然后一条命令全起来。

常用就三条：`docker compose up -d` 后台启动全部，`docker compose logs -f` 跟日志，`docker compose down` 停止并删除这些容器。加 `-v` 会连数据卷一起删，**数据库里的数据会没**。

## 清理：先看清它会删什么

```bash
$ docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

Total reclaimed space: 3.412GB
```

> 默认只删悬空的东西：没 tag 的镜像、已退出的容器、没用的网络，相对安全。加 -a 会连所有没被运行中容器使用的镜像一起删，下次用要重新拉；再加 --volumes 会删数据卷，数据库数据一起没。执行前先 docker ps -a 和 docker images 过一遍。

> **提示：把不该进镜像的东西挡在门外**
>
> `COPY . .` 会把 `.git`、`node_modules`、`.env` 一起塞进镜像。`.env` 里通常有密钥，进了镜像就等于跟着镜像分发到任何地方，而且历史层里删不掉。
>
> 在 Dockerfile 同目录建 `.dockerignore`，写四行：`node_modules`、`.git`、`.env`、`dist`。语法和 .gitignore 一样，作用是构建时这些文件不参与复制。

## 动手练一练

用官方 Node 镜像在容器里跑一行命令，再把一个本地目录挂进去，看看新建文件的属主变成谁。

1. 拉一个精简版 Node 镜像并确认它在本地 —— `docker pull node:20-alpine && docker images`
2. 在容器里打印 Node 版本，退出后容器应自动删除 —— `docker run -it --rm node:20-alpine node -v`
3. 挂载当前目录并建一个文件，回到宿主机看属主 —— `docker run --rm -v "$(pwd)":/app -w /app node:20-alpine touch test.txt && ls -l test.txt`

**完成标准**：docker ps -a 里没有残留容器，test.txt 出现在你当前目录，用 ls -l 看它的属主显示为 root。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. -p 3000:8080 表示什么？**

- A. 容器 3000 映射到宿主机 8080
- B. 宿主机 3000 映射到容器 8080
- C. 容器内外都用 3000
- D. 随机分配两个端口

<details><summary>答案</summary>

**B**。冒号左边是宿主机端口，右边是容器里进程监听的端口。写反了浏览器打不开，因为容器里的服务根本没在 3000 上监听，你连不上它。

</details>

**2. 容器在挂载目录里生成的文件属主是 root，怎么避免？**

- A. 把目录 chmod 777
- B. 用 -u 传入你本机的用户和组 ID，让容器以你的身份运行
- C. 每次都用 sudo 删文件
- D. 改用 docker cp 复制文件

<details><summary>答案</summary>

**B**。容器默认以 root 身份运行，写出的文件自然属于 root。启动时用 -u 传 UID 和 GID，或者在镜像里建普通用户并用 USER 指令指定，都能避免。

</details>

**3. docker system prune -a 会做什么？**

- A. 只删已退出的容器
- B. 删除没被运行中容器使用的所有镜像，以及已退出容器和构建缓存
- C. 只清日志文件
- D. 重启 Docker 服务

<details><summary>答案</summary>

**B**。-a 会把没有被运行中容器引用的镜像全删掉，下次用要重新拉取。它不动数据卷，但加 --volumes 就会连数据库数据一起删，执行前务必确认。

</details>

</details>

---

[← 浏览器开发者工具](03-浏览器开发者工具.md) · [目录](../Home.md) · [SSH 与密钥：免密登录服务器 →](05-SSH-与密钥-免密登录服务器.md)
