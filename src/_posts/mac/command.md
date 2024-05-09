---
title: Apple M1 芯片问题汇总
date: 2021-04-14 13:24:43
category: MacOS
tags: MacOS
cover: /imgs/arm_core.jpeg
---

## 使用 Rosetta 打开

https://www.macwk.com/article/apple-silicon-m1-application-crash-repair

## M1 Mac nvm 安装/升级/卸载

> 最好能够翻墙，并且设置终端可翻墙，否则速度太慢可能断掉。

1. 安装

   对于 M1 芯片的 Mac，低版本的 nvm 使用存在一些奇怪的问题，需要卸载后重装最新版本。

   ```zsh
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
   ```

   安装完成后，修改 zsh 终端配置，打开配置文件：

   ```zsh
   open ~/.zshrc
   ```

   在配置文件尾部，添加如下代码：

   ```zsh
   export NVM_DIR="$HOME/.nvm"
   [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
   [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
   ```

   保存后关闭，并在终端运行下发命令，让配置生效。

   ```zsh
   source ~/.zshrc
   ```

   安装工作结束！

2. 升级

   切换到 nvm 安装目录

   ```zsh
   cd ~/.nvm
   ```

   直接操作拉取最新代码即可

   ```zsh
   git fetch
   ```

3. 卸载

   移除 .nvm 目录即可，通常位置在 `~/.nvm`

   ```zsh
   rm -rf ~/.nvm
   ```

参考文章：https://blog.csdn.net/longgege001/article/details/114067242

## M1 Mac nvm 终端切换 x86/arm

```zsh
# Check what version you're running:
$ node --version
v14.15.4
# Check architecture of the `node` binary:
$ node -p process.arch
arm64
# This confirms that the arch is for the M1 chip, which is causing the problems.
# So we need to uninstall it.
# We can't uninstall the version we are currently using, so switch to another version:
$ nvm install v12.20.1
# Now uninstall the version we want to replace:
$ nvm uninstall v14.15.4
# Launch a new zsh process under the 64-bit X86 architecture:
$ arch -x86_64 zsh
# Install node using nvm. This should download the precompiled x64 binary:
$ nvm install v14.15.4
# Now check that the architecture is correct:
$ node -p process.arch
x64
# It is now safe to return to the arm64 zsh process:
$ exit
# We're back to a native shell:
$ arch
arm64
# And the new version is now available to use:
$ nvm use v14.15.4
Now using node v14.15.4 (npm v6.14.10)
```
