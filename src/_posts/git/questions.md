---
title: Git 常见问题
date: 2021-04-07 15:35:52
category: Git
tags: [Git]
cover:
---

Git 遇到的各类怪异问题，集中收集。

## 提示 hosts 问题

```zsh
Warning:Permanently added the RSA host key for IP address '13.229.188.59' to the list of known hosts.
```

遇到这种提示，多半是因为新机器刚配置的 git，或者常用的远程仓库更换了域名、IP 等。

可以将对应的域名、IP 配置进去，例如，将 github.com 添加进去，示例：

```zsh
vi /etc/hosts
insert 13.229.188.59 github.com
```

或者，删除 host 校验

```zsh
rm -rf ~/.ssh/known_hosts # 删除后重新操作即可
```

## Enter passphrase for key 每次都需要输入密码

```zsh
ssh-add ~/.ssh/id_rsa
```

## 设置某文件、目录不被跟踪

```zsh
git update-index --assume-unchanged <PATH>
```

对应的，允许某文件、目录被跟踪

```zsh
git update-index --no-assume-unchanged <PATH>
```