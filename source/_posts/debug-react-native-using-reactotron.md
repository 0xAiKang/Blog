---
title: 使用 Reactotron 调试 React Native
date: 2023-11-18 17:21:16
tags: ["React Native"]
categories: ["React Native"]
---

使用 Simulator 开发 React Native 应用时，在 Debug 模式下，虽然能 Console 调试，但是不能像 Web 开发一样，在控制台的 Network 看到网络请求，因此开发调试问题时，很不方便。

[Reactotron](https://github.com/infinitered/reactotron) 是一个用于 [React JS](https://facebook.github.io/react/) 和 [React Native](https://facebook.github.io/react-native/) App 的网络请求调试工具，就可以很好的实现该功能。

<!-- more -->

### 安装

首先需要[下载](https://github.com/infinitered/reactotron/releases)安装 Reactotron，支持Linux、Windows、Mac。

直接解压运行：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231118165745.png)

Reactotron 默认监听的是 `9090` 端口。

### 配置

在项目中，安装 Reactotron Dev 依赖：

```bash
$ npm i --save-dev reactotron-react-native
```

在项目目录下创建 `ReactotronConfig.js` 文件，并且将以下代码粘贴进去：
```js
import Reactotron from 'reactotron-react-native'

Reactotron
  .configure()      // controls connection & communication settings
  .useReactNative() // add all built-in react native plugins
  .connect()        // let's connect!
```

最后，在项目启动的 `index.js` 或 `App.js`  中，导入：

```js
import './ReactotronConfig'
```

这一步很重要，如果没有导入，Reactotron 是发现不了应用的。

### 运行

打开 Reactotron，并保持运行，然后使用 `npm start` 启动项目，正常情况下，应该可以看到已连接状态：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231118170700.png)

如果没有看到，可以看到以下，`9090` 端口是否被占用？

如果本地开着 Clash，端口有可能会被占用，因为 Clash 默认的外部控制端口也是 `9090`。

注意💡：如果是用真机调试，需要使用以下命令开启 9090 端口连接：
```bash
adb reverse tcp:9090 tcp:9090
```

如果一切顺利，就可以看到 Timeline 是有记录的：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20231118171558.png)