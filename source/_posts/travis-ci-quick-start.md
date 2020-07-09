---
title: Travis CI 快速上手
date: 2020-07-05 14:25:58
tags: ["CI", "Tutorial"]
categories: Tutorial
---

最近使用Github Pages 搭建Hexo 时，用到了一项新技术。hmm...也不能说是新技术吧，只是之前一直有听说，但却没有实际用过。

它就是持续集成，听上去好像是一个高大上的概念，但通俗一点解释就是：写完代码提交之后，会根据你的要求，自动做编译测试。

其中最出名大概就是[Travis CI](https://travis-ci.com/)了，本文的目的就是快速入门 Travis CI。

<!-- more -->

## 什么是持续集成？
持续集成(Continuous Integration)是对小周期的的代码进行更改，其目的是通过以较小的增量开发和测试来构建更健康的软件。

而Travis CI 作为一个持续集成平台，通过自动构建和测试代码，并提供更改成功的即时反馈。

### 快速上手
在正式开始之前，需要提前准备好以下先决条件：
* 一个 [GitHub](https://github.com/) 帐户
* 托管在 [Github](https://github.com/) 的项目的所有者权限

需要注意的是：Travis CI不是完全免费的服务，前100个私有构建是免费的，后续就要进行付费，如果你的项目是开源的，或者你是学生，则不受限制。

#### 在Github 上使用Travis CI
1. 将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 GitHub 账户中。
2. 前往 GitHub 的 [Applications settings](https://github.com/settings/installations)，配置 Travis CI 权限，使其能够访问你的 repository。
3. 前往 GitHub 新建 [Personal Access Token](https://github.com/settings/tokens)，只勾选 repo 的权限并生成一个新的 Token。Token 生成后请复制并保存好。
4. 回到 Travis CI，前往你的 repository 的设置页面，在 Environment Variables 下新建一个环境变量，Name 为 GH_TOKEN，Value 为刚才你在 GitHub 生成的 Token。确保 DISPLAY VALUE IN BUILD LOG 保持 不被勾选 避免你的 Token 泄漏。点击 Add 保存。
5. 在你的项目中新建一个 `.travis.yml` 文件。
6. 提交并推送以触发Travis CI构建。

其中`.travis.yml`文件的目的是告诉 Travis CI 应该做些什么。

以下示例指定了应使用Ruby 2.2和最新版本的JRuby构建的Ruby项目。
```
language: ruby
rvm:
 - 2.2
 - jruby
```

通过访问[Travis CI](https://travis-ci.com/auth) 并选择repository，检查构建状态页面，以根据构建命令的返回状态查看构建是否通过或失败。