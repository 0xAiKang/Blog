---
title: Github Pages 部署 Hexo 个人博客
date: 2020-07-04 20:09:09
tags: Tutorial
categories: Tutorial
---

关于个人博客，在很久之前就想自己搭建一套，甚至还为此买了一台服务器，但奈何自己太忙了(tai lan le) =_=，这件事情就一直搁浅了，服务器大部分时间也都是空闲状态。

这段时间，突然很想把这件事情做好，觉得不能在这么拖下去了，所以便有了这篇文章。

<!-- more -->

> 为什么使用Github Pages？

我是出于以下原因考虑的：
1. 暂时没有服务器的需要，我只想有一个能写博客的地方。
2. GitHub Pages 可以提供 https服务，我不用担心域名备案的问题。
3. 免费

总之，如果你想用最简单、最省心的方式，搭建属于自己的博客，那么 Github Pages 一定不会让你失望。
## 系统环境
* Mac OS 10.15.4
* Node.js 12
* Hexo-cli: 3.1
* NPM: 6.9

### 创建Github Pages
Github Pages分为两类，用户或组织主页、项目主页。

* 用户或组织主页：在新建仓库时，仓库名称应该以`<yourusername>.github.io`的格式去填写。`<yourusername>`指的是你的Github 的用户名称。
* 创建项目主页：在新建仓库时，名称可以任意设置，然后通过`Setting->Options->Github Pages`将 `Source`选项设置为`Master Branch`，此时这个项目就变成一个 Github Pages项目了。

需要注意的是：
1. Github Pages 只针对开源的项目是免费的，如果你不想开源，那可能就需要考虑收费的套餐了。
2. 第一种方式不能更改 Github Pages 部署分支。
3. 如果你有自己的域名，那么推荐使用方式二创建 Github Pages。如果你没有自己的域名，那也没有关系，可以使用Github Pages 提供的域名访问`http://<yourusername>.github.io`。

### 绑定域名
如果你是通过方式一，创建的Github Pages，那么可以跳过此部分。

在 2018 年 5 月 1 日之后，GitHub Pages 已经开始提供免费为自定义域名开启 HTTPS 的功能，并且大大简化了操作的流程，现在用户已经不再需要自己提供证书，只需要将自己的域名使用 CNAME 的方式指向自己的 GitHub Pages 域名即可。

首先需要在你的 DNS 解析里添加一条解析记录，例如我选择添加子域名`blog.aikang.me`，通过 CNAME 的方式指向我刚刚自定义的 GitHub Pages 域名 `0xAiKang.github.io`。

![DNS 域名解析](https://i.loli.net/2020/07/04/BDX384QPIZqniJU.png)

添加完成后等待 DNS 解析的生效的同时回到项目的`Setting`界面，将刚才的子域名与 Github Pages 绑定在一起。

保存之后，我们只需要耐心等待 GitHub 生成证书并确认域名的解析是否正常。

![等待 GitHub 生成证书并确认域名解析正常](https://i.loli.net/2020/07/04/OZ2Vu8p9tXgTj7q.png)

### 将Hexo 部署到Github Pages
域名解析成功之后，就可以通过我们刚才绑定的域名进行访问了，但是你会发现，现在只能看到一片空白，这是因为我们的网站还没有任何内容，所以下一步需要做的就是选择一套静态模版系统。

目前市场上有很多优秀的静态模板系统，比如：
* Node.js 编写的 Hexo
* Go 编写的 Hugo
* Python 编写的 Pelican
* 静态博客写作客户端 Gridea

> 为什么要选择Hexo？

最初在选择博客模版系统时，并没有发现 Gridea ，事后发现这个小众的静态博客写作客户端似乎才是我真正想要的。

不过既然选择了Hexo，也是因为它的生态环境很大，可选主题非常多，并且都是开源的。

> 如何将 Hexo 部署到 GitHub Pages？

1. 将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 GitHub 账户中。
2. 前往 GitHub 的 [Applications settings](https://github.com/settings/installations)，配置 Travis CI 权限，使其能够访问你的 repository。
3. 正常情况下你会被重定向到 Travis CI 的页面。如果没有，请 [手动前往](https://travis-ci.com/)。
4. 前往 GitHub 新建 [Personal Access Token](https://github.com/settings/tokens)，只勾选 repo 的权限并生成一个新的 Token。Token 生成后请复制并保存好。
5. 回到 Travis CI，前往你的 repository 的设置页面，在 Environment Variables 下新建一个环境变量，Name 为 GH_TOKEN，Value 为刚才你在 GitHub 生成的 Token。确保 DISPLAY VALUE IN BUILD LOG 保持 不被勾选 避免你的 Token 泄漏。点击 Add 保存。
6. 在你的 Hexo 站点文件夹中新建一个 `.travis.yml` 文件：

```
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
```
上面这个配置文件的作用是用来自动构建，编译测试。

将 `.travis.yml` 推送到 repository 中。Travis CI 会自动开始运行，并将生成的文件推送到同一 repository 下的 `gh-pages` 分支下。

#### 修改发布源
推送完成之后，会发现多了一个 `gh-gages`分支，这个分支就是用于部署站点的分支，但是GitHub Pages 会默认使用`master`分支作为发布源，所以我们需要切换发布源。

在`Setting->Option->GitHub Pages`下，使用 Source（源）下拉菜单选择发布源。

![修改默认源](https://i.loli.net/2020/07/04/AHldtP2bIhaqr8c.png)

注意：使用用户或组织主页构建的 Github Pages 不能修改发布源，只能使用默认的 `master`分支。

### 一键部署
Hexo 提供了快速方便的一键部署功能，让你只需一条命令就能将网站部署到服务器上。

在正式部署之前，我们需要先修改`_config.yml` 文件，配置参数。

```
deploy:
  type: git
  repo: <repository url> #https://bitbucket.org/JohnSmith/johnsmith.bitbucket.io
  branch: [branch]
  message: [message]
```
|参数|描述|默认值|
|-|-|-|
|type|deployer|-|
|repo|项目地址|-|
|branch|分支名称|gh-pages|

有以下两点需要注意：
1.repo 需要选择SSH 协议，HTTPS协议会报错。
2.branch 选择Github Pages中设置的那个分支，而不是拉取这个项目的分支

我这里使用的是`git` 作为 deployer，所以需要手动安装一个插件。

```
npm install hexo-deployer-git --save
```

生成站点文件并部署至远程库：
```
hexo clean && hexo deploy --generate
```

至此，就完成了使用Github Pages 部署 Hexo 个人博客的全部过程，总的来说还是很顺利的。

### 参考链接
* [Github Pages 搭建教程](https://sspai.com/post/54608)
* [将Hexo 部署到 GitHub Pages](https://hexo.io/zh-cn/docs/github-pages.html)
* [Hexo 一键部署](https://hexo.io/zh-cn/docs/one-command-deployment.html)
* [Github Pages部署个人博客（Hexo篇）](https://juejin.im/post/5acf02086fb9a028b92d8652#heading-15)