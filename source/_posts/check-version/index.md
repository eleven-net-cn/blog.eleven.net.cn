---
title: 利用 Git 钩子提交时修改版本号
date: 2020-06-11 10:07:48
tags: Git
---

前端项目在 git 提交时，往往会遗忘更新项目根目录的 package.json 文件的 version，通常不修改也不会有啥问题，但对于强迫症来说，不能忍！咱要改掉它......

#### 编写一个简单的 node 脚本 `check-version.js`

先安装几个依赖包 `yarn add inquirer chalk child_process -D`

```js
// /scripts/check-version.js

const inquirer = require('inquirer')
const chalk = require('chalk')
const { exec } = require('child_process')
const { name: projectName, version: versionCurrent } = require('../package')

const regVersion = /^[1-9]{1}\d*\.\d+\.\d+$/ // 示例: 1.0.0
// const regVersion = /^\d+\.\d+\.\d+$/ // 示例: 0.0.1 / 1.0.1
// const regVersion = /^\d+\.\d+\.\d+(-beta.?\d*)?$/ // 示例: 1.0.3 / 0.0.1-beta / 1.0.0-beta.3

console.log('\n')

// 确认 package.json 版本号
inquirer
  .prompt([
    {
      type: 'input',
      name: 'version',
      message: `请确认 ${projectName}/package.json/version 版本号（当前：${versionCurrent}）：\n`,
      default: versionCurrent,
      validate(version) {
        // 校验版本号的格式
        if (!regVersion.test(version)) {
          console.log(chalk.yellow('输入的版本号无效，请检查格式（示例：1.0.0、2.3.2）'))
          return false
        }
        return true
      },
    },
  ])
  .then(({ version: versionNew }) => {
    if (versionNew !== versionCurrent) {
      // 更新 package.json version，更新时不自动生成 tag
      command(`npm --no-git-tag-version version ${versionNew}`, {}, (error, stdout, stderr) => {
        if (!error) {
          console.log(
            chalk.green(
              `\n${projectName} 版本号（项目根目录下的 package.json/version）更新成功，version: ${versionNew} ！`,
            ),
          )
          command(
            `git add package.json && git commit -m 'ci(package.json): 更新项目版本号为：${versionNew}'`,
          )
          console.log(`\n`)
          process.exit(0)
        } else {
          console.log(chalk.yellow(`\n更新版本号（${versionNew}）失败了~\n`))
          process.exit(1)
        }
      })
    } else {
      console.log(chalk.green(`\n本次版本号未做修改，version: ${versionNew} ！\n`))
    }
  })

function command(cmd, options, callback) {
  console.log('\n')
  console.log(chalk.cyan(cmd.toString()))
  return exec(cmd, { ...options }, callback)
}
```
    
#### 配置 Git 钩子

1. 先安装 `husky`，用于管理 git 钩子，当然，使用原生的也可以。

    ```bash
    yarn add husky -D
    ```

2. 在项目的 package.json 文件中增加以下配置：

    ```json
    "husky": {
      "hooks": {
        "post-commit": "exec < /dev/tty && node scripts/check-version.js"
      }
    },
    ```

#### 使用

完成以上配置后，后续在执行 git commit 提交代码时，会自动在终端弹出交互，提示修改 package.json 文件的 version，如下图：

![](https://user-gold-cdn.xitu.io/2020/6/10/1729d6b37f7e5254?w=1326&h=348&f=png&s=55796)

如果不需要修改直接敲击 Enter 跳过，需要则输入新的版本号，会自动执行命令修改 package.json 文件的 version，并自动提交刚刚的修改，接下来 `git push` 推送代码即可。