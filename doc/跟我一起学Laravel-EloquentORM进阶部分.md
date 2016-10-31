# Eloquent ORM - 进阶部分

## 关联关系

### One To One

假设User模型关联了Phone模型，要定义这样一个关联，需要在User模型中定义一个phone方法，该方法返回一个`hasOne`方法定义的关联

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * Get the phone record associated with the user.
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

`hasOne`方法的第一个参数为要关联的模型，定义好之后，可以使用下列语法查询到关联属性了

    $phone = User::find(1)->phone;

Eloquent会假定关联的外键是基于模型名称的，因此Phone模型会自动使用`user_id`字段作为外键，可以使用第二个参数和第三个参数覆盖

    return $this->hasOne('App\Phone', 'foreign_key');
    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');


#### 定义反向关系

定义上述的模型之后，就可以使用User模型获取Phone模型了，当然也可以通过Phone模型获取所属的User了，这就用到了`belongsTo`方法了

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Phone extends Model
    {
        /**
         * Get the user that owns the phone.
         */
        public function user()
        {
            return $this->belongsTo('App\User');
            // return $this->belongsTo('App\User', 'foreign_key');
            // return $this->belongsTo('App\User', 'foreign_key', 'other_key');

        }
    }

### One To Many

假设有一个帖子，它有很多关联的评论信息，这种情况下应该使用一对多的关联，使用`hasMany`方法

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Post extends Model
    {
        /**
         * Get the comments for the blog post.
         */
        public function comments()
        {
            return $this->hasMany('App\Comment');
        }
    }

查询操作

    $comments = App\Post::find(1)->comments;
    foreach ($comments as $comment) {
        //
    }
    
    $comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

#### 定义反向关联

反向关联也是使用`belongsTo`方法，参考One To One部分。

    $comment = App\Comment::find(1);
    echo $comment->post->title;

### Many To Many

多对多关联因为多了一个中间表，实现起来比`hasOne`和`hasMany`复杂一些。

考虑这样一个场景，用户可以属于多个角色，一个角色也可以属于多个用户。这就引入了三个表: `users`, `roles`, `role_user`。其中`role_user`表为关联表，包含两个字段`user_id`和`role_id`。

多对多关联需要使用`belongsToMany`方法
    
    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class User extends Model
    {
        /**
         * The roles that belong to the user.
         */
        public function roles()
        {
            // 指定关联表
            // return $this->belongsToMany('App\Role', 'role_user');
            // 指定关联表，关联字段
            // return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');

            return $this->belongsToMany('App\Role');
        }
    }

上述定义了一个用户属于多个角色，一旦该关系确立，就可以查询了

    $user = App\User::find(1);
    foreach ($user->roles as $role) {
        //
    }

    $roles = App\User::find(1)->roles()->orderBy('name')->get();

#### 反向关联关系

反向关系与正向关系实现一样

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Role extends Model
    {
        /**
         * The users that belong to the role.
         */
        public function users()
        {
            return $this->belongsToMany('App\User');
        }
    }

#### 检索中间表的列值

对多对多关系来说，引入了一个中间表，因此需要有方法能够查询到中间表的列值，比如关系确立的时间等，使用`pivot`属性查询中间表
    
    $user = App\User::find(1);
    
    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

上述代码访问了中间表的`created_at`字段。

注意的是，默认情况下之后模型的键可以通过`pivot`对象进行访问，如果中间表包含了额外的属性，在指定关联关系的时候，需要使用`withPivot`方法明确的指定列名

    return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

如果希望中间表自动维护`created_at`和`updated_at`字段的话,需要使用`withTimestamps()`

    return $this->belongsToMany('App\Role')->withTimestamps();

### Has Many Through

这种关系比较强大，假设这样一个场景：Country模型下包含了多个User模型，而每个User模型又包含了多个Post模型，也就是说一个国家有很多用户，而这些用户都有很多帖子，我们希望查询某个国家的所有帖子，怎么实现呢，这就用到了Has Many Through关系

    countries
        id - integer
        name - string
    
    users
        id - integer
        country_id - integer
        name - string
    
    posts
        id - integer
        user_id - integer
        title - string

可以看到，posts表中并不直接包含country_id，但是它通过users表与countries表建立了关系

使用Has Many Through关系

    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Country extends Model
    {
        /**
         * Get all of the posts for the country.
         */
        public function posts()
        {
            // return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');

            return $this->hasManyThrough('App\Post', 'App\User');
        }
    }
    
方法`hasManyThrough`的第一个参数是我们希望访问的模型名称，第二个参数是中间模型名称。

    HasManyThrough hasManyThrough( 
        string $related, 
        string $through, 
        string|null $firstKey = null, 
        string|null $secondKey = null, 
        string|null $localKey = null
    )

### Polymorphic Relations (多态关联)

多态关联使得同一个模型使用一个关联就可以属于多个不同的模型，假设这样一个场景，我们有一个帖子表和一个评论表，用户既可以对帖子执行喜欢操作，也可以对评论执行喜欢操作，这样的情况下该怎么处理呢？

表结构如下

    posts
        id - integer
        title - string
        body - text
    
    comments
        id - integer
        post_id - integer
        body - text
    
    likes
        id - integer
        likeable_id - integer
        likeable_type - string

可以看到，我们使用likes表中的likeable_type字段判断该记录喜欢的是帖子还是评论，表结构有了，接下来就该定义模型了

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Like extends Model
    {
        /**
         * Get all of the owning likeable models.
         */
        public function likeable()
        {
            return $this->morphTo();
        }
    }
    
    class Post extends Model
    {
        /**
         * Get all of the product's likes.
         */
        public function likes()
        {
            return $this->morphMany('App\Like', 'likeable');
        }
    }
    
    class Comment extends Model
    {
        /**
         * Get all of the comment's likes.
         */
        public function likes()
        {
            return $this->morphMany('App\Like', 'likeable');
        }
    }

默认情况下，`likeable_type`的类型是关联的模型的完整名称，比如这里就是`App\Post`和`App\Comment`。

通常情况下我们可能会使用自定义的值标识关联的表名，因此，这就需要自定义这个值了，我们需要在项目的服务提供者对象的`boot`方法中注册关联关系，比如`AppServiceProvider`的`boot`方法中

    use Illuminate\Database\Eloquent\Relations\Relation;
    
    Relation::morphMap([
        'posts' => App\Post::class,
        'likes' => App\Like::class,
    ]);




#### 检索多态关系

访问一个帖子所有的喜欢

    $post = App\Post::find(1);  
    foreach ($post->likes as $like) {
        //
    }

访问一个喜欢的帖子或者评论
    
    $like = App\Like::find(1);   
    $likeable = $like->likeable;

上面的例子中，返回的likeable会根据该记录的类型返回帖子或者评论。

### 多对多的多态关联

多对多的关联使用方法`morphToMany`和`morphedByMany`，这里就不多废话了。

## 关联关系查询

在Eloquent中，所有的关系都是使用函数定义的，可以在不执行关联查询的情况下获取关联的实例。假设我们有一个博客系统，User模型关联了很多Post模型：

    /**
     * Get all of the posts for the user.
     */
    public function posts()
    {
       return $this->hasMany('App\Post');
    }

你可以像下面这样查询关联并且添加额外的约束

    $user = App\User::find(1);
    $user->posts()->where('active', 1)->get();

如果不需要对关联的属性添加约束，可以直接作为模型的属性访问，例如上面的例子，我们可以使用下面的方式访问User的Post

    $user = App\User::find(1);
    foreach ($user->posts as $post) {
        //
    }
    
动态的属性都是延迟加载的，它们只有在被访问的时候才会去查询数据库，与之对应的是预加载，预加载可以使用关联查询出所有数据，减少执行sql的数量。

### 查询关系存在性

使用`has`方法可以基于关系的存在性返回结果

    // 检索至少有一个评论的所有帖子...
    $posts = App\Post::has('comments')->get();

    // Retrieve all posts that have three or more comments...
    $posts = Post::has('comments', '>=', 3)->get();
    // Retrieve all posts that have at least one comment with votes...
    $posts = Post::has('comments.votes')->get();

如果需要更加强大的功能，可以使用`whereHas`和`orWhereHas`方法，把where条件放到`has`语句中。

    // 检索所有至少存在一个匹配foo%的评论的帖子
    $posts = Post::whereHas('comments', function ($query) {
        $query->where('content', 'like', 'foo%');
    })->get();

### 预加载

在访问Eloquent模型的时候，默认情况下所有的关联关系都是延迟加载的，在使用的时候才会开始加载，这就造成了需要执行大量的sql的问题，使用预加载功能可以使用关联查询出所有结果

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Book extends Model
    {
        /**
         * Get the author that wrote the book.
         */
        public function author()
        {
            return $this->belongsTo('App\Author');
        }
    }

接下来我们检索所有的书和他们的作者

    $books = App\Book::all();
    
    foreach ($books as $book) {
        echo $book->author->name;
    }

上面的查询将会执行一个查询查询出所有的书，然后在遍历的时候再执行N个查询查询出作者信息，显然这样做是非常低效的，幸好我们还有预加载功能，可以将这N+1个查询减少到2个查询，在查询的时候，可以使用`with`方法指定哪个关系需要预加载。

    $books = App\Book::with('author')->get();
    foreach ($books as $book) {
        echo $book->author->name;
    }

对于该操作，会执行下列两个sql

    select * from books
    select * from authors where id in (1, 2, 3, 4, 5, ...)

预加载多个关系

    $books = App\Book::with('author', 'publisher')->get();

嵌套的预加载

    $books = App\Book::with('author.contacts')->get();

### 带约束的预加载

    $users = App\User::with(['posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }])->get();

    $users = App\User::with(['posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

### 延迟预加载

有时候，在上级模型已经检索出来之后，可能会需要预加载关联数据，可以使用`load`方法

    $books = App\Book::all();
    if ($someCondition) {
        $books->load('author', 'publisher');
    }
    
    $books->load(['author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }]);


## 关联模型插入

### save方法

保存单个关联模型

    $comment = new App\Comment(['message' => 'A new comment.']);
    $post = App\Post::find(1);
    $post->comments()->save($comment);

保存多个关联模型

    $post = App\Post::find(1); 
    $post->comments()->saveMany([
        new App\Comment(['message' => 'A new comment.']),
        new App\Comment(['message' => 'Another comment.']),
    ]);

### save方法和多对多关联

多对多关联可以为save的第二个参数指定关联表中的属性

    App\User::find(1)->roles()->save($role, ['expires' => $expires]);

上述代码会更新中间表的expires字段。

### create方法

使用`create`方法与`save`方法的不同在于它是使用数组的形式创建关联模型的

    $post = App\Post::find(1);
    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

### 更新 "Belongs To" 关系

更新`belongsTo`关系的时候，可以使用`associate`方法，该方法会设置子模型的外键

    $account = App\Account::find(10);
    $user->account()->associate($account);
    $user->save();

要移除`belongsTo`关系的话，使用`dissociate`方法

    $user->account()->dissociate();
    $user->save();
    
### Many to Many 关系

#### 中间表查询条件

当查询时需要对使用中间表作为查询条件时，可以使用`wherePivot`， `wherePivotIn`，`orWherePivot`，`orWherePivotIn`添加查询条件。

    $enterprise->with(['favorites' => function($query) {
        $query->wherePivot('enterprise_id', '=', 12)->select('id');
    }]);



#### Attaching / Detaching

    $user = App\User::find(1);
    // 为用户添加角色
    $user->roles()->attach($roleId);
    // 为用户添加角色，更新中间表的expires字段
    $user->roles()->attach($roleId, ['expires' => $expires]);
    
    // 移除用户的单个角色
    $user->roles()->detach($roleId);
    // 移除用户的所有角色
    $user->roles()->detach();

`attach`和`detach`方法支持数组参数，同时添加和移除多个

    $user = App\User::find(1);
    $user->roles()->detach([1, 2, 3]);
    $user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

#### 更新中间表（关联表）字段

使用`updateExistingPivot`方法更新中间表

    $user = App\User::find(1);
    $user->roles()->updateExistingPivot($roleId, $attributes);

#### 同步中间表（同步关联关系）

使用`sync`方法，可以指定两个模型之间只存在指定的关联关系

    $user->roles()->sync([1, 2, 3]);
    $user->roles()->sync([1 => ['expires' => true], 2, 3]);

上述两个方法都会让用户只存在1，2，3三个角色，如果用户之前存在其他角色，则会被删除。

### 更新父模型的时间戳

假设场景如下，我们为一个帖子增加了一个新的评论，我们希望这个时候帖子的更新时间会相应的改变，这种行为在Eloquent中是非常容易实现的。

在子模型中使用`$touches`属性实现该功能

    <?php
    
    namespace App;
    
    use Illuminate\Database\Eloquent\Model;
    
    class Comment extends Model
    {
        /**
         * All of the relationships to be touched.
         *
         * @var array
         */
        protected $touches = ['post'];
    
        /**
         * Get the post that the comment belongs to.
         */
        public function post()
        {
            return $this->belongsTo('App\Post');
        }
    }

现在，更新评论的时候，帖子的`updated_at`字段也会被更新

    $comment = App\Comment::find(1);
    $comment->text = 'Edit to this comment!';
    $comment->save();



----

参考： [Eloquent: Relationships](https://laravel.com/docs/5.2/eloquent-relationships)


