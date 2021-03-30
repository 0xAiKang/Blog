---
title: 『转载』Laravel 中大型项目架构
date: 2021-03-30 23:59:36
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

初学者学习 Laravel 时分两种，一种是乖乖的将程式填入 MVC 架构內，导致 controller 与 model 异常的肥大，日后一样很难维护；一种是常常不知道程式改写在哪一个 class 內而犹豫不決，毕竟传统 PHP 都是一个页面一个档案。本文整理出适合 Laravel 的中大型项目架构，兼具容易维护、容易扩充与容易重复使用的特点，并且容易测试。

<!-- more -->

一个项目只有 MVC 是不够的，我们需要更完整的项目架构。
 
 ## Controller 过于臃肿
受RoR的影响，初学者常认为 MVC 架构就是 model ,view,controller :

* Model 就是资料库。
* Controller 负责与 HTTP 交互，调用 model 与 view。
* View 就是 HTML。

假如依照这个定义，以下这些需求改写在哪里呢？
1. 发送 Email，使用外部 API。
2. 使用 PHP 写的逻辑。
3. 依需求将显示格式作转换。
4. 依需求是否显示某些资料。
5. 依需求显示不同资料。

其中 1, 2 属于商业逻辑，而 3, 4, 5 属于显示逻辑，若依照一般人对 MVC 的定义，model 是资料库，而 view 又是 HTML，以上这些需求都不能写在 model 与 view，只能勉强写在 controller。

因此初学者开始将大量程式写在 controller，造成 controller 的肥大难以维护。

## Model 过于臃肿
既然逻辑写在 controller 不方便维护，那我将逻辑都写在 model 就好了？

当你将逻辑从 controller 搬到 model 后，虽然 controller 变瘦了，但却肥了 model，model 从原本代表资料库，現在变成还要负责商业逻辑与显示逻辑，结果更慘。

Model 代表资料库吗？把它想成是 Eloquent class就好，资料库逻辑应该写在 repository 里，这也是为什么 Laravel 5 已经沒有 models目录，Eloquent class 仅仅是放在 app 根目录下而已。

## 中大型项目架构
那我们改怎么写呢？別将我们的思维局限在 MVC 內 :

1. Model : 仅当成 Eloquent class。
2. Repository : 辅助 model，处理资料库逻辑，然后注入到 service。
3. Service : 辅助 controller，处理业务逻辑，然后注入到 controller。
4. Controller : 接收 HTTP request，调用其他 service。
5. Presenter : 处理显示逻辑，然后注入到 view。
6. View : 使用 blade 将资料 绑定 到 HTML。

上面架构我们可以发现 MVC 架构还在，由与 SOLID 的单一职责原則与依赖反转原则:

我们将资料库逻辑从 model 分离出来，由 repository 辅助 model，将 model 依赖注入进 repository。
我们将商业逻辑从 controller 分离出来，由 service 辅助 controller，将 service 依赖注入进 controller。
我們将显示逻辑从 view 分离出來，由 presenter 辅助 view，将 presenter 依赖注入进 view。

## 建立目录

在 app 目录下建立 Repositories，Services 与 Presenters 目录。

 別害怕建立目录！！

別害怕在 Laravel 预设目录以外建立的其他目录，根据 SOLID 的单一职责原则，class 功能越多，责任也越多，因此越违反单一职责原则，所以你应该将你的程式分割成更小的部分，每个部分都有它专属的功能，而不是一个 class 功能包山包海，也就是所谓的万能类别，所以整个方案不应该只有 MVC 三个部分，放手根据你的需求建立适当的目录，并将适当的 class 放到该目录下，只要我们的 class 有 namespace 帮我们分类即可。

## Repository

由于篇幅的关系，将 repository 独立成专文讨论，请参考如何使用 Repository 模式?

## Service

由于篇幅的关系，将 service 独立成专文讨论，请参考如何使用 Service 模式?

## Presenter

由于篇幅的关系，将 presenter 独立成专文讨论，请参考如何使用 Presenter 模式?

## 单元测试

由于现在 model、view、controller 的相依物件都已经拆开，也都使用依赖注入，因此每个部分都可以单独的做单元测试，如要测试 service，就将 repository 加以 mock，也可以将其他 service 加以 mock。

Presenter 也可以单独跑单元测试，将其他 service 加以 mock，不一定要跑验收测试才能测试显示逻辑。

## Conclusion
本文谈到的架构只是开始开始，你可以依照实际需求增加更多的目录与 class，当你发现你的 MVC 违反 SOLID 原则时，就大胆的将 class 从 MVC 拆开重构，然后依照以下手法 :

1. 建立新的 class 或 interface。
2. 将相依物件依赖注入到 class。
3. 在 class 內处理他的职责。
4. 将 class 或 interface 注入到 controller 或 view。
 
————————————————
版权声明：本文为CSDN博主「华尔街之猫」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_24935119/article/details/89656569

