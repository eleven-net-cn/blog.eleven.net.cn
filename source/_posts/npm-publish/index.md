---
title: NPM 包发布指南
date: 2020-06-11 10:21:34
tags: NPM
---

#### 一、准备工作
    
1. 在 NPM 官网注册账号，https://www.npmjs.com
    
    邮件验证的时候可能需要翻墙访问。
    
2. 本机安装 nodejs
    
3. 推荐也安装一下 nrm，方便随时切换 npm 源
    
    ```sh
    sudo npm i nrm -g
    ```
    
    nrm 常用命令
    
    ```sh
    nrm ls                  # 查看所有
    nrm use [目标源]         # 切换至目标源
    ```
        
#### 二、package.json 文件

包的**根目录**需要有一个 package.json 文件，可以通过 `npm init` 命令去创建，示例如下：
    
```json
{
  "name": "@eleven.xi/reset.css",
  "version": "1.0.6",
  "description": "H5 网页 reset 方案，PC&mobile",
  "main": "lib/reset.css",
  "scripts": {
    "release": "npm publish . --access=public"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/Eleven90/reset.css.git"
  },
  "keywords": [
    "reset.css"
  ],
  "author": "Eleven",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/Eleven90/reset.css/issues"
  },
  "homepage": "https://github.com/Eleven90/reset.css#readme",
  "files": ["lib", "ReadMe.md"]
}
```

**文件中重点注意以下几项：**

1. main：指定包的入口文件
2. name：指定包名

    - 发布之前都要去[NPM 官网](https://www.npmjs.com)上搜索一遍，确认想要使用的包名，是否已经被占用。
        
    - 包名支持 [@scope]/[package name] 的形式，[@scope] 类似于命名空间的作用，NPM 默认允许你使用自己注册的用户名，或者在自己的账户下申请的 organizations。
        
        典型的例子，如 Babel，插件原先使用的是 babel-plugin-xxx 的格式命名，后来因为许多个人发布的包和官方的包命名格式一样，导致难以区分，现在 babel 官方所有的包都更换成了 @babel/xxx 的格式。
        
3. version：包的版本

    - 格式 `数字.数字.数字`，每一次发布，版本都必须要更新，只能往上增加。

4. keywords：关键词

    - 希望别人通过哪些关键词搜索到你的包，可以在这里添加。

5. files：指定哪些文件夹、文件将被发布

    - 如果你的项目目录下包含了一些隐私文件，不希望被发布出去，一定要注意配置此项，仅包含可以被发布的文件夹、文件。

    - 可以在包的根目录下新建文件 `.npmignore`，指定哪些文件不被发布，书写格式与 `.gitignore` 文件一致。
    
  6. license：协议，推荐阅读了解几种开源的协议：

      - [七种开源许可证](https://www.jianshu.com/p/86251523e898)

      - [SPDX License List](https://spdx.org/licenses/)
  
  7. description、repository、author、bugs、homepage 等项，通常也推荐填写完整，详细的 package.json 每一项的含义，推荐阅读这一篇：[npm package.json属性详解](https://blog.csdn.net/zhengxiuchen86/article/details/81285030) 。

#### 三、待发布的包结构

1. 移除不必要的代码

    发布出去的包，一般只需要包含用户使用时必须要 `install` 下载的文件即可，例如一些构建脚本等无关的代码文件，不必发布出去。

    例如：你有一个开源项目，同时，该项目也提供了包供开发者使用，最佳方式应该是将该项目源码推送到 GitHub 上开源，而发布到 NPM 仓库的包，尽量不要把项目源码目录、构建脚本等非必要的文件发布到 NPM 仓库，会增大包的体积，导致安装时间变长。

2. 发布的包尽量要做好语法转译，否则要在文档中说明，提醒使用者自己完成。

3. 保护隐私

    如果待发布的包，并非开源项目，而仅仅是为了提供开发者 NPM 安装、使用，一定要做好隐私保护工作，防止源码、隐私的文档等隐秘信息被发布到 NPM 仓库。可以选用的方法如下：

      1. `.gitignore` 设置忽略哪些文件

            .gitignore 设置的忽略文件，在 git 代码管理和 npm publish 都会被忽略。

      2. .npmignore 设置忽略哪些文件

            .npmignore 优先级更高，如果同时使用了 .npmignore 和 .gitignore，只有 .npmignore 会生效。

      3. package.json 文件的 files 字段

            直接在 package.json 文件中配置 `files`，指定发布哪些文件、目录，优先级高于 .npmignore 和 .gitignore。

4. 以下文件、目录在发布时，默认会被忽略

    ```txt
    .*.swp
    ._*
    .DS_Store
    .git
    .hg
    .npmrc
    .lock-wscript
    .svn
    .wafpickle-*
    config.gypi
    CVS
    npm-debug.log
    node_modules/
    ```

5. 以下文件、目录在发布时，默认会被包含，无法忽略掉

    ```txt
    package.json
    README (and its variants)
    CHANGELOG (and its variants)
    LICENSE / LICENCE
    ```

#### 四、发布

1. 登录 NPM 账号

    1. 在终端运行命令，填写账号、密码及邮箱。

        ```sh
        npm adduser / npm login
        ```

    2. 查看是否登录上了？

        ```sh
        npm who am i
        ```

    3. 通常只有一个人可以发布，也可以添加多人，相关命令如下：

        ```sh
        npm owner ls <package name>             # 查看
        npm owner add <user> <package name>     # 添加
        npm owner rm <user> <package name>      # 删除
        ```

2. 发布

    ```sh
    npm publish . --access=public
    ```

    1. 这里注意一下发布命令中的点 `.` ，如果不带 `.` ，偶尔碰到发布会出错。

    2. 包名重复（或者说已被占用）、未登录，都会导致发布失败，注意看提示信息。

    3. 注意版本号必须要递增，相同的版本号或版本号递减会发布失败。

3. 撤回已发布的版本

    ```sh
    npm unpublish -f <package name>@<package version>
    ```

    1. 包发布后的 72 小时内，可以撤回。

    2. 已撤回的版本，该版本号就不能再重新发布了，因为在 NPM 的仓库中已经有了记录。

4. 其它问题

    1. 偶尔可能会看到如下错误:

        ```sh
        no_perms Private mode enable, only admin can publish this module
        ```

        解决办法：

        ```sh
        npm config set registry http://registry.npmjs.org
        ```

    2. NPM 官方提供了发布的指导：[https://docs.npmjs.com/misc/developers](https://docs.npmjs.com/misc/developers) ，如果遇到一些奇怪的问题，建议前往阅读。