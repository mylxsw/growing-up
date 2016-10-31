# 跟我一起学Laravel-EloquentORM高级部分

## 查询作用域

### 全局作用域

全局作用域允许你对给定模型的所有查询添加约束。使用全局作用域功能可以为模型的所有操作增加约束。

> 软删除功能实际上就是利用了全局作用域功能

实现一个全局作用域功能只需要定义一个实现`Illuminate\Database\Eloquent\Scope`接口的类，该接口只有一个方法`apply`，在该方法中增加查询需要的约束


    <?php
    
    namespace App\Scopes;
    
    use Illuminate\Database\Eloquent\Scope;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Builder;
    
    class AgeScope implements Scope
    {
        /**
         * Apply the scope to a given Eloquent query builder.
         *
         * @param  \Illuminate\Database\Eloquent\Builder  $builder
         * @param  \Illuminate\Database\Eloquent\Model  $model
         * @return void
         */
        public function apply(Builder $builder, Model $model)
        {
            return $builder->where('age', '>', 200);
        }
    }


在模型的中，需要覆盖其`boot`方法，在该方法中增加`addGlobalScope`

    <?php
    
    namespace App;
    
    use App\Scopes\AgeScope;
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The "booting" method of the model.
         *
         * @return void
         */
        protected static function boot()
        {
            parent::boot();
    
            static::addGlobalScope(new AgeScope);
        }
    }

添加全局作用域之后，`User::all()`操作将会产生如下等价sql

    select * from `users` where `age` > 200

也可以使用匿名函数添加全局约束
    
    static::addGlobalScope('age', function(Builder $builder) {
      $builder->where('age', '>', 200);
    });

查询中要移除全局约束的限制，使用`withoutGlobalScope`方法

    // 只移除age约束
    User::withoutGlobalScope('age')->get();
    User::withoutGlobalScope(AgeScope::class)->get();
    // 移除所有约束
    User::withoutGlobalScopes()->get();
    // 移除多个约束
    User::withoutGlobalScopes([FirstScope::class, SecondScope::class])->get();

### 本地作用域

本地作用域只对部分查询添加约束，需要手动指定是否添加约束，在模型中添加约束方法，使用前缀`scope`

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Scope a query to only include popular users.
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopePopular($query)
        {
            return $query->where('votes', '>', 100);
        }
    
        /**
         * Scope a query to only include active users.
         *
         * @return \Illuminate\Database\Eloquent\Builder
         */
        public function scopeActive($query)
        {
            return $query->where('active', 1);
        }
    }

使用上述添加的本地约束查询，只需要在查询中使用`scope`前缀的方法，去掉`scope`前缀即可

    $users = App\User::popular()->active()->orderBy('created_at')->get();

本地作用域方法是可以接受参数的
    
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }

调用的时候

    $users = App\User::ofType('admin')->get();

## 事件

Eloquent模型会触发下列事件

`creating`, `created`, `updating`, `updated`, `saving`, `saved`,`deleting`, `deleted`, `restoring`, `restored`

### 使用场景

假设我们希望保存用户的时候对用户进行校验，校验通过后才允许保存到数据库，可以在服务提供者中为模型的事件绑定监听

    <?php
    
    namespace App\Providers;
    
    use App\User;
    use Illuminate\Support\ServiceProvider;
    
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            User::creating(function ($user) {
                if ( ! $user->isValid()) {
                    return false;
                }
            });
        }
    
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

上述服务提供者对象中，在框架启动时会监听模型的`creating`事件，当保存用户之间检查用户数据的合法性，如果不合法，返回false，模型数据不会被持久化到数据。

> 返回false会阻止模型的`save` / `update`操作

## 序列化

当构建JSON API的时候，经常会需要转换模型和关系为数组或者json。Eloquent提供了一些方法可以方便的来实现数据类型之间的转换。

### 转换模型/集合为数组 - toArray()

    $user = App\User::with('roles')->first();
    return $user->toArray();
    
    $users = App\User::all();
    return $users->toArray();

### 转换模型为json - toJson()

    $user = App\User::find(1);
    return $user->toJson();

    $user = App\User::find(1);
    return (string) $user;

### 隐藏属性

有时某些字段不应该被序列化，比如用户的密码等，使用`$hidden`字段控制那些字段不应该被序列化
    
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> 隐藏关联关系的时候，使用的是它的方法名称，不是动态的属性名

也可以使用`$visible`指定会被序列化的白名单
    
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be visible in arrays.
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

有时可能需要某个隐藏字段被临时序列化，使用`makeVisible`方法

    return $user->makeVisible('attribute')->toArray();

### 为json追加值

有时需要在json中追加一些数据库中不存在的字段，使用下列方法，现在模型中增加一个get方法

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
    
        /**
         * The accessors to append to the model's array form.
         *
         * @var array
         */
        protected $appends = ['is_admin'];

    
        /**
         * Get the administrator flag for the user.
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] == 'yes';
        }
    }

方法签名为`getXXXAttribute`格式，然后为模型的`$appends`字段设置字段名。

## Mutators

在Eloquent模型中，Accessor和Mutator可以用来对模型的属性进行处理，比如我们希望存储到表中的密码字段要经过加密才行，我们可以使用Laravel的加密工具自动的对它进行加密。

### Accessors & Mutators

#### accessors

要定义一个accessor，需要在模型中创建一个名称为`getXxxAttribute`的方法，其中的Xxx是驼峰命名法的字段名。

假设我们有一个字段是`first_name`，当我们尝试去获取first_name的值的时候，`getFirstNameAttribute`方法将会被自动的调用
    
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Get the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

在访问的时候，只需要正常的访问属性就可以

    $user = App\User::find(1);
    $firstName = $user->first_name;

#### mutators

创建mutators与accessorsl类似，创建名为`setXxxAttribute`的方法即可

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Set the user's first name.
         *
         * @param  string  $value
         * @return string
         */
        public function setFirstNameAttribute($value)
        {
            $this->attributes['first_name'] = strtolower($value);
        }
    }

赋值方式

    $user = App\User::find(1);
    $user->first_name = 'Sally';

#### 属性转换

模型的`$casts`属性提供了一种非常简便的方式转换属性为常见的数据类型，在模型中，使用`$casts`属性定义一个数组，该数组的key为要转换的属性名称，value为转换的数据类型，当前支持`integer`, `real`, `float`, `double`, `string`, `boolean`, `object`, `array`,`collection`, `date`, `datetime`, 和 `timestamp`。

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be casted to native types.
         *
         * @var array
         */
        protected $casts = [
            'is_admin' => 'boolean',
        ];
    }

数组类型的转换时非常有用的，我们在数据库中存储json数据的时候，可以将其转换为数组形式。
    
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The attributes that should be casted to native types.
         *
         * @var array
         */
        protected $casts = [
            'options' => 'array',
        ];
    }

从配置数组转换的属性取值或者赋值的时候都会自动的完成json和array的转换

    $user = App\User::find(1);  
    $options = $user->options;
    $options['key'] = 'value';
    $user->options = $options;
    $user->save();


---

参考： 

- [Eloquent: Getting Started](https://laravel.com/docs/5.2/eloquent)
- [Eloquent: Serialization](https://laravel.com/docs/5.2/eloquent-serialization)
- [Eloquent: Mutators](https://laravel.com/docs/5.2/eloquent-mutators)


