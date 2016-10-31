# 跟我一起学Laravel-EloquentORM基础部分

使用Eloquent *['eləkwənt]* 时，数据库查询构造器的方法对模型类也是也用的，使用上只是省略了`DB::table('表名')`部分。

> 在模型中使用protected成员变量`$table`指定绑定的表名。

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * The table associated with the model.
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

Eloquent 假设每个表都有一个名为`id`的主键，可以通过`$primaryKey`成员变量覆盖该字段名称，另外，Eloquent假设主键字段是自增的整数，如果你想用非自增的主键或者非数字的主键的话，必须指定模型中的public属性`$incrementing`为false。

默认情况下，Eloquent期望表中存在`created_at`和`updated_at`两个字段，字段类型为`timestamp`，如果不希望这两个字段的话，设置`$timestamps`为false

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Flight extends Model
    {
        /**
         * Indicates if the model should be timestamped.
         *
         * @var bool
         */
        public $timestamps = false;
        
        /**
         * The storage format of the model's date columns.
         *
         * @var string
         */
        protected $dateFormat = 'U';
    }

使用`protected $connection = 'connection-name'`指定模型采用的数据库连接。

## 查询

### 基本查询操作
    
方法`all`用于返回模型表中所有的结果
    
    $flights = Flight::all();
    foreach ($flights as $flight) {
        echo $flight->name;
    }

也可以使用`get`方法为查询结果添加约束

    $flights = App\Flight::where('active', 1)
         ->orderBy('name', 'desc')
         ->take(10)
         ->get();

> 可以看到，查询构造器的方法对模型类也是可以使用的

在eloquent ORM中，`get`和`all`方法查询出多个结果集，它们的返回值是一个`Illuminate\Database\Eloquent\Collection`对象，该对象提供了多种对结果集操作的方法

    public function find($key, $default = null);
    public function contains($key, $value = null);
    public function modelKeys();
    public function diff($items)
    ...
    
该对象的方法有很多，这里只列出一小部分，更多方法参考API文档 [Collection](https://laravel.com/api/5.2/Illuminate/Database/Eloquent/Collection.html) 和[使用说明文档](https://laravel.com/docs/5.2/collections)。

对大量结果分段处理，同样是使用`chunk`方法
    
    Flight::chunk(200, function ($flights) {
        foreach ($flights as $flight) {
            //
        }
    });

### 查询单个结果

使用`find`和`first`方法查询单个结果，返回的是单个的模型实例

    // 通过主键查询模型...
    $flight = App\Flight::find(1);
    
    // 使用约束...
    $flight = App\Flight::where('active', 1)->first();

使用`find`方法也可以返回多个结果，以`Collection`对象的形式返回，参数为多个主键

    $flights = App\Flight::find([1, 2, 3]);

如果查询不到结果的话，可以使用`findOrFail`或者`firstOrFail`方法，这两个方法在查询不到结果的时候会抛出`Illuminate\Database\Eloquent\ModelNotFoundException`异常

    $model = App\Flight::findOrFail(1);
    $model = App\Flight::where('legs', '>', 100)->firstOrFail();
    
如果没有捕获这个异常的话，laravel会自动返回给用户一个`404`的响应结果，因此如果希望找不到的时候返回`404`，是可以直接使用该方法返回的

    Route::get('/api/flights/{id}', function ($id) {
        return App\Flight::findOrFail($id);
    });

### 查询聚集函数结果

与查询构造器查询方法一样，可以使用聚集函数返回结果，常见的比如`max`， `min`，`avg`，`sum`，`count`等

    $count = App\Flight::where('active', 1)->count();
    $max = App\Flight::where('active', 1)->max('price');
    

### 分页查询

分页查询可以直接使用`paginate`函数

    LengthAwarePaginator paginate( 
        int $perPage = null, 
        array $columns = array('*'), 
        string $pageName = 'page', 
        int|null $page = null
    )

参数说明

| 参数      | 类型    | 说明
|----------|---------|---
| perPage  | int     | 每页显示数量
| columns  | array   | 查询的列名
| pageName | string  | 页码参数名称
| page     | int     | 当前页码

返回值为 [LengthAwarePaginator](https://laravel.com/api/5.2/Illuminate/Contracts/Pagination/LengthAwarePaginator.html) 对象。

    $limit = 20;
    $page = 1;
    return Enterprise::paginate($limit, ['*'], 'page', $page);

    
## 插入

### 基本插入操作

插入新的数据只需要创建一个新的模型实例，然后设置模型属性，最后调用`save`方法即可
    
    $flight = new Flight;
    $flight->name = $request->name;
    $flight->save();

> 在调用`save`方法的时候，会自动为`created_at`和`updated_at`字段设置时间戳，不需要手动指定

### 批量赋值插入

使用`create`方法可以执行批量为模型的属性赋值的插入操作，该方法将会返回新插入的模型，在执行`create`方法之前，需要先在模型中指定`fillable`和`guarded`属性，用于防止不合法的属性赋值（例如避免用户传入的is_admin属性被误录入数据表）。

指定`$fillable`属性的目的是该属性指定的字段可以通过`create`方法插入，其它的字段将被过滤掉，类似于白名单，而`$guarded`则相反，类似于黑名单。

    protected $fillable = ['name'];
    // OR
    protected $guarded = ['price'];

执行`create`操作就只有白名单或者黑名单之外的字段可以更新了

    $flight = App\Flight::create(['name' => 'Flight 10']);

除了`create`方法，还有两外两个方法可以使用`firstOrNew`和`firstOrCreate`。

`firstOrCreate`方法用来使用给定的列值对查询记录，如果查不到则插入新的。`fristOrNew`与`firstOrCreate`类似，不同在于如果不存在，它会返回一个新的模型对象，不过该模型是未经过持久化的，需要手动调用`save`方法持久化到数据库。

    // 使用属性检索flight，如果不存在则创建...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);
    
    // 使用属性检索flight，如果不存在则创建一个模型实例...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);


## 更新

### 基本更新操作

方法`save`不仅可以要用来插入新的数据，也可以用来更新数据，只需先使用模型方法查询出要更新的数据，设置模型属性为新的值，然后再`save`就可以更新了，`updated_at`字段会自动更新。

    $flight = App\Flight::find(1);
    $flight->name = 'New Flight Name';
    $flight->save();

也可使用`update`方法对多个结果进行更新

    App\Flight::where('active', 1)
        ->where('destination', 'San Diego')
        ->update(['delayed' => 1]);

## 删除

### 基本删除操作

使用`delete`方法删除模型

    $flight = App\Flight::find(1);
    $flight->delete();
    
上述方法需要先查询出模型对象，然后再删除，也可以直接使用主键删除模型而不查询，使用`destroy`方法

    App\Flight::destroy(1);
    App\Flight::destroy([1, 2, 3]);
    App\Flight::destroy(1, 2, 3);

使用约束条件删除，返回删除的行数

    $deletedRows = App\Flight::where('active', 0)->delete();

### 软删除

软删除是在表中增加`deleted_at`字段，当删除记录的时候不会真实删除记录，而是设置该字段的时间戳，由Eloquent模型屏蔽已经设置该字段的数据。

要启用软删除，可以在模型中引用`Illuminate\Database\Eloquent\SoftDeletes`这个Trait，并且在`dates`属性中增加`deleted_at`字段。

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;
    
    class Flight extends Model
    {
        use SoftDeletes;
    
        /**
         * The attributes that should be mutated to dates.
         *
         * @var array
         */
        protected $dates = ['deleted_at'];
    }

要判断一个模型是否被软删除了的话，可以使用`trashed`方法
    
    if ($flight->trashed()) {
        //
    }

#### 查询软删除的模型

##### 包含软删除的模型

如果模型被软删除了，普通查询是不会查询到该结果的，可以使用`withTrashed`方法强制返回软删除的结果

    $flights = App\Flight::withTrashed()
          ->where('account_id', 1)
          ->get();

    // 关联操作中也可以使用
    $flight->history()->withTrashed()->get();

##### 只查询软删除的模型

    $flights = App\Flight::onlyTrashed()
          ->where('airline_id', 1)
          ->get();

##### 还原软删除的模型

查询到软删除的模型实例之后，调用`restore`方法还原

    $flight->restore();

也可以在查询中使用

    App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();
    
    // 关联操作中也可以使用
    $flight->history()->restore();

##### 强制删除（持久化删除）

    // Force deleting a single model instance...
    $flight->forceDelete();
    
    // Force deleting all related models...
    $flight->history()->forceDelete();

上述操作后，数据会被真实删除。

---

参考：

- [Eloquent: Getting Started](https://laravel.com/docs/5.2/eloquent)
