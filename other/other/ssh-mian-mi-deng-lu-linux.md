---
description: windows ssh登录linux时免密登录
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# SSH免密登录Linux

## Windows

Windows Terminal 生成 SSH 密钥

```powershell
ssh-keygen -t rsa
```

中途需要输入的选项，直接默认值回车即可。路径 `C:\Users\<user>\.ssh` 下生成

* id\_rsa
* id\_rsa.pub
* known\_hosts

## Linux

检查 Linux 是否有以下目录和文件，如果没有就创建

```shell
mkdir ~/.ssh
touch ~/.ssh/authorized_keys
```

修改权限

```sh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

上传 Windiws 生成的公钥到 Linux的`authorized_keys`

```sh
# cd至windows用户路径
cd C:\Users\<user>\.ssh

# user linux用户 ip 即服务器地址
scp .ssh/id_rsa.pub <user>@<ip>:~/.ssh

# 建议追加内容
cat .ssh/id_rsa.pub >> .ssh/authorized_keys

# 修改权限
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```

```sh
sudo vim /etc/ssh/sshd_config
# 确保SSH配置文件中开启以下（一般默认开启）
RSAAuthentication yes 
PubkeyAuthentication yes 
AuthorizedKeysFile .ssh/authorized_keys
```

重启 Linx SSH服务

```shell
systemctl restart sshd
```

## SSH登录 <a href="#ssh-deng-lu" id="ssh-deng-lu"></a>

Windows Terminal

```powershell
ssh <user>@<host>
```
