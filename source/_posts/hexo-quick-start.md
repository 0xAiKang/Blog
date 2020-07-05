---
title: Hexo 快速上手
date: 2020-07-05 14:16:31
tags: ["hexo", "tutorial"]
categories: tutorial
---
# 前言
最近使用Hexo 搭建了一套博客系统，整个过程还算顺利，不过还是遇到了一些问题，整理记录一下。

## 常用命令

### init
新建一个网站。如果没有设置 `folder`，Hexo 默认在目前的文件夹建立网站。
```
$ hexo init [folder]
```

### new
layout 有三种选择：
* post：新建一片文章
* page：新建一个页面
* draft：新建一篇草稿

如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。
```
$ hexo new [layout] <title>
```

### generate
生成静态文件。
```
$ hexo generate
// 等效于 hexo g
```
常用参数：
|选项|描述|
|-|-|
|-d, --deploy|文件生成后立即部署网站|
|-w, --watch|监视文件变动|
|-b, --bail|生成过程中如果发生任何未处理的异常则抛出异常|

### publish
发表草稿
```
$ hexo publish [layout] <filename>
```

### server
启动服务器。默认情况下，访问网址为： `http://localhost:4000/`。
```
$ hexo server
// 等效于 hexo s
```

### deploy
部署网站。
```
$ hexo deploy
// 等效于 hexo d
```

`-g`，`--generate`：部署之前预先生成静态文件

### clean
清除缓存文件 (db.json) 和已生成的静态文件 (public)。

在某些情况（尤其是更换主题后），如果发现对站点的更改无论如何也不生效，那可能需要运行该命令。
```
$ hexo clean
```

### list
列出网站资料。
```
$ hexo list
```

### version
显示 Hexo 版本。
```
$ hexo version
```

### 其他模式
#### 安全模式
在安全模式下，不会载入插件和脚本。当需要安装新插件遭遇问题时，可以尝试以安全模式重新执行。
```
$ hexo --safe
```

#### 调试模式
在终端中显示调试信息并记录到 debug.log。
```
$ hexo --debug
```

#### 显示草稿
显示 source/_drafts 文件夹中的草稿文章。
```
$ hexo --draft
```

## 常见问题

### CNAME 文件被删除

GitHub Pages 为我们免费提供了`<username>.github.io`这样的域名作为 GitHub Page，但如果你觉得这个域名太长了，不满意，那么你也可以绑定自己的域名。

通常绑定完成之后，会在项目目录下面生成一个叫做`CNAME`的文件，这个文件的作用就是用来记录GitHub Pages 所绑定的域名。

这个时候就会产生一个问题：
> CNAME文件会在每次 hexo deploy 时消失，然后需要重新手动绑定，这样就很繁琐。

有以下几种方式可以解决这个问题：
1. 每次 `hexo d` 之后，就去 GitHub 仓库根目录新建 CNAME文件。—— 繁琐
2. 在 `hexo g` 之后， `hexo d` 之前，把CNAME文件复制到 `public` 目录下面，里面写入你要绑定的域名。—— 繁琐
3. 将需要上传至 GitHub 的内容放在`source`文件夹，例如CNAME、favicon.ico、images等，这样在 `hexo d` 之后就不会被删除了。
4. 通过安装插件实现永久保留。

```
$ npm install hexo-generator-cname --save
```

编辑`_config.yml`
```
Plugins:
- hexo-generator-cname
```
推荐第三种方式，简单方便。

### 参考链接
* [github+hexo搭建自己的博客网站（七）注意事项](https://www.cnblogs.com/chengxs/p/7496265.html)
* [Hexo | 指令](https://hexo.io/zh-cn/docs/commands)