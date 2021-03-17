---
title: Jenkins + Gitlab 持续集成
date: 2021-03-16 22:27:42
tags: ["运维", "Jenkins", "CI", "CD"]
categories: ["运维", "Jenkins"]
---

> Jenkins 是什么？

Jenkins是一个开源的、提供友好操作界面的持续集成(CI)工具。

<!-- more -->

> Jenkins 如何与Gitlab 进行关联？

可以通过生成密钥（Webhooks 的钩子），然后到Gitlab 需要集成的项目中，设置集成功能，增加Web 钩子。

这样当进行Push 动作时，就会触发Jenkins 进行构建，然后执行相应的流水线。

对于小公司而言，开发服务器常用的架构是内网服务器（本地机器）+外网服务器内网穿透，Jenkins + 私有Gitlab 持续集成。

背景：Jenkins 和Gitlab 部署在外网服务器上，通过内网穿透对内网服务器（开发服务器）进行访问。
需求描述：每次进行Push 时，触发Jenkins 流水线，进行构建，将最新的版本同步到开发服务器上。

Jenkins的功能很强大，这里并不打算深入拓展，而是介绍一种相较简单粗暴的方式去完成持续集成。

## 创建任务

首先需要在Jenkins 上新建任务，因为需求并不复杂，这里直接选择流水线的方式

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210316144137.png)

如果需要关联TAPD，这里需要「关联TAPD」填上对应TAPD 的ID。

核心的配置在构建触发器这一块，根据Push 事件，触发执行流水线。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210316144851.png)

有以下几个点需要注意：
1. 因为是开发服务器，并没有开启合并请求。
2. Gitlab webhook URL 需要记住，后面会用到。
3. 默认允许所有分支，如果有特殊需求，可以指定分支名进行过滤。
4. 点击右下角的Generate 按钮生成Secret token，后面会用到。

配置流水线：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210316220105.png)

上半部分是连接内网服务器（开发服务器）的基础信息，下半的配置信息是需要执行的构建脚本。

构建脚本的作用其实就是去执行`git pull` 这个动作，大概长这样：

```
#!/usr/bin/bash

pull() {
    cd /var/www/project && git pull
}

pull 
```

<details>
<summary>流水线配置，点击查看详情信息</summary>
<pre>
def bdService() {
    def remote = [:]
    remote.name = 'hostname'
    remote.host = 'localhost'
    remote.port = 22
    remote.user = 'username'
    remote.password = 'password'
    remote.allowAnyHosts = true
    return remote
}
pipeline {
    agent any
    stages {
        stage('代码集成') {
            steps {
                script {
                    def  remote = bdService();
                    sshCommand remote: remote, command: "/bin/bash /opt/shell/build.sh"
                }
            }
        }
    }
}
</details>

配置完成之后，点击保存。

返回工作态，找到对应任务，点击立即构建。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210316220845.png)

通过构建历史，查看`Console Output`，能看到类似输出则表示构建成功。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210316220912.png)

构建成功之后，就可以与Gitlab 进行关联了，点击项目=>设置=>集成。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210316220941.png)

链接（URL）是之前的 Webhook url，安全令牌则是上面生成的 Secret token，SSL 证书验证视情况选择是否开启，然后点击增加Web 钩子。

至此所有的配置就基本完成了，这时可以去测试Push，看看是否会执行自动构建。