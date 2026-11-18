---
title: "Alpine Linux：配置彩色终端"
description: "无需安装 Bash，让 Alpine 的 ash 显示彩色用户名、主机名、路径及文件类型。"
pubDatetime: 2026-11-18T12:30:00Z
featured: false
draft: false
tags:
  - alpine
  - linux
  - terminal
---

Alpine 默认使用 BusyBox `ash`。无需安装 Bash，就能获得类似 Debian 的彩色提示符和 `ls` 输出。

## 预期效果

```text
root@server:/etc/ssh# ls
user@server:~$ ls
```

- **绿色**：用户名与主机名；**蓝色**：当前路径。
- root 以 `#` 结尾，普通用户以 `$` 结尾。
- `ls` 中目录通常为蓝色、可执行文件为绿色、符号链接为青色（实际取决于终端配色）。

## 全局设置

以 root 身份执行，对所有使用登录 shell 的用户生效：

```sh
cat > /etc/profile.d/color-shell.sh <<'EOF_SHELL'
case $- in
  *i*)
    PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
    alias ls='ls --color=auto -F'
    alias ll='ls -lh --color=auto -F'
    alias la='ls -la --color=auto -F'
    ;;
esac
EOF_SHELL
```

**一行搞定**（与上面的配置等效，二选一即可）：

```sh
printf '%s\n' 'case $- in' '  *i*)' "    PS1='\\[\\033[01;32m\\]\\u@\\h\\[\\033[00m\\]:\\[\\033[01;34m\\]\\w\\[\\033[00m\\]\\$ '" "    alias ls='ls --color=auto -F'" "    alias ll='ls -lh --color=auto -F'" "    alias la='ls -la --color=auto -F'" '    ;;' 'esac' > /etc/profile.d/color-shell.sh
```

重新登录 SSH 即可生效；当前会话可运行：

```sh
. /etc/profile.d/color-shell.sh
```

如果要更改机器名（以 `server` 为例）：

```sh
hostname server && echo server > /etc/hostname
```

> 若某个用户的 `~/.profile` 自行设置了 `PS1` 或 `ls` 别名，可能覆盖上述全局配置。`ls` 的颜色功能还取决于 BusyBox 编译选项。

## 恢复默认

```sh
rm /etc/profile.d/color-shell.sh
```

重新登录后生效。
