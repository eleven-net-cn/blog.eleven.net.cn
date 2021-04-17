---
title: Apple M1 芯片问题汇总
date: 2021-04-14 13:24:43
category: MacOS
tags: MacOS
cover: /imgs/arm_core.jpeg
---

## 使用 Rosetta 打开

https://www.macwk.com/article/apple-silicon-m1-application-crash-repair

## nvm 终端切换 x86/arm

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
