# 数据库操作和查询构造器

在Laravel中执行数据库操作有两种方式，一种是使用`\DB`外观对象的静态方法直接执行sql查询，另外一种是使用Model类的静态方法（实际上也是Facade的实现，使用静态访问方式访问Model的方法，内部采用了`__callStatic`魔术方法代理了对成员方法的访问。

<!-- more -->

## 查询操作

### 基本查询操作

#### 使用sql语句执行select查询操作

    $results = DB::select('select * from users where id = ?', [1]);
    foreach ($results as $res) {
        echo $res->name;
    }


返回结果为数组，数组中每一个值为一个`StdClass`对象。

也可以使用命名绑定，推荐使用这种方式，更加清晰一些

    $results = DB::select('select * from users where id = :id', ['id' => 1]);
    
#### 从数据表中取得所有的数据列

    $users = DB::table('users')->get();
    
    foreach ($users as $user)
    {
        var_dump($user->name);
    }

#### 从表中查询单行/列

使用`first`方法返回单行数据，该方法返回的是一个`stdClass`对象

    $user = DB::table('users')->where('name', 'John')->first();
    echo $user->name;

如果只需要一列的值，则可以使用`value`方法直接获取单列的值

    $email = DB::table('users')->where('name', 'John')->value('email');
    
    
#### 从数据表中分块查找数据列

该方法用于数据表中有大量的数据的操作，每次从结果集中取出一部分，使用闭包函数进行处理，然后再处理下一部分，该命令一般用于`Artisan`命令行程序中处理大量数据。

    DB::table('users')->chunk(100, function($users)
    {
        foreach ($users as $user)
        {
            //
        }
    });

在闭包函数中，如果返回`false`，则会停止后续的处理。

#### 从数据表中查询某一列的列表

比如我们希望查询出角色表中所有的`title`字段值

    $titles = DB::table('roles')->pluck('title');
    
    foreach ($titles as $title) {
        echo $title;
    }

这里的`pluck`函数有两个参数

    Collection pluck( string $column, string|null $key = null)

第一个参数为要查询的列，第二个参数是每一列的key

    $roles = DB::table('roles')->pluck('title', 'name');
    
    foreach ($roles as $name => $title) {
        echo $title;
    }

#### 聚集函数

查询构造器也提供了一些聚集函数如`count`，`max`，`min`，`avg`，`sum`等

    $users = DB::table('users')->count();
    $price = DB::table('orders')->max('price');
    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

#### 指定select查询条件

##### 查询指定的列

    $users = DB::table('users')->select('name', 'email as user_email')->get();
    
如果已经指定了select，但是又希望再次添加一些字段，使用它`addSelect`方法

    $query = DB::table('users')->select('name');
    $users = $query->addSelect('age')->get();


##### 查询不同的结果distinct

    $users = DB::table('users')->distinct()->get();

#### 使用原生表达式

使用`DB::raw`方法可以向查询中注入需要的sql片段，但是非常不推荐使用该方法，用不好会 **产生sql注入**

    $users = DB::table('users')
          ->select(DB::raw('count(*) as user_count, status'))
          ->where('status', '<>', 1)
          ->groupBy('status')
          ->get();

### Join操作

#### 内连接 Inner Join

使用`join`执行内连接操作，该函数第一个参数为要连接的表名，其它参数指定了连接约束

    $users = DB::table('users')
      ->join('contacts', 'users.id', '=', 'contacts.user_id')
      ->join('orders', 'users.id', '=', 'orders.user_id')
      ->select('users.*', 'contacts.phone', 'orders.price')
      ->get();

#### 左连接 Left Join

使用`leftJoin`方法执行左连接操作，参数和`join`一样

    $users = DB::table('users')
      ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
      ->get();
      
#### 高级Join方法

如果join方法的约束条件比较复杂，可以使用闭包函数的方式指定

    DB::table('users')
       ->join('contacts', function ($join) {
           $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
       })
       ->get();

如果join约束中要使用列值与指定数组比较，则可以使用`where`和`OrWhere`方法

    DB::table('users')
       ->join('contacts', function ($join) {
           $join->on('users.id', '=', 'contacts.user_id')
                ->where('contacts.user_id', '>', 5);
       })
       ->get();

### Union操作

要使用union操作，可以先创建一个query，然后再使用union方法去绑定第二个query

    $first = DB::table('users')
                ->whereNull('first_name');
    
    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

同样，`unionAll`方法也是可以使用的，参数与union相同。

### Where查询条件

#### 简单的wehere条件

使用`where`方法为查询增加where条件，该函数一般需要三个参数：**列名**，**操作符**（任何数据库支持的操作符都可以），**列值**。

    $users = DB::table('users')->where('votes', '=', 100)->get();
    $users = DB::table('users')->where('votes', 100)->get();

> 为了方便起见，如果只提供两个参数，则默认第二个参数为`=`，执行相等匹配。

    $users = DB::table('users')
          ->where('votes', '>=', 100)
          ->get();
    
    $users = DB::table('users')
          ->where('votes', '<>', 100)
          ->get();
    
    $users = DB::table('users')
          ->where('name', 'like', 'T%')
          ->get();

`where`条件也可以使用数组提供：

    $users = DB::table('users')->where([
        ['status','1'],
        ['subscribed','<>','1'],
    ])->get();

#### OR条件

如果where条件要使用or操作，则使用`orWhere`方法

    $users = DB::table('users')
         ->where('votes', '>', 100)
         ->orWhere('name', 'John')
         ->get();

#### 其它where条件

##### whereBetween / whereNotBetween

    $users = DB::table('users')
        ->whereBetween('votes', [1, 100])->get();
    $users = DB::table('users')
        ->whereNotBetween('votes', [1, 100])->get();


##### whereIn / whereNotIn

    $users = DB::table('users')
         ->whereIn('id', [1, 2, 3])
         ->get();
    $users = DB::table('users')
         ->whereNotIn('id', [1, 2, 3])
         ->get();

##### whereNull / whereNotNull

    $users = DB::table('users')
         ->whereNull('updated_at')
         ->get();
    $users = DB::table('users')
         ->whereNotNull('updated_at')
         ->get();

#### 高级where条件

##### 参数组（嵌套条件）

    DB::table('users')
      ->where('name', '=', 'John')
      ->orWhere(function ($query) {
          $query->where('votes', '>', 100)
                ->where('title', '<>', 'Admin');
      })
      ->get();

上述代码等价于下列sql

    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')

##### whereExists (where exist)

    DB::table('users')
      ->whereExists(function ($query) {
          $query->select(DB::raw(1))
                ->from('orders')
                ->whereRaw('orders.user_id = users.id');
      })
      ->get();

上述代码与下列sql等价

    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )

#### JSON类型的列查询

MySQL 5.7和Postgres数据库中提供了新的数据类型json，对json提供了原生的支持，使用`->`可以对json列进行查询。

    $users = DB::table('users')
          ->where('options->language', 'en')
          ->get();
    
    $users = DB::table('users')
          ->where('preferences->dining->meal', 'salad')
          ->get();

#### Ordering, Grouping, Limit, & Offset

    $users = DB::table('users')
          ->orderBy('name', 'desc')
          ->get();
          
    $users = DB::table('users')
          ->groupBy('account_id')
          ->having('account_id', '>', 100)
          ->get();
          
    $users = DB::table('orders')
          ->select('department', DB::raw('SUM(price) as total_sales'))
          ->groupBy('department')
          ->havingRaw('SUM(price) > 2500')
          ->get();

要限制查询返回的结果行数，或者是跳过指定行数的结果（`OFFSET`），可以使用`skip`和`take`方法

    $users = DB::table('users')->skip(10)->take(5)->get();



## 插入操作

### 使用sql语句执行插入

插入操作与select操作类似,使用`insert`函数

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
    
### 基本插入操作

    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );
    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);

如果希望插入后能够获取新增数据的id，则可以使用`insertGetId`方法

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );


## 更新操作

### 使用sql语句执行更新操作

执行DB中的`update`后，会返回 **操作影响的数据行数**

    DB::update('update users set votes = 100 where name = ?', ['John']);

### 基本更新操作

    DB::table('users')
          ->where('id', 1)
          ->update(['votes' => 1]);

### 指定列的增减

    DB::table('users')->increment('votes');
    DB::table('users')->increment('votes', 5);
    DB::table('users')->decrement('votes');
    DB::table('users')->decrement('votes', 5);

在执行自增/减操作的时候，也可以同时更新其它列

    DB::table('users')->increment('votes', 1, ['name' => 'John']);


## 删除操作

### 使用sql执行删除

执行DB中的`delete`后，会返回 **操作影响的数据行数**

    DB::delete('delete from users');
    
### 基本删除操作

    DB::table('users')->delete();
    DB::table('users')->where('votes', '<', 100)->delete();

如果希望truncate整个表，则使用`truncate`方法

    DB::table('users')->truncate();

## 悲观锁

使用`sharedLock`方法可以避免选定的行在事务提交之前被修改

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

另外`lockForUpdate`方法可以避免其它的共享锁修改或者是选定

    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();


## 事务处理

使用`transaction`方法的callback函数执行事务处理

    DB::transaction(function()
    {
        DB::table('users')->update(['votes' => 1]);
    
        DB::table('posts')->delete();
    });

> 在回调函数中，抛出任何异常都会导致事务回滚

如果需要手动管理事务，则使用如下函数

    DB::beginTransaction();
    DB::rollback();
    DB::commit();
    
> 使用DB类的静态方法启用的事务不仅对普通sql查询有效，对**Eloquent ORM**同样有效，因为它内部也是调用了DB类的数据库连接。

## 查看日志记录

查看请求执行的sql日志记录，需要先执行`enableQueryLog`开启，然后执行`getQueryLog`获取

    DB::connection()->enableQueryLog();
    $queries = DB::getQueryLog();


## 其它操作

执行一般的sql语法

    DB::statement('drop table users');

监听查找事件，可以用来对执行的sql进行记录
    
    DB::listen(function($sql, $bindings, $time)
    {
        // $query->sql
        // $query->bindings
        // $query->time

    });

获取某个数据库连接

    $users = DB::connection('foo')->select(...);

如果还不能满足需求，可以获取PDO对象

    $pdo = DB::connection()->getPdo();

这样不管什么操作都可以做了吧

另外含有两个方法，用于重新连接到指定数据库和断开连接

    DB::reconnect('foo');
    DB::disconnect('foo')d;


-----

参考： [Laravel 5.2 官方文档](https://laravel.com/docs/5.2/database)

