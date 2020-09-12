---
title: 什么是DevOps、CI、CD、K8S
date: 2020-09-12 22:40:00
tags: ["DevOps", "K8S"]
categories: ["Docker"]
---

之所以要写这片笔记，是因为前几天在使用 gitlab 提交代码时，遇到了点问题。

gitlab 提示我 commit 失败。跟进了一下，并没有找到答案。

![image](https://note.youdao.com/yws/res/77963/7FE5E992EF4342039DD24A95068C11F5)

只是了解到一个叫做 `CI/CD`的东西。后来又延伸扩展到`DevOps`、`K8S` 这些新概念。

![image](https://note.youdao.com/yws/res/77952/98C84A2F89DC42378E046A896A815AC1)

## 什么是 DevOps？
如题，什么是 DevOps ？根据字面意思理解就是：`Dev` + `Ops`，开发（Development）和运营（Operations）这两个领域的合并。

就我个人的理解，它是一个概念、一种思维，是一种通力合作，共同解决问题的方式。

这里我就不追根溯源去解释为什么要合并开发和运营了，因为历史原因，总是存在着这样的问题。具体看参考链接一。

![image](https://note.youdao.com/yws/res/78012/F4EE3FB3314241BC886F8D1CE425922E)

DevOps 也不仅仅是一种软件的部署方法。它通过一种全新的方式，来思考如何让软件的作者（开发部门）和运营者（运营部门）进行合作与协同。使用了DevOps模型之后，会使两个部门更好的交互。
其中，`自动化部署`的概念就是从中产生的。

## 什么是 CI/CD？
Gitlab 的`CI/CD`到底是什么呢？

昨天大致了解了下 `Gitlab CI/CD`，不是很明白，但觉得很厉害。
首先来看下官方文档的简介：
> 软件开发的连续方法基于自动执行脚本，以最大限度地减少在开发应用程序时引入错误的可能性。从新代码的开发到部署，它们需要较少的人为干预甚至根本不需要干预。
它涉及在每次小迭代中不断构建，测试和部署代码更改，从而减少基于有缺陷或失败的先前版本开发新代码的机会。

这里有三种主要的方法，根据最适合你的策略进行选择。

### 持续集成
考虑一个应用程序，其代码存储在Gitlab中的存储库中。开发人员每天多次推送代码更改，对于每次推动到存储库，都可以创建一组脚本来自动构建和测试应用程序，从而减少向应用程序引入错误的可能性。这种方法被称为：**持续集成（Continuous Integration）**

### 持续交付
**持续交付 Continuous Delivery**是持续集成的一个步骤，应用程序不仅在推送到代码库的每个代码更改时都构建和测试，而且作为一个额外的步骤，它也会连续部署，尽管部署是手动触发的。

### 持续部署
**持续部署 Continuous Deployment**也是持续集成的又一步，类似于持续交付。不同之处在于，不必手动部署应用程序，而是将其设置为自动部署。完全不需要人工干预就可以部署应用程序。

## 什么是 K8S？

## 参考链接
* [什么是DevOps？--程序员小灰](https://blog.csdn.net/bjweimengshu/article/details/79031552)
* [DevOps 到底是什么？](https://www.cnblogs.com/servicehot/p/6510199.html)
* [使用GitLab介绍CI / CD](https://docs.gitlab.com/ee/ci/introduction/index.html)
* [Gitlab CI/CD 快速入门](http://www.ttlsa.com/auto/gitlab-cicd-quick-start/)
* [Gitlab CI/CD 入门](https://docs.gitlab.com/ee/ci/quick_start/README.html)
* [Gitlab CI 示例](https://docs.gitlab.com/ee/ci/README.html)
* [Docker 集成](https://docs.gitlab.com/ee/ci/docker/README.html)
* [Git Runner 是什么？](https://docs.gitlab.com/runner/)
* [安装Gitlab Runner](https://docs.gitlab.com/runner/install/)
* [什么是 Auto DevOps](https://docs.gitlab.com/ee/topics/autodevops/index.html)
* [K8S 是什么？知乎](https://zhuanlan.zhihu.com/p/29232090)
* [为什么 K8S 很酷](https://zhuanlan.zhihu.com/p/33640916)
* [K8S 中文指南](https://github.com/rootsongjc/kubernetes-handbook)
* [一文了解 K8S 是什么？](https://zhuanlan.zhihu.com/p/28810342)
