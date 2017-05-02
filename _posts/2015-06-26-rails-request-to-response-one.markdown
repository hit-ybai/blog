---
layout: post
title: "[Rails] 从Request到Response（1）"
date: 2015-06-26 18:55:27 +0800
comments: true
categories: 程序开发
tag:
- rake
- rails
- unicorn
- 学习报告
---
*本文翻译自：[Rails from Request to Response 系列](http://andrewberls.com/blog/post/rails-from-request-to-response-part-1--introduction)；个人选择了自己感兴趣的部分进行翻译，需要阅读原文的同学请戳前面的链接。*

## 第一部分 导言（Introduction）
### 服务器
在讲 Rails 调用栈之前，先简单介绍一下不同服务器应用的作用，其中并不会涉及到各个服务器应用（比如`Thin`和`Unicorn`或`Nginx`）的细节，因为文章的重点是讲 Rails 端的一些东西。

这里举一个`Unicorn`的简单例子，管窥整个 Rails 应用。

### Unicorn架构
`Unicorn`是一个实现了[Rack](http://rack.github.io/)接口的服务器应用，通过多个`Worker`并行处理请求（request）。启动时，主进程会将 Rails App 代码加在到内存中，随后以加载进来内存为原料进行复制，生成一定数量的`Worker`，并对他们进行监控和信号捕获（比如被用作关闭和重启的QUIT，TERM，USR1信号等等）。这些`Worker`负责处理的一个个真实的Web请求（request）。

下图是`Unicorn`的架构（[这幅图片](https://github.com/blog/517-unicorn)来自Github一篇很棒的文章）：
![Unicorn Architecture](http://7xjra1.com1.z0.glb.clouddn.com/unicorn_architecture.png)

这些`Worker`从共享的`Socket`中读取HTTP请求（request），并将它们发给Rails应用。随后得到响应（response），写回到共享的`Socket`中。这个过程中的大部份都发生在Unicorn`HttpServer`类的`#process_client`方法中，下面是相关部分的代码：
```ruby
# unicorn/lib/unicorn/http_server.rb

def process_client(client)
  status, headers, body = @app.call(env = @request.read(client))

  ...

  http_response_write(client, status, headers, body,
                      @request.response_start_sent)

  client.shutdown
  client.close
rescue => e
  handle_error(client, e)
end
```
*其中省略了一些关于[HTTP 100](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.1.1)状态处理和[Rack socket hijacking](http://blog.phusion.nl/2013/01/23/the-new-rack-socket-hijacking-api/)相关的代码，感兴趣的话可以阅读[完整版本](https://github.com/defunkt/unicorn/blob/728b2c70cda7787a80303c6fa2c2530dcb490c90/lib/unicorn/http_server.rb#L570)。*

我们可以看到，这个方法的核心逻辑相当的简明！

第一行是[Rack specification](https://github.com/rack/rack/blob/master/SPEC)：Rake App 其实就是一个 Ruby Object，我们只需要为它写一个可以接受 hash 参数`env`的`#call`方法，让它的返回`[status, headers, body]`（译者：还不是很明白`Rake`是什么鬼的同学可以去看一下这个[视频](http://railscasts-china.com/episodes/the-rails-initialization-process-by-kenshin54)，亲测好评）。

这个就是`Rails`，`Sinatra`，`Padrino`等那些兼容了Rake接口的框架的核心。回到`#process_client`方法，可以看到，我们向`@app`的`#call`方法传递 env 参数，并在 client 关闭之前，将响应（response）写回。

你没猜错，这个`@app`就是我们的 Rails 项目，我们来看他的声明：
```ruby
# blog/config/application.rb

module Blog
  class Application < Rails::Application
    ...
  end
end
```

这就是 Rails 调用栈的入口，但是如果你仔细观察，你会发现`#call`并没有定义在`Blog::Application`，而是被声明在了父类`Rails::Application`中。

现在开始，我们需要了解 Rails 应用的继承机制，以及一个请求（request）是如何在 Rails 内部被处理的。

### Rails Application and Engines
我们之前提到，整个 Rails 应用的入口`#call`被定义在了`Rails::Application`中，我们通过继承来使用它。这里是它的定义（[源码](https://github.com/rails/rails/blob/0dea33f770305f32ed7476f520f7c1ff17434fdc/railties/lib/rails/application.rb#L139)）：
```ruby
# rails/railties/lib/rails/application.rb

module Rails
  class Application < Engine

    # Implements call according to the Rack API. It simply
    # dispatches the request to the underlying middleware stack.
    def call(env)
      env["ORIGINAL_FULLPATH"] = build_original_fullpath(env)
      env["ORIGINAL_SCRIPT_NAME"] = env["SCRIPT_NAME"]
      super(env)
    end

    ...

  end
end
```
这里并没有什么东西，大部分功能通过调用`super`来执行。如果我们跟着代码看，可以发现`Rails::Application`类继承于`Rails::Engine`类。如果你熟悉[Rails engines](http://edgeguides.rubyonrails.org/engines.html)，你会惊喜地发现，`Rails::Application`就是一个超级`engine`!

我们来看看`Engine`类中的`#call`方法：
```ruby
# rails/railties/lib/rails/engine.rb

module Rails
  class Engine < Railtie

    def call(env)
      env.merge!(env_config)
      if env['SCRIPT_NAME']
        env.merge! "ROUTES_#{routes.object_id}_SCRIPT_NAME" => env['SCRIPT_NAME'].dup
      end
      app.call(env)
    end

    ...

  end
end
```
所以，`Engine`类继承于`Rails::Railtie`。通过观察[源码](https://github.com/rails/rails/blob/0dea33f770305f32ed7476f520f7c1ff17434fdc/railties/lib/rails/railtie.rb)，我们可以发现Railtie是Rails框架的核心，他为`initializers`，`config keys`，`generators`和`rake tasks`等提供钩子方法。

所有Rails的主要部件（`ActionMailer`，`ActionView`，`ActionController`和`ActiveRecord`）都是一个`Railtie`，这也就是为什么你可以随意拆装组合这些部件。

在`Engine`的`#call`方法中我们看到了`#call`的另一个代理方法（delegation），这里的`app`代表的是什么？在同一个文件中，我们发现了他的定义：
```ruby
# rails/railties/lib/rails/engine.rb

# Returns the underlying rack application for this engine.
def app
  @app ||= begin
    config.middleware = config.middleware.merge_into(default_middleware_stack)
    config.middleware.build(endpoint)
  end
end
```
现在我们到了一个牛逼的位置。这个engine构造了一个类似Rack application的中间件，并将`#call`方法代理（delegate）给它，他的终点是我们应用的`routes`，`ActionDispatch::Routing::RouteSet`类的一个实例。

Rack middleware可以用来「过滤」请求（request）和响应（response），并且可以将一个请求（request）处理过程分解成多个步骤，并视为一个「管道」进行处理，比如：处理权限认证，缓存等等。

你可以通过执行`rake middleware`来列出一个应用所使用的全部中间件。这里是我对 Blog 应用执行这条指令所得到的结果：
```markdown
$ RAILS_ENV=production rake middleware
use Rack::Sendfile
use #<ActiveSupport::Cache::Strategy::LocalCache::Middleware:0x007f7ffb206f20>
use Rack::Runtime
use Rack::MethodOverride
use ActionDispatch::RequestId
use Rails::Rack::Logger
use ActionDispatch::ShowExceptions
use ActionDispatch::DebugExceptions
use ActionDispatch::RemoteIp
use ActionDispatch::Callbacks
use ActiveRecord::ConnectionAdapters::ConnectionManagement
use ActiveRecord::QueryCache
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use ActionDispatch::Flash
use ActionDispatch::ParamsParser
use Rack::Head
use Rack::ConditionalGet
use Rack::ETag
run Blog::Application.routes
```

其中的大部分都不会讲，因为没有必要去了解全部这些中间件，即使一个请求（request），在到达`Blog::Application.routes`之前，会途径列表中从上到下所有的中间件。

到这里，我们已经完成了`App server / Rails application stack`的导言部分的介绍，接下来会着重介绍`Rails routing / dispatch stack`。

### 译者总结
我们可以讲`rake app`简化地表示为一个函数：
> ƒ: *env_set* → *{ [status, headers, body] }*


`Rails app`其实就是若干个函数的嵌套，逐层对输入的`env`和返回`[status, headers, body]`进行加工。整个Rails的执行过程，不过如此。

```
env(1) → env(2) → ... → env(i) → ... env(n)
```

令：`s: [status, headers, body]`
```
s(n) → s(n-1) → ... → s(i) → ... s(1)
```
看完这篇文章后，终于理解了[《Ruby社区应该去Rails化了》](http://robbinfan.com/blog/40/ruby-off-rails)中这段话的意思：

> ### Rails为何不适合做Web Service?
> 我发现了一个有意思的现象，最早的一批用Ruby开发Web Service服务的网站，都选择了用Rails开发，而在最近几年又不约而同抛弃Rails重写Web服务框架。当初用Rails的原因很简单，因为产品早期起步，不确定性很高，使用Rails快速开发，可以最大限度节约开发成本和时间。但为何当请求量变大以后，Rails不再适合了呢？

> 这主要是因为Rails本身是一个full-stack的Web框架，所有的设计目标就是为了开发Website，所以Rails框架封装过于厚重，对于需要更高性能更轻薄的Web Service应用场景来说，暴露出来了很多缺陷：
> #### Rails调用堆栈过深，URL请求处理性能很差

> Rails的设计目标是提供Web开发的 最佳实践 ，所以无论你需要不需要，Rails默认提供了开发Website所有可能的组件，但其中绝大部分你可能一辈子都用不上。例如Rails项目默认添加了20个middleware，但其中10个都是可以去掉的，我们自己的项目当中手工删除了这些middleware：

```ruby
config.middleware.delete 'Rack::Cache'   # 整页缓存，用不上
config.middleware.delete 'Rack::Lock'    # 多线程加锁，多进程模式下无意义
config.middleware.delete 'Rack::Runtime' # 记录X-Runtime（方便客户端查看执行时间）
config.middleware.delete 'ActionDispatch::RequestId' # 记录X-Request-Id（方便查看请求在群集中的哪台执行）
config.middleware.delete 'ActionDispatch::RemoteIp'  # IP SpoofAttack
config.middleware.delete 'ActionDispatch::Callbacks' # 在请求前后设置callback
config.middleware.delete 'ActionDispatch::Head'      # 如果是HEAD请求，按照GET请求执行，但是不返回body
config.middleware.delete 'Rack::ConditionalGet'      # HTTP客户端缓存才会使用
config.middleware.delete 'Rack::ETag'    # HTTP客户端缓存才会使用
config.middleware.delete 'ActionDispatch::BestStandardsSupport' # 设置X-UA-Compatible, 在nginx上设置
```

> 其中最夸张的是ActionDispatch::RequestIdmiddleware，只有在大型应用部署在群集环境下进行线上调试才可能用到的功能，有什么必要做成默认的功能呢？ Rails的哲学是：提供最全的功能集给你，如果你用不到，你自己手工一个一个关闭掉 ，但是这样带来的结果就是默认带了太多不必要的冗余功能，造成性能损耗极大。

> 我们看一个Ruby web框架请求处理性能评测 ，这个评测不访问数据库，也不测试并发性能，主要是测试框架处理URL请求路由，渲染文本，返回结果的处理速度。

> Rack: 1570.43 request/s
> Campig: 1166.16 request/s
> Sinatra: 912.81 request/s
> Padrino: 648.68 request/s
> Rails: 291.27 request/s

> Sinatra至少是Rails速度的3倍以上。