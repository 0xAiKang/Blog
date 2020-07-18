---
title: how to put console.log the standard output of is recorded to a file
date: 2020-07-18 16:16:51
tags: ["Node"]
categories: ["一些经验"]
---

最近遇到了这样一个需求，在不改动之前的任何一行代码的前提下，如何把`console.log`的标准输出全部记录到文件中呢？

<!-- more -->

我是没有选择那些大名鼎鼎的日志模块，如：
* [winston](https://github.com/winstonjs/winston) - A logger for just about everything.
* [log4js](https://github.com/log4js-node/log4js-node) - A port of log4js to node.js

因为我的需求够简单，只需要能把日志记录到文件就行，所以使用了下面这种最简单的方式：
```
var log_file = fs.createWriteStream(path.resolve(__dirname, ".pm2") + '/debug.log', {flags : 'w'});
var log_stdout = process.stdout;

/**
 * 重载console.log 函数
 */
console.log = function() {
    var res = "",
    len = arguments.length;
    for(var i=0; i<len; i++){
        res += arguments[i];
    }

    log_file.write(util.format(res) + '\n');
    log_stdout.write(util.format(res) + '\n');
};
```

### 参考链接
* [Node: log in a file instead of the console](https://stackoverflow.com/questions/8393636/node-log-in-a-file-instead-of-the-console)
