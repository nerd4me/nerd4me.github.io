---
title: 基于 GitHub Pages + Hexo 搭建个人博客
index_img: /img/Blog-Post_thumbnail.png
tags:
  - Hexo
  - Fluid
abbrlink: d2b9b4ed
date: 2022-10-19 22:30:53
---

{% note primary %}
本博客基于 `Github Pages + Hexo`，`Hexo` 是一个快速、简单且功能强大的博客框架。支持使用 `Markdown` 写文章，`Hexo` 会在几秒内生成带有各种自定义主题、并且集成了各项功能的网站页面。
{% endnote %}

### 零、准备工作

#### 1. 使用个人 [GitHub](https://github.com/) 创建仓库，并配置 `GitHub Pages`

{% note info %}
**注意:**
此仓库用于存放我们的博客页面，仓库名必须使用：`<GitHub用户名>.github.io` 格式，比如本博客对应的仓库名就是: `nerd4me.github.io`
{% endnote %}

仓库创建好之后，我们可以在仓库根路径下创建一个名为 `index.html` 的静态 `HTML` 文件来验证我们的网站是否搭建成功

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Nerd4me's Blog</title>
</head>
<body>
    <h1>Hello, Blog World ~</h1>
</body>
</html>
```

在 `nerd4me.github.io` 仓库对应的 `GitHub Pages` 设置页面 (访问路径：`Settings -> Pages`) 可以找到我们的主页访问地址：[https://nerd4me.github.io](https://nerd4me.github.io)，在浏览器中访问该地址，能正常访问表示我们的 `GitHub Pages` 搭建成功。

#### 2. 安装 `Git` 和 `NodeJS`

`Hexo` 是基于 `NodeJS` 编写的，在开始之前我们需要先安装 `NodeJS` 和 `npm` 工具。具体可以网上自己找教程或者参考我如下的步骤

我的本地环境是 `Windows 11 + PowerShell`，包管理工具使用的是 [Scoop](https://scoop.sh/)，`NodeJS` 通过 `nvm` 来管理，如下安装步骤需要在 `PowerShell` 里执行

```powershell
scoop install git # 安装Git

scoop install nvm # 安装 nvm
nvm list available # 查看可用的 NodeJS 版本，这里建议使用 LTS 版本
nvm install 16.18.0 # 安装 NodeJS，我这里安装的是最新 LTS 版本 16.18.0
sudo nvm use 16.18.0 # NodeJS 版本使用 16.18.0，注意这里需要管理员权限，可以先使用 scoop 安装 sudo 这个工具
```

### 一、安装 `Hexo`

{% note info %}
此处只列出本博客所需的关键步骤，更多说明详见官方文档：[https://hexo.io/zh-cn/](https://hexo.io/zh-cn/)
{% endnote %}

#### 1. 全局安装 `hexo-cli` 工具

```bash
npm install -g hexo-cli

hexo -v # 查看版本，目前最新版本为 4.3.0
```

#### 2. 创建一个项目 `my-blog` 并初始化

```bash
hexo init my-blog
cd my-blog
npm install
```

#### 3. 生成网页文件 & 本地启动

```bash
hexo generate # 生成页面，此命令可以简写为 `hexo g`
hexo server # 本地启动，可简写为 `hexo s`
```

{% note success %}
通过 `hexo g` 生成的页面文件在项目 `public` 目录下，通过 `hexo clean` 命令可以清理生成的页面文件，当我们的配置未生效时，建议执行下清理命令
{% endnote %}

#### 4. 本地访问

通过浏览器访问：http://localhost:4000/  可以看到一个比较简陋的页面，没关系，下一章节我们将介绍如何更换主题

### 二、安装&配置主题

通过上面章节所介绍的步骤，我们已经能够通过本地访问博客页面了，但 `Hexo` 默认的主题不太好看，好在官方提供了数百种主题供我们选择，可以根据个人喜好更换，具体可以点击 [这里](https://hexo.io/themes/) 查看，本博客使用的是 [Fluid](https://fluid-dev.github.io/hexo-fluid-docs/) 主题，这里仅介绍此主题的安装与配置。

#### 1. 安装 [Fluid](https://fluid-dev.github.io/hexo-fluid-docs/) 主题

官方提供了两种 [安装方式](https://fluid-dev.github.io/hexo-fluid-docs/start/#%E8%8E%B7%E5%8F%96%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC)，这里使用官方推荐的 `npm` 方式

```bash
npm install --save hexo-theme-fluid
```
然后在博客根路径下创建 `_config.fluid.yml` 文件，并将主题的 `./node_modules/hexo-theme-fluid/_config.yml` 文件内容复制过去

#### 2. 指定主题

将如下修改应用到 `Hexo` 博客目录中的 `_config.yml`:

```yaml
theme: fluid  # 指定主题

language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改
```

#### 3. 创建「关于页」

首次使用主题的「关于页」需要手动创建

```bash
hexo new page about
```

创建成功后修改 `/source/about/index.md`，添加 layout 属性。

修改后的文件示例如下：

```yaml
---
title: 标题
layout: about
---

这里写关于页的正文，支持 Markdown, HTML
```

{% note warning %}
**WARNING**
`layout: about` 必须存在，并且不能修改成其他值，否则不会显示头像等样式。
{% endnote %}

#### 4. 更新 `Fluid` 主题

{% note warning %}
适用于通过 `npm` 安装主题
{% endnote %}

在博客目录下执行命令：

```bash
npm update --save hexo-theme-fluid
```

#### 5. 本地启动

执行如下命令重新生成页面，并启动 `Hexo` 服务

```bash
hexo clean
hexo g
hexo s
```

浏览器访问 http://localhost:4000 , 可以看到页面变得漂亮多了

### 三、创建文章

修改 `_config.yml` 文件，这项配置是为了在生成文章的时候同时生成一个同名的资源目录用于存放图片等资源文件

```yaml
post_asset_folder: true
```

创建文件名为 `my-blog-build-remark` 文章

```bash
hexo new post my-blog-build-remark
```

设置文章的标题及其他元数据信息

```yaml
---
title: 基于 GitHub Pages + Hexo 搭建个人博客
date: 2022-10-16 19:42:53
tags: ['hexo','fluid']
---
```

如上命令执行成功后，在 `source/_posts/` 目录下生成了一个 `Markdown` 文件和一个同名的资源目录

在 `source/_posts/my-blog-build-remark` 目录中放置一个图片文件 `posts-file-tree.png`，整体目录结构如下：

```shell
$ source/_posts (main)> tree
.
├── hello-world.md
├── my-blog-build-remark
│   └── posts-file-tree.png
└── my-blog-build-remark.md
```

然后在文章的 `Markdown` 文件里通过如下方式即可引用对应的图片

```markdown
{% asset_img posts-file-tree.png 目录结构 %}
```

{% note primary %}
关于图片的引用方式，不只有这一种，具体可参考官方文档 https://hexo.io/zh-cn/docs/asset-folders.html ，里边有非常详细的介绍
{% endnote %}

文章创建并编辑好之后，就可以通过 `hexo g && hexo s` 命令来启动服务并在本地预览文章

### 四、配置指南

{% note success %}
如无特殊说明，如下配置文件一律默认为**主题配置**文件：`_config.fluid.yml` 
{% endnote %}

#### 1. 页面 title 修改

修改 `_config.yml` 文件

```yaml
title: Nerd4me's Blog
subtitle: ''
description: ''
keywords:
author: nerd4me
language: zh-CN
timezone: ''
```

#### 2. 博客标题

页面左上角的博客标题，默认使用**站点配置(_config.yml)**中的 `title`，这个配置同时控制着网页在浏览器标签中的标题。
如需单独区别设置，可在**主题配置**中设置

```yaml
navbar:
  blog_title: "Nerd4me's Blog"
```

#### 3. 首页 - Slogan(打字机)

首页大图中的标题文字，可在**主题配置**中设定是否开启，这里支持配置固定的 `text` 或者从远程 `api` 实时获取，优先级 `api > text`

```yaml
index:
  slogan:
    enable: true

    text: "求知若饥，虚心若愚。"
    api:
      enable: true
      url: "https://v1.hitokoto.cn/"
      method: "GET"
      headers: {}
      keys: ["hitokoto"]
```

### 五、网页访问统计

目前 `Fluid` 支持多种统计网站，这里只介绍 `LeanCloud` 的配置，使用 `LeanCloud` 之前，需要先注册账户，具体请访问其 [官网](https://www.leancloud.cn/) 完成账户注册，并新建应用（需实名认证），在 `控制台 -> 应用 -> 设置 -> 应用凭证` 页面中找到对应的 `AppID`、`AppKey`、`REST API 服务器地址` 等信息填入 **主题配置** 中。

```yaml
web_analytics:  # 网页访问统计
  leancloud:
    app_id: # AppID
    app_key: # AppKey
    # REST API 服务器地址，国际版不填
    # Only the Chinese mainland users need to set
    server_url: # REST API 服务器地址

    # 开启后不统计本地路径( localhost 与 127.0.0.1 )
    # If true, ignore localhost & 127.0.0.1
    ignore_local: true
```

{% note warning %}
如无特殊需要，记得配置 `ignore_local: true`，这样 LeanCloud 在 localhost 域名下访问不会增加数据
{% endnote %}

#### 1. 展示 PV 与 UV 统计

页脚可以展示 PV 与 UV 统计数据，目前支持两种数据来源：`LeanCloud` 与 `不蒜子`

```yaml
footer:
  statistics:
    enable: true
    source: "leancloud"
    pv_format: "总访问量 {} 次"
    uv_format: "总访客数 {} 人"
```

#### 2. 展示文章日期/字数/阅读时长/阅读数

```yaml
post:
  meta:
    author: # 作者，优先根据 front-matter 里 author 字段，其次是 hexo 配置中 author 值
      enable: false
    date: # 文章日期，优先根据 front-matter 里 date 字段，其次是 md 文件日期
      enable: true
      format: "LL a" # 格式参照 ISO-8601 日期格式化 See: http://momentjs.cn/docs/#/parsing/string-format/
    wordcount: # 字数统计
      enable: true
      format: "{} 字"
    min2read: # 估计阅读全文需要的时长
      enable: true
      awl: 2
      wpm: 60
      format: "{} 分钟"
    views: # 浏览量计数
      enable: true
      source: "leancloud"
      format: "{} 次"
```

#### 3. 文章评论功能

开启评论需要在**主题配置**中开启并指定评论插件，这里使用基于 `LeanCloud` 的 `Valine`

```yaml
post:
  comments:
    enable: true
    # 指定的插件，需要同时设置对应插件的必要参数
    # Options: utterances | disqus | gitalk | valine | waline | changyan | livere | remark42 | twikoo | cusdis | giscus
    type: valine

# Valine
# 基于 LeanCloud
# See: https://valine.js.org/
valine:
  appId: # LeanCloud AppID
  appKey: # LeanCloud AppKey
```

### 六、发布 GitHub Pages

安装 `hexo-deployer-git`

```bash
npm install hexo-deployer-git --save
```

修改站点配置 `_config.yml`

```yaml
deploy:
  type: git
  repo: <repository url> # https://github.com/nerd4me/nerd4me.github.io.git
  branch: [branch]
  token: [token]
```

生成站点文件并推送至远程 `GitHub` 仓库

```bash
hexo clean
hexo deploy
```

登入 `Github`，请在库设置（Repository Settings）中将默认分支设置为 `_config.yml` 配置中的分支名称。稍等片刻，您的站点就会显示在您的Github Pages中。


### 七、参考资料

- [Hexo Docs](https://hexo.io/zh-cn/docs/)
- [Hexo Fluid 用户手册](https://fluid-dev.github.io/hexo-fluid-docs/)