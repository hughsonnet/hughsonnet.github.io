# Windows 11 SMB 共享提示用户名或密码错误的解决方法

## 问题描述

使用 iPhone「文件」App 连接 Windows 11 的 SMB 共享文件夹时，即使输入正确的本地账户用户名和密码，仍提示「用户名或密码无效」或「认证错误」。

## 一、排查 ForceGuest 设置

以管理员身份打开 PowerShell 或 CMD，执行：

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v ForceGuest
```

查看返回结果：

- `0x0`：Classic 模式，使用实际本地账户进行网络身份验证，通常无需修改。
- `0x1`：Guest only 模式，将本地账户的网络登录映射为 Guest（来宾）账户。

如果 `ForceGuest` 为 `0x1`，而 Guest 账户又处于禁用状态，就可能导致 SMB 身份验证失败，即使输入的用户名和密码正确。

## 二、修改 ForceGuest

确认值为 `0x1` 后，以管理员身份执行：

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Lsa" /v ForceGuest /t REG_DWORD /d 0 /f
```

将 `ForceGuest` 修改为 `0`，使 Windows 使用真实的本地账户进行网络身份验证。

修改后可以重新执行第一步的查询命令，确认结果为 `0x0`。

## 三、重新连接 SMB 共享

在 iPhone「文件」App 中选择「浏览」→「···」→「连接服务器」。

输入 Windows 电脑的局域网地址，例如：

```text
smb://192.168.1.100
```

选择「注册用户」，填写：

- **用户名**：`计算机名\本地用户名`
- **密码**：Windows 本地账户密码（不是 PIN）

连接成功后即可访问已共享的文件夹。

> 注意：如果 `ForceGuest` 原本就是 `0x0`，应继续排查账户凭据及其他 SMB 认证设置，无需重复修改。不要通过启用 Guest 或关闭密码保护共享来解决此问题。
