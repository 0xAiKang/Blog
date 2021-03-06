---
title: 『转载』如何使用Repository 模式
date: 2021-04-20 21:01:38
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

若将数据库逻辑都写在 Model 里，会造成 model 代码的臃肿难以维护，基于 **SOLID** 原则，我们应该使用 **Repository** 模式辅助 Model，将相关的数据库逻辑封装在不同的 Repository，方便后期项目的维护。

<!-- more -->

## 数据库逻辑
在 CURD 中，CUR 比较稳定，但 Read 的部分则变化万千，大部分的数据库逻辑都在描述 Read 部分，若将数据库逻辑写在 Controller 或 Model 都不合适，会造成 Controller 或 Model 代码臃肿，如后难以维护。

## Model
使用 Repository 模式之后，Model 仅仅当成 Eloquent Class 即可，不需要包含数据库逻辑，仅保留如下部分：
* Property： 如 `$table`、`$fillable` ...
* Mutator： 包括 mutator 与 accessor
* Method： relation 类的方法，比如使用 `hasMany()` 与 `belongsTo()`

单一对应关系：
* hasOne
* belongsTo
* morphTo
* morphOne

多个对应关系指的是使用以下关键词定义的关联模型：
* hasMany
* belongsToMany
* morphMany
* morphToMany
* morphedByMany

## Repository
在开发时常常会在 Controller 直接调用 Model 写数据库逻辑，如下：获取数据库中用户 `age>20` 的数据。

```php
public function index()
{
    return User::where('age','>',20)->orderBy('age')->get();
}
```

这样写逻辑会有几个问题：
* 将数据库逻辑写在 Controller，造成 Controller 代码臃肿难以维护。
* 违反了 SOLID 的单一职责原则，数据库逻辑不应该写在 Controller 中。
* Controller 直接操作 Model，使得对 Controller 做单元测试困难。

比较好的方式是使用 Repository：
* 将 Model 依赖注入到 Repository。
* 将数据库逻辑写在 Repository。
* 将 Repository 依赖注入到 Service。

`app/Repositories/UserRepostitory.php` 中的内容：

```php
<?php

namespace App\Repositories;

use App\User;

/**
 * Class UserRepository
 * @package App\Repositories
 */
class UserRepository
{
    /**
     * @var User
     */
    private $user;

    /**
     * UserRepository constructor.
     * @param $user
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }

    /**
     * @param $age
     * @return \Illuminate\Database\Eloquent\Collection|static[]
     */
    public function getAgeLargerThan($age)
    {
        return $this->user
            ->where('age', '>', $age)
            ->orderBy('age')
            ->get();
    }
}
```

在控制器`app\Controllers\UserController.php` 中使用依赖注入：
```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;
use Illuminate\Http\Request;

/**
 * Class UserController
 *
 * @package App\Http\Controllers
 */
class UserController extends Controller
{
    /**
     * @var \App\Repositories\UserRepository
     */
    protected $userRepository;

    /**
     * UserController constructor.
     * @param $userRepository
     */
    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    /**
     * @return \Illuminate\Database\Eloquent\Collection|static[]
     */
    public function index()
    {
        return $this->userRepository->getAgeLargerThan(20);
    }
}
```

将相依的 `UserRepository` 依賴注入到 `UserController`，并从原本直接依赖 `User Model` 改成依赖注入的 `UserRepository`。

优点：
* 将数据库逻辑写在 Repository 里，解决了 Controller 代码臃肿的问题。
* 符合 SOLID 的单一职责原则：数据库逻辑写在 Repository 里，没写在 Controller 里。
* 符合 SOLID 的依赖反转原则：Controller 并非直接相依与 Repositroy，而是将 Repository 依赖注入进 Controller。

> 注意⚠️：实际上建议 Repository 仅依赖注入进 Service，而不是直接注入在 Controller。

## 其他问题

### 是否该建立 Repository Interface？

理论上使用依赖注入时，应该使用 Interface ，不过 Interface 目的在于更换数据库，让代码达到开放封闭的要求，但是实际上要更改 Reposiroty 的机会也不多，除非是从 MySQL 更换到 MongoDB，此时就应该建立 Repository Interface。
不过由于我们使用了依赖注入，将来要从 Class 改成 Interface 也很方便，只要在 Constructor 的 type hint 改成 Interface 即可，维护成本很低，所以在此大可使用 Repository Class 即可，不一定得用Interface而造成 Over Design，等真正需要修改时，再重构 Interface 即可。

### 是否该使用 Query Scope?

Laravel 4.2 就有 QueryScope，到后面的版本也都还保留着，它让我们可以将逻辑代码写在 Model ，解决了维护与重复使用的问题。
如 app/User.php 里的代码：

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

/**
 * App\User
 * 
 * @mixin \Eloquent
 */
class User extends Authenticatable
{
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];

    /**
     *
     * @param Builder $query
     * @param integer $age
     *
     * @return Builder
     */
    public function scopeGetAgerLargerThan($query, $age)
    {
        return $query->where('age', '>', $age)
            ->orderBy('age');
    }
}
```

QueryScope 必须以 scope开头，第一个参数为 queryBuilder，一定要加上；第二个参数以后为自己要传入的参数。
由于回传必须是一个 queryBuilder ，因此不需要加上 `get()`，在
`app/Controllers/UserController.php` 中使用代码：
```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;
use App\User;
use Illuminate\Http\Request;

/**
 * Class UserController
 *
 * @package App\Http\Controllers
 */
class UserController extends Controller
{
    /**
     * @var \App\Repositories\UserRepository
     */
    protected $userRepository;

    /**
     * UserController constructor.
     * @param $userRepository
     */
    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    /**
     * @return \Illuminate\Database\Eloquent\Collection|static[]
     */
    public function index()
    {
        return User::getAgerLargerThan(20)->get();
    }
}
```

在 Controller 中使用 QueryScope 时，不需要加上 Prefix，由于其本质是 queryBuilder，所以还要加上 get() 才能获得 Conllection 数据。

由于 QueryScope 是写在 Model，不是写在 Controller，所以基本上解决了 Controller 臃肿违反 SOLID 的单一职责原则的问题， Controller 也可以重复使用 QueryScope ，已经比直接将资料库逻辑写在 Controlelr 中好很多。
不过若在中大型项目中，仍然有以下问题：

* Model 已经有原来的责任，若再加上 queryScope，造成 Model 过于臃肿难以维护。
* 若数据库逻辑很多，可能拆成多个 Repository，可是确很难拆成多个 Model。
* 单元测试困难，必须面临 mock Eloquent 的问题。

## 最后
实际开发时，可以一开始 1 个 Repository 对应 1 个 Model，但是也不必太过执着于 1 个 Repository，一定要对应 1 个 Model，可将 Repository 视为逻辑上的数据库逻辑类别即可，可以横跨多个Model处理，也可以 1 个 Model 拆成多个 Repository，视情况而定。
Repository 使得数据库逻辑从 Controller 或 Model 中解放，不仅更容易维护、更容易拓展、更容易重复使用，也更容易测试。


----- 

## 是否需要使用 Repository ？
倒底该不该用Repository，对于这个问题，从未停止过讨论。我认为没有绝对的用或者不用，需要根据项目实际情况而定。

结合自己的一些项目经验，我的理解是：对于小项目而言，复杂查询并不多，直接使用ORM效率更高，前期快速开发才是关键，过早使用Repository 反而会造成过度设计; 而对于起步本身就是中大型项目，则可以考虑使用Repository 将复杂的查询和业务逻辑分开。

单一职责原则：
* Request 负责表单验证
* Model 负责维护ORM
* Controller 负责获取请求参数
* Service 负责处理业务逻辑
* Repository 负责从数据库里取数据

这里有两个讨论很精彩：[绝不 使用 Repository??](https://learnku.com/laravel/t/16338)、[绝不 使用 Repository?](https://learnku.com/laravel/t/10323/never-use-repository)