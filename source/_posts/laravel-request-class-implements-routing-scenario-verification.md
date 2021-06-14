---
title: Laravel Request 类实现路由场景验证
date: 2021-06-14 16:19:23
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

在Laravel 中，有很多方法可以验证传入的数据，对于表单请求，通常主要有两种方式：
1. 在控制器中使用`ValidatesRequests` 的 `validate()`方法
2. 创建表单验证类

<!-- more -->

对于第一种方式，只适用一些功能单一、验证规则比较简单的验证场景。

对于复杂一些的验证场景，使用表单验证，会更方便一些。

经常使用表单验证的同学可能会知道，Request 类也不会万能的，对于一些重复使用的验证规则，默认的Request 类，并没有提供好的验证规则复用方法。

所以有没有某种方案，最终可以解决以下需求：
1. `rules()` 方法只需要返回一个该请求的验证规则数组
2. 基于路由场景验证，不同的验证场景可以使用相同的验证规则
3. 对于字段相同，但是验证规则不同的情况，可以重置验证规则

感谢 [sirping](https://learnku.com/blog/sirping) 的 [Laravel 验证类 实现 路由场景验证 和 控制器场景验证](https://learnku.com/articles/38825#863a85)，提供了一个基于路由的场景验证的简单易用方案。

## 重写FormRequest 类

因为每一个Request 类，后面都会使用到场景验证，所以这里直接创建一个基类继承于 `FormRequest`类，并重写相关方法：

`app/Http/Requests/BaseRequest.php`
```php
<?php

namespace App\Http\Requests;

use App\Traits\ApiResponse;
use Illuminate\Contracts\Validation\Factory as ValidationFactory;
use Illuminate\Contracts\Validation\Validator;
use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Http\Exceptions\HttpResponseException;

/**
 * Class BaseRequest
 *
 * @package App\Http\Requests
 */
class BaseRequest extends FormRequest
{
    use ApiResponse;

    /**
     * @var null
     */
    protected $scene = null;

    /**
     * 是否自动验证
     *
     * @var bool
     */
    protected $autoValidate = true;

    /**
     * @var array
     */
    protected $onlyRule=[];

    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function validateResolved()
    {
        if (method_exists($this, 'autoValidate')) {
            $this->autoValidate = $this->container->call([$this, 'autoValidate']);
        }

        if ($this->autoValidate) {
            $this->handleValidate();
        }
    }

    /**
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    protected function handleValidate()
    {
        $this->prepareForValidation();

        if (! $this->passesAuthorization()) {
            $this->failedAuthorization();
        }

        $instance = $this->getValidatorInstance();

        if ($instance->fails()) {
            $this->failedValidation($instance);
        }
    }

    /**
     * 定义 getValidatorInstance 下 validator 验证器
     *
     * @param $factory
     * @return mixed
     */
    public function validator($factory)
    {
        return $factory->make($this->validationData(), $this->getRules(), $this->messages(), $this->attributes());
    }

    /**
     * 验证方法（关闭自动验证时控制器调用）
     *
     * @param string $scene  场景名称 或 验证规则
     */
    public function validate($scene = '')
    {
        if (!$this->autoValidate) {
            if (is_array($scene)) {
                $this->onlyRule = $scene;
            } else {
                $this->scene = $scene;
            }

            $this->handleValidate();
        }
    }

    /**
     * 获取 rules
     *
     * @return array
     */
    protected function getRules()
    {
        return $this->handleScene($this->container->call([$this, 'rules']));
    }

    /**
     * 场景验证
     *
     * @param array $rule
     * @return array
     */
    protected function handleScene(array $rule)
    {
        if ($this->onlyRule) {
            return $this->handleRule($this->onlyRule, $rule);
        }

        $sceneName = $this->getSceneName();
        if ($sceneName && method_exists($this, 'scene')) {
            $scene = $this->container->call([$this, 'scene']);
            if (array_key_exists($sceneName, $scene)) {
                return $this->handleRule($scene[$sceneName], $rule);
            }
        }
        return  $rule;
    }

    /**
     * 处理Rule
     *
     * @param array $sceneRule
     * @param array $rule
     *
     * @return array
     */
    private function handleRule(array $sceneRule, array $rule)
    {
        $rules = [];
        foreach ($sceneRule as $key => $value) {
            if (is_numeric($key) && array_key_exists($value, $rule)) {
                $rules[$value] = $rule[$value];
            } else {
                $rules[$key] = $value;
            }
        }
        return $rules;
    }

    /**
     * 获取场景名称
     *
     * @return string
     */
    protected function getSceneName()
    {
        return is_null($this->scene) ? $this->route()->getAction('_scene') : $this->scene;
    }

    /**
     * 通过重写 failedValidation，方便Request 类抛出异常
     *
     * @param Validator $validator
     * @throws
     */
    protected function failedValidation(Validator $validator)
    {
        throw new HttpResponseException(
            $this->failed($validator->errors()->first())
        );
    }
}
```
其中，`ApiResponse` 是一个封装了返回客户端内容的 Trait。

## 添加路由场景方法

然后在 `app\Providers\AppServiceProvider.php` 类中的 `boot()` 方法中添加场景方法：
```php
<?php

namespace App\Providers;

use Illuminate\Routing\Route;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    // ...

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        // 添加场景验证 scene 方法
        Route::macro('scene', function ($scene = null) {
            $action = Route::getAction();
            $action['_scene'] = $scene;
            Route::setAction($action);
        });
    }
}
```

## 使用路由场景方法

该自定义方法用于路由场景验证，在 `Route->action` 增加一个 `_scene` 属性。其实用法和路由别名函数是一样的：
```php
Route::post('add','UserController@add')->scene('add');
```
## 路由场景验证 

`UserRequest` 使用示例：
```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\BaseRequest;

class UserRequest extends BaseRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        // 定义验证规则
        return [
            'name' => 'required|string|unique:users',
            'email' => 'required|email|unique:users',
        ];
    }

    /**
     * 场景规则
     *
     * @return array
     */
    public function scene()
    {
        // 格式  ['场景名' => [规则]]
        return [
             // add 场景
            'add' => [
                'name',                          // 复用 rules() 下 name 规则
                'email' => 'email|unique:users'  // 重置规则
            ],
             // edit场景
            'edit' => ['name'],
        ]
    }
}    
```
`scene()` 方法中的场景，不是控制器的方法名称，而是需要通过路由去自定义，可以使任意合法的名称，不一定要与控制器方法名保持一致。

至此就完成了上面提到的三个需求，使用起来也比较简单，没有破坏框架原本用法。

## 参考链接
* [Laravel 验证类 实现 路由场景验证 和 控制器场景验证](https://learnku.com/articles/38825#863a85)
* [修改 Laravel FormRequest 验证，实现场景验证](https://learnku.com/laravel/t/31215)