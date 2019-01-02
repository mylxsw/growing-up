[TOC]

作为一名phper，在使用Lumen框架开发微服务的时候，API文档的书写总是少不了的，比较流行的方式是使用swagger来写API文档，但是与Java语言原生支持 annotation 不同，php只能单独维护一份swagger文档，或者在注释中添加annotations来实现类似的功能，但是注释中书写Swagger注解是非常痛苦的，没有代码提示，没有格式化。

本文将会告诉你如何借助phpstorm中annotations插件，在开发Lumen微服务项目时（Laravel项目和其它php项目方法类似）快速的在代码中使用注释来创建swagger文档。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 框架配置

我们使用当前最新的 Lumen 5.7 来演示。演示代码放到了github，感兴趣的可以参考一下

    https://github.com/mylxsw/lumen-swagger-demo

### 安装依赖

在Lumen项目中，首先需要使用 composer 安装SwaggerLume项目依赖

    composer require darkaonline/swagger-lume

![-w568](https://ssl.aicode.cc/15464000116329.jpg)

### 项目配置

在`bootstrap/app.php`文件中，去掉下面配置的注释（大约在26行），启用Facades支持。

    $app->withFacades();

启用`SwaggerLume` 项目的配置文件，在 `Register Container Bindings`部 分前面，添加

    $app->configure('swagger-lume');

然后，在 `Register Service Providers` 部分，注册 `SwaggerLume` 的ServiceProvider

    $app->register(\SwaggerLume\ServiceProvider::class);

在项目的根目录，执行命令 `php artisan swagger-lume:publish` 发布swagger相关的配置
    
![-w406](https://ssl.aicode.cc/15464004457617.jpg)

执行该命令后，主要体现以下几处变更

![-w617](https://ssl.aicode.cc/15464008291835.jpg)

- 在 `config/` 目录中，添加了项目的配置文件 `swagger-lume.php`
- 在 `resources/views/vendor` 目录中，生成了 `swagger-lume/index.blade.php` 视图文件，用于预览生成的API文档

从配置文件中我们可以获取以下关键信息

- **api.title** 生成的API文档显示标题
- **routes.api** 用于访问生成的API文档UI的路由地址默认为 `/api/documentation`
- **routes.docs** 用于访问生成的API文档原文，json格式，默认路由地址为 `/docs`
- **paths.docs** 和 **paths.docs_json** 组合生成 api-docs.json 文件的地址，默认为 `storage/api-docs/api-docs.json`，执行`php artisan swagger-lume:generate`命令时，将会生成该文件


## 语法自动提示

纯手写swagger注释肯定是要不得的，太容易出错，还需要不停的去翻看文档参考语法，因此我们很有必要安装一款能够自动提示注释中的注解语法的插件，我们常用的IDE是 **phpstorm**，在 phpstorm 中，需要安装 **PHP annotation** 插件

![](https://ssl.aicode.cc/15464033226895.jpg)

安装插件之后，我们在写Swagger文档时，就有代码自动提示功能了

![2019-01-02 13_33_59](https://ssl.aicode.cc/2019-01-02%2013_33_59.gif)


## 书写文档

Swagger文档中包含了很多与具体API无关的信息，我们在 `app/Http/Controllers` 中创建一个 `SwaggerController`，该控制器中我们不实现业务逻辑，只用来放置通用的文档信息

    <?php
    namespace App\Http\Controllers;
    
    use OpenApi\Annotations\Contact;
    use OpenApi\Annotations\Info;
    use OpenApi\Annotations\Property;
    use OpenApi\Annotations\Schema;
    use OpenApi\Annotations\Server;
    
    /**
     *
     * @Info(
     *     version="1.0.0",
     *     title="演示服务",
     *     description="这是演示服务，该文档提供了演示swagger api的功能",
     *     @Contact(
     *         email="mylxsw@aicode.cc",
     *         name="mylxsw"
     *     )
     * )
     *
     * @Server(
     *     url="http://localhost",
     *     description="开发环境",
     * )
     *
     * @Schema(
     *     schema="ApiResponse",
     *     type="object",
     *     description="响应实体，响应结果统一使用该结构",
     *     title="响应实体",
     *     @Property(
     *         property="code",
     *         type="string",
     *         description="响应代码"
     *     ),
     *     @Property(property="message", type="string", description="响应结果提示")
     * )
     *
     *
     * @package App\Http\Controllers
     */
    class SwaggerController
    {}
    
接下来，在业务逻辑控制器中，我们就可以写API了
    
    <?php
    namespace App\Http\Controllers;
    
    use App\Http\Responses\DemoAdditionalProperty;
    use App\Http\Responses\DemoResp;
    use Illuminate\Http\Request;
    use OpenApi\Annotations\Get;
    use OpenApi\Annotations\MediaType;
    use OpenApi\Annotations\Property;
    use OpenApi\Annotations\RequestBody;
    use OpenApi\Annotations\Response;
    use OpenApi\Annotations\Schema;
    
    class ExampleController extends Controller
    {
    
        /**
         * @Get(
         *     path="/demo",
         *     tags={"演示"},
         *     summary="演示API",
         *     @RequestBody(
         *         @MediaType(
         *             mediaType="application/json",
         *             @Schema(
         *                 required={"name", "age"},
         *                 @Property(property="name", type="string", description="姓名"),
         *                 @Property(property="age", type="integer", description="年龄"),
         *                 @Property(property="gender", type="string", description="性别")
         *             )
         *         )
         *     ),
         *     @Response(
         *         response="200",
         *         description="正常操作响应",
         *         @MediaType(
         *             mediaType="application/json",
         *             @Schema(
         *                 allOf={
         *                     @Schema(ref="#/components/schemas/ApiResponse"),
         *                     @Schema(
         *                         type="object",
         *                         @Property(property="data", ref="#/components/schemas/DemoResp")
         *                     )
         *                 }
         *             )
         *         )
         *     )
         * )
         *
         * @param Request $request
         *
         * @return DemoResp
         */
        public function example(Request $request)
        {
            // TODO 业务逻辑
    
            $resp         = new DemoResp();
            $resp->name   = $request->input('name');
            $resp->id     = 123;
            $resp->age    = $request->input('age');
            $resp->gender = $request->input('gender');
    
            $prop1        = new DemoAdditionalProperty();
            $prop1->key   = "foo";
            $prop1->value = "bar";
    
            $prop2        = new DemoAdditionalProperty();
            $prop2->key   = "foo2";
            $prop2->value = "bar2";
    
            $resp->properties = [$prop1, $prop2];
    
            return $resp;
        }
    }


这里，我们在响应结果中，引用了在SwaggerController中定义的 `ApiResponse`，还引用了一个没有定义的`ExampleResp`对象，我们可以 `app\Http\Responses` 目录（自己创建该目录）中实现该ExampleResp对象，我们将响应对象都放在这个目录中

    <?php
    
    namespace App\Http\Responses;
    
    use OpenApi\Annotations\Items;
    use OpenApi\Annotations\Property;
    use OpenApi\Annotations\Schema;
    
    /**
     * @Schema(
     *     title="demo响应内容",
     *     description="demo响应内容描述"
     * )
     *
     * @package App\Http\Responses
     */
    class DemoResp extends JsonResponse
    {
    
        /**
         * @Property(
         *     type="integer",
         *     description="ID"
         * )
         *
         * @var int
         */
        public $id = 0;
    
        /**
         * @Property(
         *     type="string",
         *     description="用户名"
         * )
         *
         * @var string
         */
        public $name;
    
        /**
         * @Property(
         *     type="integer",
         *     description="年龄"
         * )
         *
         * @var integer
         */
        public $age;
    
        /**
         * @Property(
         *     type="string",
         *     description="性别"
         * )
         *
         * @var string
         */
        public $gender;
    
        /**
         * @Property(
         *     type="array",
         *     @Items(ref="#/components/schemas/DemoAdditionalProperty")
         * )
         *
         * @var array
         */
        public $properties = [];
    }

返回对象引用其它对象
    
    <?php
    namespace App\Http\Responses;
    
    use OpenApi\Annotations\Property;
    use OpenApi\Annotations\Schema;
    
    /**
     *
     * @Schema(
     *     title="额外属性",
     *     description="额外属性描述"
     * )
     *
     * @package App\Http\Responses
     */
    class DemoAdditionalProperty
    {
        /**
         * @Property(
         *     type="string",
         *     description="KEY"
         * )
         *
         * @var string
         */
        public $key;
    
        /**
         * @Property(
         *     type="string",
         *     description="VALUE"
         * )
         *
         * @var string
         */
        public $value;
    }
    
## 生成文档

执行下面的命令，就可以生成文档了，生成的文档在`storage/api-docs/api-docs.json`。

    php artisan swagger-lume:generate

## 预览文档

打开浏览器访问 [http://访问地址/docs](http://localhost:8888/docs)，可以看到如下内容

![](https://ssl.aicode.cc/15464113862333.jpg)

访问 [http://访问地址/api/documentation](http://localhost:8888/api/documentation)，我们看到

![](https://ssl.aicode.cc/15464142787969.jpg)

接口详细信息展开

![](https://ssl.aicode.cc/15464143250849.jpg)

## 更多

本文简述了如何在Lumen项目中使用代码注释自动生成Swagger文档，并配合phpstorm的代码提示功能，然而，学会了这些还远远不够，你还需要去了解Swagger文档的语法结构，在 [swagger-php](https://github.com/zircote/swagger-php) 项目的 **Examples** 目录中包含很多使用范例，你可以参考一下。

团队项目中使用了swagger文档，但是总得有个地方管理文档吧，这里推荐一下 [Wizard](https://github.com/mylxsw/wizard) 项目，该项目是一款用于团队协作的文档管理工具，支持Markdown文档和Swagger文档，感兴趣的不妨尝试一下。
