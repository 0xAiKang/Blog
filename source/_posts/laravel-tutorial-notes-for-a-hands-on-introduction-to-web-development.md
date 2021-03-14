---
title: L01 Laravel 教程- Web 开发实战入门课程笔记
date: 2021-03-14 12:10:01
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

[L01 Laravel 教程- Web 开发实战入门](https://learnku.com/courses/laravel-essential-training/8.x)课程笔记。

<!-- more -->

## 第一章
构建应用（8.*）：
```
$ composer create-project laravel/laravel weibo --prefer-dist "8.*"
```

构建应用（5.*）：
```
$ composer create-project laravel/laravel Laravel --prefer-dist "5.7.*"
```

Ubuntu 中查看所有PHP 版本：
```
$ update-alternatives --display php
```

Ubuntu 中快速切换PHP 版本：
```
$ sudo update-alternatives --config php
```

工作原理：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20210314120650.png)

注意事项：
1. 路由的服务提供者类中设置命名空间
2. `blade.php` 是Laravel 的一套模版引擎，有自己的一套规则，通过继承父视图，可以减少很多重复代码
3. `art tinker` 是 Laravel 框架自带的命令，用以调出 Laravel 的交互式运行时

## artisan 命令
[Artisan](https://learnku.com/docs/laravel/6.x/artisan) 是 Laravel 提供的 CLI（命令行接口）。

常用命令如下：
|命令|说明|
|-|-|
|php artisan key:generate|生成App Key|
|php artisan make:controller|生成控制器|
|php artisan make:model|生成模型|
|php artisan make:policy|生成授权策略|
|php artisan make:seeder|生成Seeder 文件|
|php artisan migrate|执行迁移|
|php artisan migrate:rollback|回滚迁移|
|php artisan migrate:refresh|重置数据库|
|php artisan db:seed|填充数据库|
|php artisan migrate:refresh --seed|进行数据库迁移同时填充数据库|
|php artisan tinker|进入tinker 环境|
|php artisan route:list|查看路由列表|

## 第一章学到了什么
* 如何构建一个Laravel 应用
* 对新建的Laravel 项目进行基本配置
* 手动创建控制器、静态视图
* 了解路由、控制器、视图的基本协作方式
* 了解如何使用通用视图
* 了解Artisan 命令的基本使用

## 第二章

```
$ composer require laravel/ui:^3.0 --dev
```

`composer require` 命令是用来安装扩展包的命令，参数`--dev`表示仅仅只在开发环境中使用。

上面命令安装完成之后，使用以下命令来引入 bootstrap：
```
$ php artisan ui bootstrap
```

建议使用`yarn`命令代替`npm` 命令：
```
$ yarn install
```

前端代码编译出现问题时，可以尝试将`node_moudles`文件夹删除，再次安装相关依赖。

```
$ npm run watch-poll
```
上面这条命令的作用其实就是监视前端文件变化，如果有变化的话，就马上编译。

> 生成环境中，为什么不用一直开启这个命令？

这是因为服务端只需要手动编译一次就好，也就是使用`npm run dev`命令。

之所以说Laravel 是全栈框架，就是因为它在一个项目中把前端和后端所有东西都包揽了。

### Laravel 的前端工作流
Laravel 的前端工作流是通过 Sass、NPM、Yarn、Laravel Mix 构成一套前端工作流。

* `Sass` 是一种可用于编写CSS 的语言。
* `Yarn` 是一个用于代替NPM 客户端的新的包管理器。
* `Laravel Mix` 是一个前端任务自动化管理工具。`Laravel Mix` 可以自动编译`resources`下面的文件。
* `{{ }}` 是在Html 中内嵌PHP 的Blade 语法，表示包含在该区域内的代码使用PHP 来编译执行。

### 第二章学到了什么
1. Laravel 的前端工作流
2. 局部视图的订单和引用
3. 命名路由的定义和使用

## 第三章
### Eloquent ORM
其特点是一个模型对应数据库中的一个表。

在进行数据库迁移时，`up` 方法会被调用，在进行数据库回滚时，`down`方法会被调用。

所以无论是初次创建表，还是后面增加字段，都需要去`up` 方法下进行定义，这样在数据库迁移时才会生效。

创建一张表：
```
$ php artisan make:migration create_followers_table --create="followers"
```

增加一个字段：
```
$ php artisan make:migration add_activation_to_users_table --table=users
```

数据库的回滚与迁移直接对应着 `databases` 文件夹下的迁移文件。

创建模型的同时并进行迁移：
```
$ php artisan make:model Model/Articles -m 
```

可以使用以下命令进行数据库交互：
```
$ php artisan tinker
```
该命令可以进入Eloquent 模型，直接进行数据库交互。在该模式下，Eloquent 模型的方法均可以使用。

### 第三章学到了什么？
1. Eloquent 模型的定义与应用
2. 数据库迁移与回滚（数据表生成与删除）
3. 模型的创建与使用
4. tinker 的使用

## 第四章
在本地可以这样访问Homestead 的数据库：
```
$ mysql -uhomestead -h127.0.0.1 -P33060 -p // 或者 mysql -uhomestead -h192.168.10.10 -P3306 -p
```

但是在项目（Homestead）中，只能这样访问：
```
$ mysql -uhomestead -h127.0.0.1 -P3360 -p // 或者 mysql -uhomestead -h192.168.10.10 -P3306 -p
```
因为端口做了映射（Homestead 3306=> 主机 33060），而项目又运行在Homestead 中，所以项目配置中的端口不能写成`33060`，否则无法正常访问。

### 隐形路由绑定
这个『隐形路由绑定』倒底是个啥玩意？简单理解就是通过控制器把模型绑定在路由中了。

路由代码：
```
Route::get('/users/{user}', 'UsersController@show')->name('users.show');
```

控制器及模型代码：
```
use App\Models\User;

public function show (User $user){
  return view("users.show", compact("user")); 
}
```
上面这段代码有很多知识点：
1. 控制器方法`show` 是通过路由获取的
2. `User $user` 是定义在控制器中的方法的Eloquent 模型类型声明
3. 由于show 方法传参时声明了类型——Eloquent 模型，对应的变量名`$user` 会匹配路由片段中的`{user}`，这样Laravel 会自动注入与请求URL 传入的ID 对应的用户模型实例。

这里利用了隐形路由绑定，直接读取对应ID 的用户的实例。

其实这个和ThinkPHP 中的路由传参很像，只不过不同的是ThinkPHP 中没有定义模型。

```
Route::get('hello/:name', 'index/hello');

public function hello($name){
  return $name;  
}
```

Laravel 是如何接收前端的参数的？
```
public function sotre(Request $request){
  // 通过使用Illuminate\Http\Request 实例来接收用户输入
}
```

### 第四章学到了什么
1. 使用RESTFUL 来构建路由资源
2. 通过表单与控制器协同处理数据
3. 验证表单提交的数据，并返回相应的内容
4. 利用Composer 安装相应扩展包
5. 使用闪存来展示用户信息

## 第五章
Laravel 提供了`attempt` 方法用于登录验证。
```
if (Auth::attempt(['email' => $email, 'password' => $password])) {
    // 该用户存在于数据库，且邮箱和密码相符合
}
```

`attempt` 方法接收一个数组作为第一个参数，会去数据库中找寻对应的值，逻辑如下：
1. 找寻`email`字段匹配的值
2. 如果没找到，直接返回false
3. 如果能找到：
  i. 先将传参password进行加密，与数据库中的值进行比对
  ii. 如果两个值匹配，会创建一个会话给验证通过的用户，在会话创建的同时，也会种下一个名为 laravel_session 的 HTTP Cookie，以此 Cookie 来记录用户登录状态，最终返回 true
  iii. 如果不匹配，返回false

登录成功之后，可以使用`Auth::user()` 获取用户信息。

```
Route::get('login', 'SessionsController@create')->name('login'); 
```
通过`name()` 方法定义路由名称，这样需要访问该路由时，直接访问该名称就好。

### 第五章学到了什么
1. Auth 认证的使用
2. 了解Laravel 常用登录机制的具体实现
3. 集成Bootstrap Javascript 组件
4. 通过 “记住我” 来记住用户登录状态

## 第六章
通常开发编辑用户时，需要先从数据库中获取到该用户当前的信息，然后再进行编辑。

在Laravel 中，只需要几行代码就可以完成这件事情。
```
public function edit(User $user){
  return view("users.edit", compact("user"))
}
```
这里通过『隐形路由绑定』，把对应ID 的用户的实例作为控制器参数。

### 中间件访问限制
有时我们会希望未登录的用户，不能访问某些功能，这时可以通过 Auth 提供的中间很方便的完成。

```
$this->middleware("auth", [
    // 指定这几个方法不使用Auth 去验证
    "except" => ["show", "create", "store", "index", "confirmEmail"]
]);
```

### 授权验证
但需要注意的时，这里仅仅限制的是未登录，而有些功能则是需要在登录状态下进行限制，比如：ID 为1 的用户不能修改ID 为2 的用户的信息。

这个就不是中间件职责范围内能做的事情了，这个需要授权策略来完成。
```
// 定义策略
public function update(User $currentUser, User $user)
{
    return $currentUser->id === $user->id;
}
```

还需要在控制器中验证才算正真使用了。
```
// 使用策略
public function update(User $user)
{
    $this->authorize("update", $user);
    // ... 
}
```

### 数据填充
数据填充需要使用Seeder 类，如果需要进行数据填充，需要调用 Seeder 的call 方法。

以用户模型为例，填充步骤如下：
1. 首先创建用户工厂，定义填充数据
```
$ php artisan make:factory UserFactroy
```

2. 创建用户生成器，实现run 方法
```
$ php artisan make:seeder UsersTableSeeder
```

3. 在DatabaseSeeder 类中实现run 方法，调用 call 方法
```
public function run()
{
    // \App\Models\User::factory(10)->create();
    Model::unguard();

    $this->call(UsersTableSeeder::class);
  
    Model::reguard();
}
```

4. 重置数据库
```
$ php artisan migrate:refresh
```

5. 填充数据
```
php artisan db:seed
```

### 第六章学到了什么
1. 通过路由传参与控制器进行交互（隐形路由绑定
2. 使用Patch 动作更新用户信息，Delete 动作删除用户
3. 使用Auth 中间件过滤用户请求、guest 中间件
4. 使用权限策略，对一些必要的动作进行权限验证
5. 使用数据填充来生成假数据
6. 重置数据库以及迁移数据库并生成新数据
7. 通过数据库迁移来进行数据库字段的更新

### 第七章

什么时候应该在控制器中增加`User $user` 这样的代码呢？

看路由，看路由，看路由，看路由是如何定义的。

如果路由中有这样的东西，那么一定要是要的。
```
Route::get("/users/{user}", "UserController@show")->name("users.show");
```

这是为什么呢？因为隐形路由绑定。

这里还有一个细节就是如何判断一个路由或者一个控制器是否是隐形路由绑定？
除了只是看路由之外，还需要看是否有与之对应的模型。这一点很重要哦。

另外什么时候需要`Request $request` 呢？也是看路由，Post 方法一定需要的。
```
// 定义路由
Route::get("password/{token}", "PasswordController@showResetForm")->name("password.reset");

// 方式一
public function showResetForm($token){
  var_dump($token);
}

// 方式二
public function showResetForm(Request $request){
  $token = $request->route()->parameter('token');
  var_dump($token);
}
```

Laravel 中的几种操作数据库的方式：
```
public function store(User $user){
  // 方式一
  $user->name = "boo";
  $user->save();
  
  // 方式二
  $user->update([
    "name" => "boo",
  ]);
  
  // 方式三
  User::where("id", $user->id)->update([
    "name" => "boo",
  ]);
  
  // 方式四
  DB::table("users")->where("id", $user->id)->update([
    "name" => "boo",
  ]);
}
```

### 第七章学到了什么
* 使用迁移为数据库表增加字段
* 在模型中，定义监听器，监听操作
* 使用Laravel 发送邮件功能
* 在本地（log）调试发送邮件功能
* 通过邮件发送注册链接来激活用户
* 通过邮件来找回密码

## 第八章
> Laravel 中模型与模型之间是如何进行关联的呢？

答案是通过主键与外键进行关联。

通过Eloquent 关联模型与模型之间的关系：
1. 一对一
2. 一对多
3. 多对一
4. 多对多

`Auth::user()` 方法可以获取到当前用户的实例。

在User模型中定义了一个方法，然后通过`Auth::user()`获取到的实例进行调用。

如果没有一对多的关系，需要这样创建一条微博：
```
App\Models\Status::create()
```

如果将用户模型与微博模型进行关联之后，可以得到以下方法：
```
$user->statuses()->create()
```

其中`statuses()` 是在用户模型中定义好的（名称可以不一样)：
```
public function statuses()
{
    // user表正向关联 status表
    return $this->hasMany(Status::class, "user_id", "id");
}
```

### 第八章学到了什么
* 两个模型之间如何进行关联
* 通过模型关联获取数据
* 对微博发布时间进行友好处理，并中文化
* 建立工厂、以及生成器、并生成假数据
* 通过数据关联来创建微博
* 通过数据关联来删除微博
* 修复批量赋值的错误

## 第九章
正式写SQL 之前，可以先使用tinker 通过模型操作数据。

通过在模型中定义一些方法，以便可以在其他地方直接获取到数据。

### 第九章学到了什么
* 多对多关系应用
* 新增和销毁多对多关联
* 使用 with 来避免N+1 问题  