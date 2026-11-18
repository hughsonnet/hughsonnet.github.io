---
title: "Alpine Linux：配置 Debian 风格的彩色终端"
description: "使用 Alpine 默认的 BusyBox ash，为 root 和普通用户统一设置彩色命令提示符，并让 ls 自动区分目录与文件类型。"
pubDatetime: 2026-11-18T12:30:00Z
featured: false
draft: false
tags:
  - linux
  - alpine
  - terminal
---

Alpine Linux 默认使用 BusyBox `ash`，终端配置比较简洁。其实不需要额外安装 Bash，也能拥有类似 Debian 的彩色提示符和目录颜色。下面采用全局配置方式：一次设置，root 和普通用户都能使用。

## 预期效果

```text
root@hk:/etc/ssh# ls
chinese@hk:~$ ls
```

其中，用户名和主机名显示为**绿色**，当前路径显示为**蓝色**；root 以 `#` 结尾，普通用户以 `$` 结尾。执行 `ls` 时，目录、可执行文件和符号链接也会根据类型着色。

## 设置主机名

以 root 身份执行（将 `hk` 换成自己的主机名）：

```sh
hostname hk
printf '%s\n' hk > /etc/hostname
```

这样既修改当前主机名，也保存重启后使用的主机名。

## 为所有用户配置彩色提示符

使用 `/etc/profile.d/` 比反复往 `/etc/profile` 末尾追加内容更容易维护。以 root 身份创建文件：

```sh
mkdir -p /etc/profile.d
cat > /etc/profile.d/color-shell.sh <<'EOF_SHELL'
# Alpine BusyBox ash: colorful prompt and ls aliases
case $- in
  *i*)
    PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
    alias ls='ls --color=auto -F'
    alias ll='ls -lh --color=auto -F'
    alias la='ls -la --color=auto -F'
    ;;
esac
EOF_SHELL
chmod 644 /etc/profile.d/color-shell.sh
```

配置中 `\u` 表示当前用户名，`\h` 表示主机名，`\w` 表示当前路径，`\$` 会根据用户身份显示 `#` 或 `$`。ANSI 颜色码 `32` 对应绿色，`34` 对应蓝色。

重新通过 SSH 登录后，交互式登录 shell 应会读取全局配置。要在当前会话预览效果，也可以执行：

```sh
. /etc/profile.d/color-shell.sh
```

> 如果某个用户的 `~/.profile` 另外设置了 `PS1` 或 `ls` 别名，个人配置可能覆盖全局设置。检查时可以先运行 `printf '%s\n' "$PS1"` 和 `alias ls`。

## 检查文件和目录颜色

```sh
mkdir -p ~/color-test
printf '#!/bin/sh\necho ok\n' > ~/color-test/run.sh
chmod +x ~/color-test/run.sh
ls -F ~/color-test
```

常见的默认配色是：目录蓝色、可执行文件绿色、符号链接青色、普通文件使用终端默认颜色。具体颜色取决于 `ls` 的实现和终端主题。

Alpine 自带的 BusyBox `ls` 通常已足够，不必为着色单独安装 Bash 或 GNU coreutils。若 `ls --color=auto` 提示不支持，可先运行 `ls --help` 查看系统 BusyBox 的编译选项。

## 验证普通用户

如果系统中已经有 `chinese` 用户，可以从 root 登录环境测试：

```sh
su - chinese
```

预期提示符：

```text
chinese@hk:~$
```

使用 `su -` 而不是 `su`，是为了加载该用户的登录环境。通过 SSH 正常登录同样会加载登录 shell 的配置。

## 恢复默认设置

不再需要时，删除独立配置文件即可：

```sh
rm /etc/profile.d/color-shell.sh
```

下次登录便不再加载这套全局提示符和别名。如果此前还修改过个人 `~/.profile`，需要另外检查并移除其中的重复配置。

---

对于只有几 GB 磁盘的 Alpine VPS，保留 BusyBox `ash` 加一份简洁的 `/etc/profile.d/` 配置，已经足够满足日常 SSH 管理需求。
