---
title: Hugo 毫秒级静态
date: 2022-11-26 10:10:10
---
## Hugo Demo  - Win

> `Hugo` 自称是使用 `Go`语言开发全世界最 `快`的静态博客框架
>
> - 2023.1.17 Hugo确实是全球最快的静态框架，它十分优秀，
>   但是它的官方文档有个很大的问题就是概念递归（而且没有解释）
>   [为什么 Hugo 的文档如此糟糕？](https://zhuanlan.zhihu.com/p/479344529)

### 参考

- 文档

  - [Hugo 中文文档 gohugo.org](https://www.gohugo.org/)
  - [Hugo 中文网 gohugo.cn](https://www.gohugo.cn/)
  - [Hugo 官网 gohugo.io](https://gohugo.io/)
  - [Hugo 中文帮助文档 hugo.aiaide.com](https://hugo.aiaide.com/)
  - [Hugo 中文文档 hugocn.netlify.app](https://hugocn.netlify.app/)
  - [Hugo In Action](https://hugo-in-action.foofun.cn/zh/docs/part1/)

### 安装

- [下载安装包 gohugoio/hugo](https://github.com/gohugoio/hugo/releases)

  `extened 版本`： 有些主题的需要利用进行 `SCSS/SASS` 构建，

  如果下 `普通版`就可能会报错显示： `you need the extended version to build SCSS/SASS`
- [Git 安装](http://git-scm.com/)（可选）
- [Hugo 安装指南](https://gohugo.io/getting-started/installing/) （基本上都是使用各个包工具安装）

### 环境变量

`此电脑 – 属性 – 高级系统设置 – 高级 – 环境变量 – Path`

添加存放 `Hugo.exe`的文件夹路径

### 检查

查看 `Hugo版本`来检查是否安装成功

```bash
hugo version
```

正常返回示例： （有版本号）

```diff
+ hugo v0.108.0-a0d64a46e36dd2f503bfd5ba1a5807b900df231d windows/amd64 BuildDate=2022-12-06T13:37:56Z VendorInfo=gohugoio
```

### 步骤

1. 新建自定义文件夹

   ```bash
   hugo new site blog
   ```

   新建完成的文件夹目录树如下：

   ```
   .
   ├── archetypes
   │   └── default.md # 默认生成格式
   ├── config.toml # 配置文件
   ├── content # 存放Markdown文件
   ├── layouts # 页面结构
   ├── static # 静态资源
   └── themes # 主题
   ```

   > 其中 `config.toml` # 配置文件的内容一般默认如下
   >
   > ```
   > baseURL               = "https://127.0.0.1:1313"
   > languageCode          = "zh-CN"
   > title                 = "站点标题"
   > theme                 = "主题"
   > # 解决Github Page的CSS、JS丢失
   > # remote_theme          = "minima" 
   > preserveTaxonomyNames = true
   > disablePathToLower    = true
   > hasCJKLanguage        = true
   > ```
   >
   > `Hugo`以前旧主题采用 `.yml`文件，新版大多改用 `.toml`文件，以及 `多文件`配置
   >
   > ```
   > config
   > └─ _default
   >     ├─ config.toml
   >     ├─ menus.toml
   >     └─ params.toml
   > ```
   >
2. Hugo并没有默认主题，需要自己去选[官方主题列表](https://themes.gohugo.io/)

   此处[快速入门教程](https://www.gohugo.cn/getting-started/quick-start/)使用的是 [hyde主题](https://github.com/spf13/hyde) 作为示例。

   > `hyde主题`部署的站点参考此： [创建Hugo+Github Pages博客](https://ichochy.com/posts/20200802.html)
   >

   先下载主题并将其添加到站点的 `theme` 目录中：

   ```bash
   cd blog
   git init
   git submodule add https://github.com/spf13/hyde.git themes/hyde
   ```

   > *非 git 用户请注意：*
   >
   > - 如果没有安装 git，前往主题仓库下载主题压缩包。
   > - 解压 .zip 文件，获得主题文件夹目录。
   > - 将该目录重命名为 `hyde`，并将其移动到 `themes/` 目录。
   >

   接着将主题添加到站点的配置文件中：

   ```bash
   echo 'theme = "hyde"' >> config.toml
   ```
3. 新建文章

   使用 `new` 命令辅助工作（例如添加标题和日期）

   ```bash
   hugo new posts/A.md
   ```
4. 启动 Hugo 服务器

   ```bash
   hugo server --buildFuture
   ```

## 部署到Github

```text
hugo
cd public
git init
git remote add origin git@github.com:用户名/用户名.github.io.git
git add .
git commit -m 'init'
git push -f --set-upstream origin master
```

## Github Action

**区别**

> 手动部署到 `Github Pages`流程:
>
> - 使用Hugo生成静态网页，将静态网页 `push`到建好的 `Github Pages`的 `Repo` 中，一般是 `<username>/<username>.github.io`
>
> 自动部署 `Github Pages`。
>
> - 本地新建文章，`push`到 `Github`仓库的 `master`分支。`master`分支存放博客文章的源码。
> - `push` 操作自动触发预先配置的 `Actions`。
> - `GitHub Action`自动执行 `yml`文件中的"`action`"，构建打包，推送至 `gh-pages repo`。

### 存档

- `.github/workflows/github-pages.yml`展示

  > `同仓库`下的 `master`分支 `输出`到 `gh-pages`分支
  >

  ```yaml
  name: github pages

  on:
    push:
      branches:
        - master  # Set a branch that will trigger a deployment
    pull_request:

  jobs:
    deploy:
      runs-on: ubuntu-22.04
      permissions:
        contents: write
      concurrency:
        group: ${{ github.workflow }}-${{ github.ref }}
      steps:
        - uses: actions/checkout@v3
          with:
            submodules: true  # Fetch Hugo themes (true OR recursive)
            fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

        - name: Setup Hugo
          uses: peaceiris/actions-hugo@v2
          with:
            hugo-version: 'latest'
            extended: true

        - name: Build
          run: hugo 

        - name: Deploy
          uses: peaceiris/actions-gh-pages@v3
          if: github.ref == 'refs/heads/master'
          with:
            github_token: ${{ secrets.GITHUB_TOKEN }}
            publish_dir: ./public
            publish_branch: gh-pages
  ```
- ⭐️部署到外部存储库: 参考[💾external_repository](https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-deploy-to-external-repository-external_repository)

  ```yaml
        - name: Deploy
          uses: peaceiris/actions-gh-pages@v3
          with:
            # ❌deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
            # ⚠️注意 GITHUB_TOKEN 没有访问外部存储库的权限
            # 参考 ⭐️ 部署到外部存储库 external_repository ⬇️
            # https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-deploy-to-external-repository-external_repository   
            personal_token: ${{ secrets.CI_TOKEN }}
            external_repository: 用户名/用户名.github.io
            publish_branch: gh-pages  
            publish_dir: ./public 
  ```

### 部署到[Cloudflare](https://www.cloudflare.com/zh-cn)

- [Hugo by Github with Cloudflare Pages](https://immmmm.com/hugo-github-cloudflare/)
- [Hi , Cloudflare Pages](https://immmmm.com/hi-cloudflare/)
- 让 `CF` 构建 `Hugo` 需要加个 `环境变量`，指定高版本 `HUGO_VERSION` 为 `0.92.0`

## sh脚本

### 本地预览server

新建个 `.sh`文件，方便快速 `本地启动Hugo`

```sh
# ! /bin/sh

# 本地启动Demo
hugo server --disableFastRender
# 此处
# --disableFastRender 启用对更改的完全重新渲染
# --buildFuture 解决日期发布的时区问题
# --minify 压缩生成的静态资源

```

### 提交Push

`hugo` 一键打包上传 `github` ，前提是之前上传过 `GitHub` 仓库

采用 `windows` 支持的 `.sh` 可执行文件，在根目录新建一个 `hugo.sh` 文件，里面放上一些命令

```
hugo

# cd public

time=$(date "+%Y-%m-%d %H:%M:%S")
echo $time

git add .
git commit -m "💾$time"
git push
exit
```

可以在文件夹双击运行 `hugo.sh`，也可以在终端输入命令。

```
.\hugo.sh
```

省去了输入繁琐的 `Git` 上传指令，并且自动生成当前系统时间作为 `commit`

上传的是整体的 `hugo` 目录，如果只想上传生成的 `/public` ，在中间加入 `cd public`

## 屏蔽项.gitignore

```
# Hugo 默认输出目录
public/
resources/

# NPM
node_modules/

## OS Files
# Windows
Thumbs.db
ehthumbs.db
Desktop.ini
$RECYCLE.BIN/

# OSX
.DS_Store

# Linux
.directory

# Hugo
.hugo_build.lock
jsconfig.json

```

## [关于 `_index.md`](https://gohugo.io/content-management/sections)

官方文档： [https://gohugo.io/content-management/sections](https://gohugo.io/content-management/sections)

- 简单来说： 加了 `_index.md`的目录会被视作 `Section`
- 部分需要借助 `Section`来实现的功能需要在指定文件夹添加 `_index.md`

```
---
layout: index
---
```

## 其他内容

1. 图片引用

   - 可参考此 `posts`结构

     ```
     content
     ├─ posts
     │    ├─ Hugo 毫秒级 静态框架
     │    │    └─ index.md
     │    ├─ 博文文章名
     │    │    ├─ img
     │    │    │    ├─ 图片.png
     │    │    └─ index.md
     └─ search.md
     ```
     | 图片路径            | `Markdown`写法                  |
     | ------------------- | --------------------------------- |
     | `img/example.png` | `![example](./img/example.png)` |
   - 或者直接放在 `static`文件夹内

     | 图片路径               | `Markdown`写法             |
     | ---------------------- | ---------------------------- |
     | `static/example.png` | `![example](/example.png)` |
2. 站点数据类内容

   - 谷歌分析 | [https://analytics.google.com/analytics/web](https://analytics.google.com/analytics/web)
   - 谷歌SEO | [https://search.google.com/search-console](https://search.google.com/search-console)
3. 添加其他静态页面

   - 直接扔在 `static`静态资源文件夹内即可，用 `URL/文件夹名字`来索引
4. [支持渲染 `html` 代码](https://zburu.com/archives/183.html/)

   - 写入配置，以便在文章中插入 `iframe`, `video`, 以及其他比如 `summary`等标签

     如果是 `config.toml`，加下面配置；

     ```
     [markup]
       [markup.goldmark]
         [markup.goldmark.renderer]
           unsafe = true
     ```
     如果是 `config.yaml`，加下面配置；

     ```
     markup:
       goldmark:
         renderer:
           unsafe: true
     ```
     否则就会显示下面注释，自动过滤 `html` 代码。

     ```
     <!-- raw HTML omitted -->
     ```
5. [Hugo 文章日期设定上的小问题](https://blog.hly0928.com/post/hugo-post-date-issue/)

   - > Hugo 在生成静态页面的时候，不会生成超过当前时间的文章；而 Hugo 默认采用的是 [格林尼治平时](https://zh.wikipedia.org/wiki/%E6%A0%BC%E6%9E%97%E5%B0%BC%E6%B2%BB%E6%A8%99%E6%BA%96%E6%99%82%E9%96%93) (GMT)，比北京时间 (UTC+8) 晚了 8 个小时。也就是说，当北京时间在 08:00 之前，而将文章发布日期设在当天时，Hugo 就默认不会生成这个页面。
     >

     使用 `--buildFuture` 参数，这是比较简单的方式，只需要在用 Hugo 生成页面时加上 `--buildFuture` 参数即可，[Hugo官方文档](https://gohugo.io/functions/format/) 了解更多信息。

     用 Hugo 本地服务器预览页面时，也可以使用本参数：

     ```
     hugo server --buildFuture
     ```
