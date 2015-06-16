---
layout: post
title: "[Rails]「从零单排」[第零篇] 所谓MVC"
date: 2015-01-19 14:55:27
comments: true
categories: 程序开发
tag:
- MVC
- rails
- 从零单排
---
##何为MVC

为了明确一个`Web Application`中各个部分的职责，我们人为规定三个层级：Controller层，Model层和View层。这是一种设计上的解耦。

我们假设这三个层级分别对应code层面上的三个类：

```ruby
class Model
end
```
```ruby
class View
end
```
```ruby
class Controller
end
```
#### 完整的request生命周期
首先用户在Web Browser上输入一个URL，传输到Server端，每个合法的URL都会映射到Controller类的一个实例方法。

```ruby
# 我们可以设置 '/index' => Controller#index 来规定:
#   当用户在browser中输入"http://yoursiteurl.com/index"时
#   执行Controller类的index方法

class Controller
  def index
    render html: "<strong>Hello World!</strong>".html_safe
  end
end
```

Controller中的实例方法是一次`request`的入口，负责调度程序的各个部分，生成`web response`（输出）。

所谓入口，指的是一次访问的<b>整个生命周期，都随着这个方法的开始而开始，结束而结束。</b>从这个角度来看，我们完全可以不使用Model和View，就能完成一个`Web Application`。

####数据存取
我们可以看到，在一次request结束后，Controller实例方法中分配的内存会被回收，无法被再次使用。为了能够将Controller处理过程中的数据在今后继续使用，我们需要对这部分数据进行持久化存储。

所以我们期望有一个类，他的所有变量都永远不会消失。这个类，就叫做Model，至于它如何做到所有的变量永不消失，我们回头再讲。

```ruby
class Model
  # ...
end

class Controller
  def index
    Model.counter += 1
    render html: "<p>你是该网站的第#{Model.counter}名访客</p>".html_safe
  end
end
```

####页面展示
我们注意到，在上面的两个例子中都出现了类似`render html: xxx`之类的代码。如果我们不想在Controller中出现HTML代码，可以把它放在View层。

```ruby
class Model
  # ...
end

class View
  def self.index(counter)
    "<p>你是该网站的第#{counter}名访客</p>"
  end
end

class Controller
  def index
    Model.counter += 1
    render html: View.index(Model.counter).html_safe
  end
end
```
