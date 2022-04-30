---
title: Git 命令清单
date: 2020-03-02 12:12:29
category: Git
tags: Git
cover:
---

Git 常用命令做一波整理，方便随时查阅。

> 主要内容出自阮一峰大佬的科普文章：http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html 。

![](https://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png)

几个专用名词的译名如下：

- Workspace：工作区
- Index / Stage：暂存区
- Repository：仓库区（或本地仓库）
- Remote：远程仓库

## 一、新建代码库

```zsh
# 在当前目录新建一个 Git 代码库
$ git init

# 新建一个目录，将其初始化为 Git 代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

## 二、配置

Git 的设置文件为 `.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

```zsh
# 显示当前的 Git 配置
$ git config --list

# 编辑 Git 配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息(设置本机的用户名，邮箱)
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"

# 查看用户信息
$ git config user.name -g
$ git config user.email -g

# 查看公开密钥
$ cat ~/.ssh/id_rsa.pub

# 验证密钥是否已连接成功(以 github 为例)
$ ssh -T git@github.com

# 创建 SSH Key
$ ssh-keygen -t rsa -C "youremail@example"

# 解决每次要求输入用户名/密码的问题
$ git config --global credential.helper store

# 设置 http 代理
$ git config --global http.proxy 10.100.10.100:3128

# 设置 Git 对文件/文件夹大小写敏感
$ git config core.ignorecase false

# 设置全局 pull rebase
$ git config --global pull.rebase true

# gitlab ip 变化后
$ rm -rf ~/.ssh/known_hosts 删除后重新操作即可
```

## 三、增加/删除文件

```zsh
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .
$ git add -A 添加所有文件

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

## 四、代码提交

```zsh
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次 commit 之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有 diff 信息
$ git commit -v

# 使用一次新的 commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次 commit 的提交信息
$ git commit --amend -m [message]

# 重做上一次 commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

## 五、分支

```zsh
# 有时候看不到远程的分支，那么更新下远程库的索引
$ git fetch

# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 拉取一个本地不存在的新分支，并切换到该分支
$ git checkout -b [branch-name] [origin/branch-name]

# 新建一个分支，指向指定 commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个 commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除分支(如果要删除的分支，本地仓库有代码没有 push 到远程仓库，-d 是删不掉的，需要用-D; 切回去先 push 再删也行.)
$ git branch -D [branch-name]

# 批量删除本地分支（排除 master/develop/test 分支）
$ git branch -a | grep -v -E 'master|develop|test' | xargs git branch -D

# 批量删除本地分支（仅包含 releases）
$ git branch -a | grep 'releases' | xargs git branch -D

# 批量删除远程分支（排除 master/develop/test 分支）
$ git branch -r| grep -v -E 'master|develop|test' | sed 's/origin\///g' | xargs -I {} git push origin :{}

# 批量删除远程分支（仅包含 releases）
$ git branch -r| grep 'releases' | sed 's/origin\///g' | xargs -I {} git push origin :{}

# 重命名分支(不会覆盖已存在的同名分支)
$ git branch -m [branch-name] [new-name]

# 重命名分支(会覆盖已存在的同名分支)
$ git branch -M [branch-name] [new-name]

# 删除远程分支
$ git push origin -d [branch-name] （将服务器上的“远程分支”删除，删除远程分支常用这个。）
$ git branch -dr [remote/branch] （仅将本地的“远程分支”删除）

# 处理已经不存在的分支（有时不存在了，但仍然显示）
$ git remote show origin 查看远程库和分支的情况
$ git remote prune origin 可以移除一些已经不存在的分支

# 比较两个分支不同
$ git diff [分支 A] [分支 B] >> xx.txt （比较两个分支不同，输出到某文本）
```

## 六、标签

tag 对应某次 commit 节点，是一个点，是不可移动的。

branch 对应一系列 commit，是很多点连成的一根线，有一个 HEAD 指针，可以依靠 HEAD 指针移动。

两者的区别决定了使用方式，改动代码用 branch ，不改动只查看用 tag。

> tag 和 branch 的相互配合使用，有时候起到非常方便的效果，例如：已经发布了 v1.0、v2.0、v3.0 三个版本，这个时候，我突然想不改现有代码的前提下，在 v2.0 的基础上加个新功能，作为 v4.0 发布。就可以检出 v2.0 的代码作为一个 branch ，然后作为开发分支。

```zsh
# 列出所有 tag
$ git tag

# 新建一个 tag 在当前 commit
$ git tag [tag]

# 新建一个 tag，并加上注释说明
$ git tag -a [tag-name] -m "some words"

# 新建一个 tag 在指定 commit
$ git tag [tag] [commit]

# 删除本地 tag
$ git tag -d [tag]

# 删除远程 tag
$ git push origin -d [tagName]
$ git push origin :refs/tags/[tagName]

# 查看 tag 信息
$ git show [tag]

# 提交指定 tag
$ git push [remote] [tag]

# 提交所有 tag
$ git push [remote] --tags

# 新建一个分支，指向某个 tag
$ git checkout -b [branch] [tag]

# 重命名 tag (新建 tag 指向原来的 tag => 再删除旧的 tag => 并删除远程旧 tag)
$ git tag [new-tag-name] [old-tag-name]
$ git tag -d [old-tag-name]
$ git push [remote] -d [old-tag-name]
```

## 七、查看信息

```zsh
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示 commit 历史，以及每次 commit 发生变更的文件
$ git log --stat

# 查看分支合并图
$ git log --graph

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个 commit 之后的所有变动，每个 commit 占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个 commit 之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次 diff
$ git log -p [file]

# 显示过去 5 次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个 commit 的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新 commit 之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog
```

## 八、远程同步

```zsh
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库(本地仓库与远程仓库建立连接)，并命名(别名)
$ git remote add [shortname] [url]

# 删除一个远程仓库
$ git remote rm [shortname]

# 远程仓库更名后，修改连接地址
$ git remote set-url [shortname] [new-url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 推送分支到远程仓库(第一次需要-u，建立关系)
$ git push -u [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force
git push [remote] -f

# 推送所有分支到远程仓库
$ git push [remote] --all
```

## 九、撤销

### Checkout

checkout，没有 add 到暂存区的，撤回到版本库的状态; 已 add 到暂存区未 commit，其后的修改，撤回到暂存区状态。

```zsh
# 恢复暂存区的指定文件到工作区 (回退某文件到工作区)
$ git checkout [file]

# 恢复某个 commit 的指定文件到暂存区和工作区 (回退到未 commit 状态)
$ git checkout [commit] [file]
# 例如: git checkout 36393f5fc07e54f6704d23d4d92bf0b3773e523c xxx/main/src/util/eg.js

# 恢复暂存区的所有文件到工作区 (回退所有文件到工作区)
$ git checkout .
```

### Reset

checkout 是撤销修改，reset 是在版本间穿梭。

```zsh
# 重置指定文件到指定版本
$ git reset [版本号] [文件路径(从顶层一路相对下来)]

# 重置暂存区的指定文件，与上一次 commit 保持一致，但工作区不变
$ git reset [file]

# 重置暂存区
$ git reset HEAD

# 重置暂存区与工作区，与上一次 commit 保持一致
$ git reset --hard

# 重置当前分支的指针为指定 commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的 HEAD 为指定 commit，同时重置暂存区和工作区，与指定 commit 一致
$ git reset --hard [commit] => 回退到某个版本
$ git reset --hard HEAD~1 => 回退到上一个版本

# 重置当前 HEAD 为指定 commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个 commit，用来撤销指定 commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
# 适用场景： 如果我们想撤销之前的某一版本，但是又想保留该目标版本后面的版本，记录下这整个版本变动流程，就可以用这种方法。
$ git revert [commit]
```

Reset 和 Revert 的对比：https://blog.csdn.net/yxlshk/article/details/79944535

### Stash

stash 可用于临时解决 bug 时，将眼前的工作隐藏；随后切换到解决 bug 的分支，完成 bug，提交，合并；
回到原先工作的分支，查看隐藏，恢复；over!

```zsh
# 暂时将未提交的变化移除(隐藏工作现场)
$ git stash

# 将移除的移入(恢复工作现场)，同时之前 stash 的内容也即删除了(不想删除，请用 apply 命令)
$ git stash pop

# 查看所有隐藏的工作现场
$ git stash list

# 恢复 XX，同时把 stash 内容删除
$ git stash pop XX

# 恢复 xx 但 stash 内容不删除
$ git stash apply XX

# 删除 XX
$ git stash drop XX
```

## 十、其他

```zsh
# 生成一个可供发布的压缩包
$ git archive

# 停止追踪某文件，并保留在本地
$ git rm --cached XXXX

# 停止追踪某文件，并删除本地文件
$ git rm --f XXXX
```
