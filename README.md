# Welcome!

https://blog.eleven.net.cn/

## 运行命令

- 新建文章（指定嵌套的文件夹、文件名及文章标题）

  ```bash
  npx hexo new "[post_title]" -p [folder_path/file_name.md]
  ```

  - post_title  文章标题
  - folder_path 文章所在文件夹
  - file_name   文章文件名

- 本地调试

  ```bash
  yarn serve
  ```

- 部署发布

  ```bash
  yarn release
  ```

- 其它

  ```bash
  yarn build    # 编译
  yarn clean    # 清空编译结果
  yarn deploy   # 部署（到 github pages）
  ```

## 目录

```bash
blog
  ├── .deploy_git/        # deploy 部署目录
  ├── node_modules/
  ├── public/             # 编译结果目录
  ├── scaffolds/          # 模版
  ├── src/                # 文章书写
  │     ├── _posts/         # 文章
  │
  ├── themes/             # 主题
  │
  ├── _config.yml         # hexo 基础配置
  ├── .gitignore 
  ├── db.json
  ├── package.json
  ├── README.md
  ├── yarn.lock
```

## Quick Start About Hexo

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

> 更详细的 Hexo 命令行说明，请前往：https://hexo.io/zh-cn/docs/commands 。

### Create a new post

``` bash
$ npx hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ npx hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ npx hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ npx hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)
