---
title: 如何更好的使用 Laravel 软删除
date: 2021-04-10 20:26:01
tags: ["PHP", "Laravel"]
categories: ["PHP", "Laravel"]
---

通常对于数据库中比较重要的数据，不会直接删除，而是采用软删除。

<!-- more -->

Laravel 的Eloquent 也提供相应的功能达到软删除模型的目的，不过个人觉得Laravel 的软删除存在一些问题：

 > Laravel中使用了一个日期字段作为标识状态，`deleted_at` 默认值为`NULL`，如果记录被删除了，`deleted_at` 的值则为当前时间戳，所以只能通过`is null ` or `not is null`查询一条记录是否被删除，这会导致Mysql 引擎放弃使用索引而进行全表扫描，查询效率可想而知。

可以通过重写`SoftDeletes.php` 类来修改Laravel SoftDelete 的逻辑：
```php
<?php
namespace App\Models\Traits;

use Illuminate\Database\Eloquent\SoftDeletes;

trait SoftDeletesEx {

    use SoftDeletes;

    /**
     * Boot the soft deleting trait for a model.
     *
     * @return void
     */
    public static function bootSoftDeletes() {
        static::addGlobalScope(new SoftDeletingScopeEx());
    }

    /**
     * Get the name of the "deleted at" column.
     *
     *
     * @return string
     */
    public function getDeletedAtColumn()
    {
        // 自定义标识字段
        return 'is_deleted';
    }

    /**
     * Perform the actual delete query on this model instance.
     *
     * @return void
     */
    protected function runSoftDelete() {
        $query = $this->newQueryWithoutScopes()->where($this->getKeyName(), $this->getKey());

        // 0. 正常 1. 已删除
        $this->{$this->getDeletedAtColumn()} = $time = 1;

        $query->update([
            $this->getDeletedAtColumn() => $time
        ]);
    }

    /**
     * Restore a soft-deleted model instance.
     *
     * @return bool|null
     */
    public function restore() {
        // If the restoring event does not return false, we will proceed with this
        // restore operation. Otherwise, we bail out so the developer will stop
        // the restore totally. We will clear the deleted timestamp and save.
        if ($this->fireModelEvent('restoring') === false) {
            return false;
        }

        $this->{$this->getDeletedAtColumn()} = 0;

        // Once we have saved the model, we will fire the "restored" event so this
        // developer will do anything they need to after a restore operation is
        // totally finished. Then we will return the result of the save call.
        $this->exists = true;

        $result = $this->save();

        $this->fireModelEvent('restored', false);

        return $result;
    }

    /**
     * Determine if the model instance has been soft-deleted.
     *
     * @return bool
     */
    public function trashed() {
        return ! ($this->{$this->getDeletedAtColumn()} === 0);
    }
}
```
通过定义`is_deleted` 字段来标示是否删除，默认值`0. 未删除`，`1. 已删除`，同时给该字段添加普通索引。

接着还需要重写`SoftDeletingScope.php` 类，约束默认查询`is_deleted = 0` 的记录：
```php
<?php
namespace App\Models\Traits;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletingScope;

class SoftDeletingScopeEx extends SoftDeletingScope {

    /**
     * 将约束加到 Eloquent 查询构造中，这样默认查询的就是 `is_deleted` = 0 的记录了
     * Apply the scope to a given Eloquent query builder.
     *
     * @param \Illuminate\Database\Eloquent\Builder $builder
     * @param \Illuminate\Database\Eloquent\Model $model
     * @return void
     */
    public function apply(Builder $builder, Model $model) {
        $builder->where($model->getQualifiedDeletedAtColumn(), 0);
    }

    /**
     * Extend the query builder with the needed functions.
     *
     * @param \Illuminate\Database\Eloquent\Builder $builder
     * @return void
     */
    public function extend(Builder $builder) {
        foreach ($this->extensions as $extension) {
            $this->{"add{$extension}"}($builder);
        }

        $builder->onDelete(function (Builder $builder) {
            $column = $this->getDeletedAtColumn($builder);

            return $builder->update([
                $column => \DB::Raw('UNIX_TIMESTAMP(NOW())')
            ]);
        });
    }

    /**
     * Add the restore extension to the builder.
     *
     * @param \Illuminate\Database\Eloquent\Builder $builder
     * @return void
     */
    protected function addRestore(Builder $builder) {
        $builder->macro('restore', function (Builder $builder) {
            $builder->withTrashed();

            return $builder->update([
                $builder->getModel()
                    ->getDeletedAtColumn() => 0
            ]);
        });
    }

    /**
     * Add the without-trashed extension to the builder.
     *
     * @param \Illuminate\Database\Eloquent\Builder $builder
     * @return void
     */
    protected function addWithoutTrashed(Builder $builder) {
        $builder->macro('withoutTrashed', function (Builder $builder) {
            $model = $builder->getModel();

            $builder->withoutGlobalScope($this)
                ->where($model->getQualifiedDeletedAtColumn(), 0);

            return $builder;
        });
    }

    /**
     * Add the only-trashed extension to the builder.
     *
     * @param \Illuminate\Database\Eloquent\Builder $builder
     * @return void
     */
    protected function addOnlyTrashed(Builder $builder) {
        $builder->macro('onlyTrashed', function (Builder $builder) {
            $model = $builder->getModel();

            $builder->withoutGlobalScope($this)
                ->where($model->getQualifiedDeletedAtColumn(), '<>', 0);

            return $builder;
        });
    }
}
```

最后应用到Model 中：
```php
<?php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use App\Models\Traits\SoftDeletesEx;

class User extends Model
{
    use SoftDeletesEx;
    
    protected $table = 'user';
    
    protected $dates = ["is_deleted"];
}
```

## 参考链接
* [laravel框架自定义软删除](https://blog.csdn.net/loophome/article/details/81978010)
* [Laravel5软删除（SoftDeletes）的deleted_at改造](http://blog.dreamlikes.cn/archives/892)