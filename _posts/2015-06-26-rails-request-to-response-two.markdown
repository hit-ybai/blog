---
layout: post
title: "[Rails] 从Request到Response（2）"
date: 2015-06-28 18:55:27 +0800
comments: true
categories: 程序开发
tag:
- dispatcher
- rails
- route
- 学习报告
---
*本文翻译自：[Rails from Request to Response 系列](http://andrewberls.com/blog/post/rails-from-request-to-response-part-2--routing)；个人选择了自己感兴趣的部分进行翻译，需要阅读原文的同学请戳前面的链接。*

##第二部分 路由（Routing）

Blog::Application.routes 的定义也在 engine.rb 文件中：
```ruby
# rails/railties/lib/rails/engine.rb

def routes
  @routes ||= ActionDispatch::Routing::RouteSet.new
  @routes.append(&Proc.new) if block_given?
  @routes
end
```

Blog::Application.routes 会返回一个 ActionDispatch::Routing::RouteSet 的实例。这是我们第一次接触到 ActionDispatch 模块（module），它是 ActionPack 的一部分。

ActionDispatch ：
负责处理例如「路由（Routing）」，「参数解析（Parameter Parsing）」，「Cookies」和「Sessions」之类的工作；

ActionPack ：
Rails 源码的顶层的 gem 之一，功能包括：路由功能， controller （ActionController）的定义和 view （ActionView）的渲染；

RouteSet ：
此类是一个Route的集合，包括一个 route 的数组（Mapper），和一个 named route （NamedRouteCollection： name 对应着一组普通的 route 集合的Hash）。负责解释和分派请求（Request）到对应的控制器（Controller）动作（Action）。

现在继续来看 engine ，回到 RouteSet 的 #call 方法，相关的代码在 *actionpack/lib/action_dispatch/routing/route_set.rb* 中：
```ruby
# rails/actionpack/lib/action_dispatch/routing/route_set.rb

module ActionDispatch
  module Routing
    class RouteSet

      def call(env)
        @router.call(env)
      end

      ...

    end
  end
end
```
RouteSet 不会处理 @router 里的事情。如果我们看一下 [RouteSet constructor](https://github.com/rails/rails/blob/0dea33f770305f32ed7476f520f7c1ff17434fdc/actionpack/lib/action_dispatch/routing/route_set.rb#L299)，会发现 @router 其实是 Journey::Router 的一个实例。

###Journey
Journey 是 ActionDispatch 的核心路由模块。它的[代码](https://github.com/rails/rails/tree/0dea33f770305f32ed7476f520f7c1ff17434fdc/actionpack/lib/action_dispatch/journey)是非常有趣的，其中用到了广义状态转移图（generalized transition graph）和非确定性有限状态自动机（non-deterministic finite automata）来配对URLs和路由。

有兴趣的话可以拿出你的计算机科学与技术本科的教科书（译注：指的应是《形式语言与自动机》）复习一下！Journey 甚至还包含了一个完整的 [yacc 语法文件](https://github.com/rails/rails/blob/0dea33f770305f32ed7476f520f7c1ff17434fdc/actionpack/lib/action_dispatch/journey/parser.y)来对 routes 进行语法分析。

Journey::Router 的 #call 方法有些长，下面是相关部分：
```ruby
# rails/actionpack/lib/action_dispatch/journey/router.rb

module ActionDispatch
  module Journey
    class Router

      def call(env)
        find_routes(env).each do |match, parameters, route|
          env[@params_key] = (set_params || {}).merge parameters
          status, headers, body = route.app.call(env)
          return [status, headers, body]
        end
      end

      ...

    end
  end
end
```
我们看到了和 Unicorn 中 #process_client 一样的 Rack 接口。 #find_routes 通过路由表 GTG (generalized transition graph) simulator 对URL进行执行。路由表本身是由 config/routes.rb 中规定的路由信息，构造的一个 Journey::Routes 实例，它包含了全部的路由（多个 Journey::Route 实例），个人十分推荐通过 *RailsCasts* [#231](http://railscasts.com/episodes/231-routing-walkthrough)和[#232](http://railscasts.com/episodes/232-routing-walkthrough-part-2)来学习路由结构的更多细节。

倘若发现了匹配的路由，我们便调用与 route 相关联的 app 上的 #call 方法。只要我们现在在 Rails 源码中看到引用 app 的地方，就可以假定它是 Rack app 。其实每个 controller 中 action 的行为都和一个 Rack app 一样。与 route 相关联的 app 的初始化过程有一些 Tricky。

在构造路由表时， ActionDispatch::Routing::Mapper 类调用 add_route ，它将会构造 Journey::Route 的一个实例，与 controller endpoint  ——  the Rack app 相关联。

然而，在  mapper.rb  中这个类的定义有将近 2,000 行，以下是删减版的 Mapper#add_route 定义：
```ruby
# rails/actionpack/lib/action_dispatch/routing/mapper.rb

module ActionDispatch
  module Routing
    class Mapper

      def add_route(action, options)
        path = path_for_action(action, options.delete(:path))

        ...

        mapping = Mapping.new(@set, @scope, URI.parser.escape(path), options)
        app, conditions, requirements, defaults, as, anchor = mapping.to_route
        @set.add_route(app, conditions, requirements, defaults, as, anchor)
      end

      ...

    end
  end
end
```
在这里， @set 是我们之前提到的 RouteSet ，这里的 app 其实是 ActionDispatch::Routing::Dispatcher 的一个实例。 Dispatcher 类定义在 route_set.rb 中，这是 #call 的一个简单定义。

为了说的更清楚，想象一下我们正在通过URL /posts 访问我们的博客应用，它的路由指向了 Posts Controller 的 index 方法。

```ruby
# rails/actionpack/lib/action_dispatch/routing/route_set.rb

module ActionDispatch
  module Routing
    class Dispatcher

      def call(env)
        params = env[PARAMETERS_KEY]
        prepare_params!(params) 
        # params = {:action=>"index", :controller=>"posts"}

        controller = controller(params, @defaults.key?(:controller))
        # controller = PostsController

        dispatch(controller, params[:action], env)
      end

      ...

    end
  end
end
```

在这里， controller 方法已经解析了控制器参数（比如 posts ），并且给了我们一个类（PostsController）的引用，得到了基本的参数哈希（params hash）。

现在，已经准备好了去调用控制器（controller）上的方法（aciton），来得到完整的响应（response）。以下是 dispatch 方法的定义：

```ruby
# rails/actionpack/lib/action_dispatch/routing/route_set.rb
  
def dispatch(controller, action, env)
  controller.action(action).call(env)
end
```

到这里，我们成功地将 route 映射到了一个 controller/action ，作为下一个 post 处理的请求（request）被压入ActionController工作栈。

路由是 Rails 中一个相当复杂的一个部分，他的方法链很长。像之前所说的，我们并没有必要了解它的全部细节。Rails 具有魔力的地方就是它为你隐藏了所有这些细节。当然去探索这整个执行过程到底是如何发生的是一件很有趣的事情。

###译者总结：
实际上每个存在的 Controller 和 Action 的组合都是一个 Rack App ，假设它们存在于一个集合 rack_app_set 中；同时假设所有系统可接受的合法URL的集合为 URL_set 。

则路由为函数：
> ƒ: *URL_set* → *rack_app_set*

