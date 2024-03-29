# Windows SSH 密钥设置指南

## 安装 OpenSSH 服务端

::: tip
需要管理员权限。
:::

通过 Powershell 执行以下命令安装：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

在 Powershell 中输入：

```powershell
Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
```

若返回内容中有如下字段则代表安装成功：

```powershell
Name  : OpenSSH.Server~~~~0.0.1.0
State : Installed
```

## 启动 SSH 服务并设置为自动启动

::: tip
需要管理员权限。
:::

启动 `sshd` 服务：

```powershell
Start-Service sshd
```

将 `sshd` 服务设置为自动启动：

```powershell
Set-Service -Name sshd -StartupType 'Automatic'
```

::: info
若不设置需要在每次重启后手动开启 `sshd`。
:::

确认防火墙规则：

```powershell
Get-NetFirewallRule -Name *ssh*
```

::: tip
防火墙规则一般在安装时会配置好，若安装时未添加防火墙规则"OpenSSH-Server-In-TCP"，则通过以下命令添加
```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```
:::

## 开启密钥登陆

See My Server 只支持密钥登陆，因此此处需要用户配置密钥登陆才可正常使用。

### 生成密钥

::: info
如果已经拥有 SSH 密钥，可跳过此步。
:::

::: warning
在客户端的执行的步骤。
:::

在客户端 Powershell 中执行以下命令：

```powershell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

此命令会在 `~/.ssh` 目录下生成私钥 `id_rsa` 和公钥 `id_rsa.pub`。

### 配置密钥登陆

::: warning
在服务端的执行的步骤。
:::

#### 配置公钥

打开客户端生成的 `id_rsa.pub`，将公钥内容复制到服务端 `~/.ssh` 目录下的 `authorized_keys` 文件中。

::: info
实际上 Windows 默认的 `authorized_keys` 位于 `ProgramData\ssh\administrators_authorized_keys`，是否更改应根据个人需求和现实情况而定。
:::

#### 修改文件权限

在服务端 Powershell 中执行以下命令：

```powershell
icacls.exe "C:\Users\username\.ssh\authorized_keys" /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
```

#### 修改 SSH 配置文件

::: tip
需要管理员权限。
:::

```
#允许公钥授权访问，确保条目不被注释
PubkeyAuthentication yes

#授权文件存放位置，确保条目不被注释
AuthorizedKeysFile	.ssh/authorized_keys

#可选，关闭密码登录，提高安全性
PasswordAuthentication no

#注释掉默认授权文件位置，确保以下条目被注释
#Match Group administrators
#       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

#### 重启 `sshd` 服务

::: tip
需要管理员权限。
:::

在服务端 Powershell 中执行以下命令：

```powershell
Restart-Service sshd
```






## 参考

[Windows OpenSSH 服务器启用密钥登录](https://zhuanlan.zhihu.com/p/404179039)