# 程序猿必读-防范CSRF跨站请求伪造

![DSC00964](https://oayrssjpa.qnssl.com/2017-02-27-DSC00964.jpg?imageView2/2/w/600/h/1000/interlace/0/q/100)

CSRF（Cross-site request forgery，中文为**跨站请求伪造**）是一种利用网站可信用户的权限去执行未授权的命令的一种恶意攻击。通过**伪装可信用户的请求来利用信任该用户的网站**，这种攻击方式虽然不是很流行，但是却难以防范，其危害也不比其他安全漏洞小。

本文将简要介绍CSRF产生的原因以及利用方式，然后对如何避免这种攻击方式提供一些可供参考的方案，希望广大程序猿们都能够对这种攻击方式有所了解，避免自己开发的应用被别人利用。

> CSRF也称作**one-click attack**或者**session riding**，其简写有时候也会使用**XSRF**。

[TOC]

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 什么是CSRF？

简单点说，CSRF攻击就是 **攻击者利用受害者的身份，以受害者的名义发送恶意请求**。与XSS（Cross-site scripting，跨站脚本攻击）不同的是，XSS的目的是获取用户的身份信息，攻击者窃取到的是用户的身份（session/cookie），而CSRF则是利用用户当前的身份去做一些未经过授权的操作。

CSRF攻击最早在2001年被发现，由于它的请求是从用户的IP地址发起的，因此在服务器上的web日志中可能无法检测到是否受到了CSRF攻击，正是由于它的这种隐蔽性，很长时间以来都没有被公开的报告出来，直到2007年才真正的被人们所重视。

## CSRF有哪些危害

CSRF可以盗用受害者的身份，完成受害者在web浏览器有权限进行的任何操作，想想吧，能做的事情太多了。

- 以你的名义发送诈骗邮件，消息
- 用你的账号购买商品
- 用你的名义完成虚拟货币转账
- 泄露个人隐私
- ...

## 产生原理以及利用方式

要完成一个CSRF攻击，必须具备以下几个条件：

- 受害者已经登录到了目标网站（你的网站）并且没有退出
- 受害者有意或者无意的访问了攻击者发布的页面或者链接地址

![](https://oayrssjpa.qnssl.com/2017-02-27-14882028931608.jpg)

（图片来自网络，出处不明，百度来的😂）

整个步骤大致是这个样子的：

1. 用户小明在你的网站A上面登录了，A返回了一个session ID（使用cookie存储）
2. 小明的浏览器保持着在A网站的登录状态，事实上几乎所有的网站都是这样做的，一般至少是用户关闭浏览器之前用户的会话是不会结束的
3. 攻击者小强给小明发送了一个链接地址，小明打开了这个地址，查看了网页的内容
4. 小明在打开这个地址的时候，这个页面已经自动的对网站A发送了一个请求，这时候因为A网站没有退出，因此只要请求的地址是A的就会携带A的cookie信息，也就是使用A与小明之间的会话
5. 这时候A网站肯定是不知道这个请求其实是小强伪造的网页上发送的，而是误以为小明就是要这样操作，这样小强就可以随意的更改小明在A上的信息，以小明的身份在A网站上进行操作

### 利用方式

利用CSRF攻击，主要包含两种方式，一种是基于GET请求方式的利用，另一种是基于POST请求方式的利用。

#### GET请求利用

使用GET请求方式的利用是最简单的一种利用方式，其隐患的来源主要是由于在开发系统的时候没有按照HTTP动词的正确使用方式来使用造成的。**对于GET请求来说，它所发起的请求应该是只读的，不允许对网站的任何内容进行修改**。

但是事实上并不是如此，很多网站在开发的时候，研发人员错误的认为GET/POST的使用区别仅仅是在于发送请求的数据是在Body中还是在请求地址中，以及请求内容的大小不同。对于一些危险的操作比如删除文章，用户授权等允许使用GET方式发送请求，在请求参数中加上文章或者用户的ID，这样就造成了只要请求地址被调用，数据就会产生修改。

现在假设攻击者（用户ID=121）想将自己的身份添加为网站的管理员，他在网站A上面发了一个帖子，里面包含一张图片，其地址为`http://a.com/user/grant_super_user/121`

    <img src="http://a.com/user/grant_super_user/121" />

设想管理员看到这个帖子的时候，这个图片肯定会自动加载显示的。于是在管理员不知情的情况下，一个赋予用户管理员权限的操作已经悄悄的以他的身份执行了。这时候攻击者121就获取到了网站的管理员权限。


#### POST请求利用

相对于GET方式的利用，POST方式的利用更加复杂一些，难度也大了一些。攻击者需要伪造一个能够自动提交的表单来发送POST请求。

    <script>
    $(function() {
        $('#CSRF_forCSRFm').trigger('submit');
    });
    </script>
    <form action="http://a.com/user/grant_super_user" id="CSRF_form" method="post">
        <input name="uid" value="121" type="hidden">
    </form>

只要想办法实现用户访问的时候自动提交表单就可以了。

## 如何防范

### 防范原理

防范CSRF攻击，其实本质就是要求网站**能够识别出哪些请求是非正常用户主动发起的**。这就要求我们**在请求中嵌入一些额外的授权数据，让网站服务器能够区分出这些未授权的请求**，比如说在请求参数中添加一个字段，这个字段的值从登录用户的Cookie或者页面中获取的（这个字段的值必须对每个用户来说是随机的，不能有规律可循）。攻击者伪造请求的时候是无法获取页面中与登录用户有关的一个随机值或者用户当前cookie中的内容的，因此就可以避免这种攻击。

### 防范技术

#### Synchronizer token pattern

令牌同步模式（Synchronizer token pattern，简称STP）是在用户请求的页面中的所有表单中嵌入一个token，在服务端验证这个token的技术。token可以是任意的内容，但是一定要保证无法被攻击者猜测到或者查询到。攻击者在请求中无法使用正确的token，因此可以判断出未授权的请求。

#### Cookie-to-Header Token

对于使用Js作为主要交互技术的网站，将CSRF的token写入到cookie中

    Set-Cookie: CSRF-token=i8XNjC4b8KVok4uw5RftR38Wgp2BFwql; expires=Thu, 23-Jul-2015 10:25:33 GMT; Max-Age=31449600; Path=/

然后使用javascript读取token的值，在发送http请求的时候将其作为请求的header

    X-CSRF-Token: i8XNjC4b8KVok4uw5RftR38Wgp2BFwql

最后服务器验证请求头中的token是否合法。

#### 验证码

使用验证码可以杜绝CSRF攻击，但是这种方式要求每个请求都输入一个验证码，显然没有哪个网站愿意使用这种粗暴的方式，用户体验太差，用户会疯掉的。

### 简单实现STP

首先在index.php中，创建一个表单，在表单中，我们将session中存储的token放入到隐藏域，这样，表单提交的时候token会随表单一起提交

    <?php
    $token = sha1(uniqid(rand(), true));
    $_SESSION['token'] = $token;
    ?>
    <form action="buy.php" method="post">
        <input type="hidden" name="token" value="<?=$token; ?>" />
        ... 表单内容
    </form>

在服务端校验请求参数的`buy.php`中，对表单提交过来的token与session中存储的token进行比对，如果一致说明token是有效的

    <?php
    if ($_POST['token'] != $_SESSION['token']) {
        // TOKEN无效
        throw new \Exception('Token无效，请求为伪造请求');
    }
    // TOKEN有效，表单内容处理
    
对于攻击者来说，在伪造请求的时候是无法获取到用户页面中的这个`token`值的，因此就可以识别出其创建的伪造请求。

### 解析Laravel框架中的VerifyCSRFToken中间件

在Laravel框架中，使用了`VerifyCSRFToken`这个中间件来防范CSRF攻击。

在页面的表单中使用`{{ CSRF_field() }}`来生成token，该函数会在表单中添加一个名为`_token`的隐藏域，该隐藏域的值为Laravel生成的token，Laravel使用随机生成的40个字符作为防范CSRF攻击的token。

    $this->put('_token', Str::random(40));
    
如果请求是ajax异步请求，可以在`meta`标签中添加token

    <meta name="CSRF-token" content="{{ CSRF_token() }}">

使用`jquery`作为前端的框架时候，可以通过以下配置将该值添加到所有的异步请求头中

    $.ajaxSetup({
        headers: {
            'X-CSRF-TOKEN': $('meta[name="CSRF-token"]').attr('content')
        }
    });

在启用session的时候，Laravel会生成一个名为`_token`的值存储到session中。而使用前面两种方式在页面中加入的token就是使用的这一个值。在用户请求到来时，`VerifyCSRFToken`中间件会对符合条件的请求进行CSRF检查

    if (
      $this->isReading($request) ||
      $this->runningUnitTests() ||
      $this->shouldPassThrough($request) ||
      $this->tokensMatch($request)
    ) {
      return $this->addCookieToResponse($request, $next($request));
    }
    
    throw new TokenMismatchException;


在`if`语句中有四个条件，只要任何一个条件结果为`true`则任何该请求是合法的，否则就会抛出`TokenMismatchException`异常，告诉用户请求不合法，存在CSRF攻击。

第一个条件`$this->isReading($request)`用来检查请求是否会对数据产生修改

    protected function isReading($request)
    {
        return in_array($request->method(), ['HEAD', 'GET', 'OPTIONS']);
    }

这里判断了请求方式，如果是`HEAD`，`GET`，`OPTIONS`这三种请求方式则直接放行。你可能会感到疑惑，为什么GET请求也要放行呢？这是因为Laravel认为这三个请求都是请求查询数据的，**如果一个请求是使用GET方式，那无论请求多少次，无论请求参数如何，都不应该最数据做任何修改**。

第二个条件顾名思义是对单元测试进行放行，第三个是为开发者提供了一个可以对某些请求添加例外的功能，最后一个`$this->tokensMatch($request)`则是真正起作用的一个，它是Laravel防范CSRF攻击的关键

    $sessionToken = $request->session()->token();
    $token = $request->input('_token') ?: $request->header('X-CSRF-TOKEN');
    
    if (! $token && $header = $request->header('X-XSRF-TOKEN')) {
      $token = $this->encrypter->decrypt($header);
    }
    
    if (! is_string($sessionToken) || ! is_string($token)) {
      return false;
    }
    
    return hash_equals($sessionToken, $token);

Laravel会从请求中读取`_token`参数的的值，这个值就是在前面表单中添加的`CSRF_field()`函数生成的。如果请求是异步的，那么会读取`X-CSRF-TOKEN`请求头，从请求头中读取token的值。

最后使用`hash_equals`函数验证请求参数中提供的token值和session中存储的token值是否一致，如果一致则说明请求是合法的。

你可能注意到，这个检查过程中也会读取一个名为`X-XSRF-TOKEN`的请求头，这个值是为了提供对一些javascript框架的支持（比如Angular），它们会自动的对异步请求中添加该请求头，而该值是从Cookie中的`XSRF-TOKEN`中读取的，因此在每个请求结束的时候，Laravel会发送给客户端一个名为`XSRF-TOKEN`的Cookie值

    $response->headers->setCookie(
        new Cookie(
            'XSRF-TOKEN', $request->session()->token(), time() + 60 * $config['lifetime'],
            $config['path'], $config['domain'], $config['secure'], false
        )
    );

## 写在最后

本文只是对CSRF做了一个简单的介绍，主要是侧重于CSRF是什么以及如何应对CSRF攻击。有一个事实是我们无法回避的：**没有绝对安全的系统**，你有一千种防御对策，攻击者就有一千零一种攻击方式，但不管如何，我们都要尽最大的努力去将攻击者拦截在门外。如果希望深入了解如何发起一个CSRF攻击，可以参考一下这篇文章 [从零开始学CSRF](http://www.freebuf.com/articles/web/55965.html)。

作为一名web方向的研发人员，无论你是从事业务逻辑开发还是做单纯的技术研究，了解一些安全方面的知识都是很有必要的，多关注一些安全方向的动态，了解常见的攻击方式以及应对策略，必将在你成长为一名大牛的路上为你“推波助澜”。

本文将会持续修正和更新，最新内容请参考我的 [GITHUB](https://github.com/mylxsw) 上的 [程序猿成长计划](https://github.com/mylxsw/growing-up) 项目，欢迎 Star，更多精彩内容请 [follow me](https://github.com/mylxsw)。

## 参考

- [wikipedia: Cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery)
- [Cross-Site Request Forgeries](http://shiflett.org/articles/cross-site-request-forgeries)
- [浅谈CSRF攻击方式](http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html)


