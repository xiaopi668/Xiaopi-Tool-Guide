# SSH 与密钥：免密登录服务器

> 生成一对密钥，让服务器认你的机器，不再每次输密码

`开发环境与工具链` · 建议 14 分钟 · 进阶

## 学完这一节，你会

- 说清公钥和私钥各自能做什么
- 生成 ed25519 密钥并装到服务器上
- 用 config 给常用服务器起别名
- 按报错文本定位连不上的原因

## SSH 在做两件事

SSH（Secure Shell）是一条加密通道。你敲的每个字符、服务器返回的每个字节都在通道里加密传输，中间的路由器只能看到有一串流量，看不到内容。

连上之前要做两次认证：**你确认服务器是真的**（否则可能连到一台冒充它的机器），**服务器确认你是你**。前者靠指纹，后者靠密码或密钥。密钥登录就是后者的免密方案。

## 第一次连一台新机器

```bash
$ ssh root@203.0.113.10
The authenticity of host '203.0.113.10 (203.0.113.10)' can't be established.
ED25519 key fingerprint is SHA256:q5Kd8s2fH1mZ7cVbQ0nLp4xYaJ6tR9wUeS3gP8Zp8.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

> 它在问你信不信这台机器。输 yes 后指纹写进 known_hosts，以后不再问。看不到内容就别急着输 yes：用别的渠道核对一下指纹，这一步能挡住一次冒充。

## 公钥和私钥：一个是锁，一个是钥匙

`ssh-keygen` 一次生成两个文件：`id_ed25519` 是私钥，`id_ed25519.pub` 是公钥。公钥可以随便发——它只是锁，贴到 GitHub、贴到服务器都无所谓；私钥是钥匙，**永远不要发给任何人，不要粘进网页，不要提交进仓库**。

认证过程是：服务器拿你的公钥出一道只有对应私钥能解的题，你本地的私钥解开了，就证明你是你。私钥全程不出你的机器——这也是它比密码安全的地方，服务器那边根本不存在能直接偷走的东西。

## 生成一对 ed25519 密钥

```bash
$ ssh-keygen -t ed25519 -C "you@example.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/you/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Your identification has been saved in /Users/you/.ssh/id_ed25519
Your public key has been saved in /Users/you/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:q5Kd8s2fH1mZ7cVbQ0nLp4xYaJ6tR9wUeS3gP8Zp8 you@example.com
```

> -t ed25519 指定算法，比老的 rsa 更短更安全；-C 只是备注，写在公钥末尾，方便你在 GitHub 上认出是哪台机器的。两个回车都直接按，就是默认路径、不设口令。已有同名文件时它会问你 Overwrite (y/n)，拿不准就输 n 换个文件名。

## 看一眼公钥，贴到该贴的地方

```bash
cat ~/.ssh/id_ed25519.pub        # 输出以 ssh-ed25519 开头的一整行
pbcopy < ~/.ssh/id_ed25519.pub   # macOS 复制到剪贴板
clip < ~/.ssh/id_ed25519.pub     # Windows cmd 复制
# PowerShell 用：Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

> 这一整行就是要贴进 GitHub 的 Settings、SSH and GPG keys、New SSH key 里的内容。贴错成私钥（文件名不带 .pub 的那个）是最常见的失误，一旦贴出去就立刻作废并重新生成。

## 把公钥装到服务器上

```bash
$ ssh-copy-id -i ~/.ssh/id_ed25519.pub root@203.0.113.10
Number of key(s) added: 1

Now try logging into the machine, with:   ssh root@203.0.113.10
and check to make sure that only the key(s) you wanted were added.
```

> 它会用密码登录一次，把你的公钥追加到服务器上 ~/.ssh/authorized_keys 末尾。Windows 没有 ssh-copy-id，用这句代替：type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh root@203.0.113.10 "cat >> ~/.ssh/authorized_keys"。

## ~/.ssh/config：给常用服务器起别名

```bash
Host vps
    HostName 203.0.113.10
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60

Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
```

> 配好之后只敲 ssh vps 就行，端口和密钥自动带上。ServerAliveInterval 60 表示每分钟发一次心跳，防止空闲连接被路由器掐断。这个文件的权限必须是 600，否则 ssh 会告警 Bad owner or permissions 并整份忽略它。

## 权限 600 是硬要求

OpenSSH 拒绝使用权限过宽的私钥，会直接报错退出：`WARNING: UNPROTECTED PRIVATE KEY FILE!` 后面跟 `Permissions 0644 for /home/you/.ssh/id_ed25519 are too open.`

Linux 和 macOS 上执行四条：`chmod 700 ~/.ssh`、`chmod 600 ~/.ssh/id_ed25519`、`chmod 644 ~/.ssh/id_ed25519.pub`、`chmod 600 ~/.ssh/config`。**不要用 chmod 777 绕过去**：那是把私钥变成全机器可读，等于把钥匙挂在门上。

Windows 没有这套权限位，对应的是文件 ACL，一般不用改；只有把 .ssh 目录复制进 WSL 时才要注意属主。

## 四个真实报错，对着查

| 报错文本 | 原因 | 处理 |
| --- | --- | --- |
| Permission denied (publickey). | 服务器没找到匹配的公钥，或本地没提供私钥 | 确认 ssh-copy-id 成功，加 -v 看实际用了哪把钥匙 |
| Permissions 0644 ... are too open | 私钥权限太宽 | chmod 600 ~/.ssh/id_ed25519 |
| WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED! | 服务器换过密钥，或你连到了别的机器 | 核对后 ssh-keygen -R 主机名 删掉旧记录 |
| Connection refused | 端口没开、sshd 没跑，或防火墙拦了 | 换端口试 ssh -p 2222，再查服务器上 sshd 是否在跑 |

## Permission denied (publickey) 的排查顺序

### 1. 先加 -v 看清楚

执行 `ssh -v root@203.0.113.10`，重点看 `Offering public key:` 和 `Authentications that can continue:` 两行。前者列出你实际提供的钥匙，后者告诉你服务器还接受哪些方式。

### 2. 确认公钥真的到了服务器

用密码登录一次，执行 `cat ~/.ssh/authorized_keys`，看有没有你那一整行。**注意是不是同一个用户**：装到 root，却用 deploy 登录，一样不通。

### 3. 检查服务器端权限

服务器上 `~/.ssh` 要 700，`authorized_keys` 要 600，家目录不能被组或其他用户写。`chmod g-w,o-w ~` 能修掉最常见的那种。

### 4. 确认 sshd 允许密钥登录

`/etc/ssh/sshd_config` 里应为 `PubkeyAuthentication yes`，改完 `systemctl restart ssh`。**改之前先留一个已登录的会话别关**，配错会把自己锁在外面。

### 5. 换一把钥匙显式指定

本地有多把钥匙时，默认可能只试 id_rsa。显式指定：`ssh -i ~/.ssh/id_ed25519 root@203.0.113.10`。

## GitHub 上用同一套密钥

```bash
$ ssh -T git@github.com
Hi yourname! You've successfully authenticated, but GitHub does not provide shell access.
```

> 第一次执行会问要不要信任 github.com 的指纹，输 yes。看到这行就说明公钥生效了，之后 git clone git@github.com:用户名/仓库.git 不再要密码。报 Permission denied (publickey) 就回到上一步的清单，多半是公钥没加到 GitHub，或者密钥不在默认路径。

> **注意：私钥一旦泄露，删掉也没用**
>
> 私钥文件名不带 .pub，内容以 `-----BEGIN OPENSSH PRIVATE KEY-----` 开头。它一旦进了聊天记录、截图、Git 提交，就等于钥匙已经公开，**事后删掉不算补救**，要按已泄露处理。
>
> 处理顺序：登录服务器，把 authorized_keys 里对应的那行删掉；在 GitHub 上删掉那把 key；重新 `ssh-keygen` 生成一对。如果私钥进过 Git 历史，还要用 `git filter-repo` 清掉，因为仓库可能早被克隆过。

## 动手练一练

生成一把专用的 ed25519 密钥，把公钥装到一台你能登录的服务器上，再配一个别名连一次。

1. 生成一把带备注的密钥（已有旧密钥就换个文件名，别覆盖） —— `ssh-keygen -t ed25519 -C "dev@laptop"`
2. 把公钥装到服务器，然后只用密钥登录一次 —— `ssh-copy-id -i ~/.ssh/id_ed25519.pub root@203.0.113.10`
3. 在 ~/.ssh/config 里加 Host 别名，用别名连 —— `ssh vps`
4. 修好权限，并验证 GitHub 那边的密钥状态 —— `chmod 600 ~/.ssh/id_ed25519 && ssh -T git@github.com`

**完成标准**：用别名 ssh vps 连上去不再询问密码，ssh -T git@github.com 返回以 Hi 开头、带你自己用户名的一行。

## 小测验

<details>
<summary>展开题目与答案（3 题）</summary>

**1. 哪个文件可以安全地发给别人，或者贴到网站上？**

- A. id_ed25519
- B. id_ed25519.pub
- C. 两个都不行
- D. 两个都可以

<details><summary>答案</summary>

**B**。带 .pub 的是公钥，本质是一把锁，放进服务器的 authorized_keys 或 GitHub 上都是它的正常用途。不带 .pub 的私钥是钥匙，任何情况下都不该离开你的机器。

</details>

**2. 私钥权限是 0644，ssh 会怎样？**

- A. 照常登录
- B. 忽略权限继续，只是速度变慢
- C. 报 UNPROTECTED PRIVATE KEY FILE 并拒绝使用这把钥匙
- D. 自动把它改回 0600

<details><summary>答案</summary>

**C**。OpenSSH 拒绝使用组或其他用户可读的私钥，报 Permissions 0644 are too open 后直接退出，服务器都连不上。改成 chmod 600 就能解决，不要为了省事用 777，那等于让整台机器都能读你的钥匙。

</details>

**3. 连一台重装过的服务器，报 REMOTE HOST IDENTIFICATION HAS CHANGED，怎么办？**

- A. 把 known_hosts 整个删掉
- B. 确认是同一台机器后，用 ssh-keygen -R 主机名 删掉它的旧记录
- C. 在 ssh 命令里关掉严格检查
- D. 重新生成一对密钥

<details><summary>答案</summary>

**B**。这个警告表示服务器指纹和上次记录不一致，可能是重装，也可能是有人在中间冒充。先核对，再用 ssh-keygen -R 精确删掉那一台，别整个删 known_hosts。

</details>

</details>

---

[← Docker 基础：把环境装进盒子](04-Docker-基础-把环境装进盒子.md) · [目录](../Home.md) · [终端效率：别名、历史与补全 →](../efficiency/01-终端效率-别名-历史与补全.md)
