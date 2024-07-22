---
title: Git 非常规问题记录
date: 2021-04-07 15:35:52
category: Git
tags: [Git]
cover:
---

## 区分目录指定不同设置

在区分企业与个人项目的提交信息时较为常用

```zsh
# 推荐单独创建配置文件
touch ~/.gitconfig-xx
```

例如，需要单独设置 username/user-email

```shell
[user]
    name = xx
    email = xxx@gmail.com
```

```zsh
# 打开 Git 默认的配置文件，添加指定目录下的规则
open ~/.gitconfig
```

例如，指定 `~/xx/xx/` 目录使用上方创建的 `.gitconfig-xx` 配置，则在 `~/.gitconfig` 文件中添加如下配置：

```zsh
[includeIf "gitdir:~/xx/xx/"]
  path = ~/.gitconfig-xx
```

## 与 fork 的上游仓库保持同步

一般有 3 种办法可用：

1. 命令行同步（安全，推荐）

```zsh
git remote -v
# 将上游仓库添加进来
git remote add upstream git@github.com:xxx/xxx.git
# 拉取更新
git fetch upstream
# 合并过来
git merge upstream/master
git push origin master
```

若干可以注意的地方，如下图：

![](/imgs/git_fetch_fork_repo.png)

2. 借助第三方库：https://github.com/wei/pull (如果自己有修改，会被覆盖。所以，比较适合仅参与 PR 的项目同步)

3. 借助机器人程序 Backstroke：https://github.com/backstrokeapp/server

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
