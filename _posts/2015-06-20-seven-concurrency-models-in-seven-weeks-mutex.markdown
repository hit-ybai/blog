---
layout: post
title: "并发模型学习 (2)"
date: 2015-06-20 11:20:27 +0800
comments: true
categories: 读书报告
tags:
- 并发
- 并行
- 读书报告
---
#### 锁儿
多个线程共享内存时，避免同时修改同一个部分内存造成的问题，需要用**锁**达到线程互斥的目的。某一时间，至多有一个线程持有锁。
```ruby
 class Counter
    def initialize
        @count = 0
    end
    def increment
        @count += 1
    end
    def count
        @count
    end
 end
counter = Counter.new

def thread(counter)
  10_000.times { counter.increment }
end

t1 = Thread.new { thread(counter) }
t2 = Thread.new { thread(counter) }

[t1, t2].each(&:join)
puts counter.count
```
执行结果
```bash
➜  /private/tmp ruby:(system: jruby 1.7.19)
$ ruby test.rb
13779
➜  /private/tmp ruby:(system: jruby 1.7.19)
$ ruby test.rb
16440
```
这段代码创建了一个`counter`对象和两个线程，每个线程调用`counter.increment` 10,000次。这段代码看上去很简单，但很脆弱。

几乎每次运行都将获得不同的结果，产生这个结果的原因是两个线程使用`counter.count`时发生了**竞态条件**（即代码行为取决于各操作的时序）

我们来看一下Java编译器是如何解释`++count`的。其字节码：
```
getfield #2 ;//获取count的值
iconst_1    ;//设置加数
iadd        ;//count加设置好的加数
putfield #2 ;//将更新的值写回count
```
这就是通称的*读-改-写*模式。

如果两个线程同时调用`increment`，线程1执行`getfield #2`，获取值42。在线程1执行其他动作之前，线程2也执行了`getfield #2`，获得值42。不过，现在两个线程都将获得的值加1，将43写回count中。导致`count`只被增加了一次。

竞态条件的解决方案：对`count`进行同步（synchronize）访问。
```ruby
class Counter
  attr_reader :count
  def initialize
    @count         = 0
    @counter_mutex = Mutex.new
  end

  def increment
    @counter_mutex.synchronize { @count += 1 }
  end
end

def thread(counter)
  10_000.times { counter.increment }
end

counter = Counter.new
t1 = Thread.new { thread(counter) }
t2 = Thread.new { thread(counter) }

[t1, t2].each(&:join)
puts counter.count
```
执行结果
```bash
➜  /private/tmp ruby:(system: jruby 1.7.19)
$ ruby test.rb
20000
```
线程进入`increment`方法时，获得`counter_mutex`锁，函数返回的时候释放该锁。同一时间最多有一个进程可以执行函数体，其他线程调用方法时将被阻塞，直到锁被释放。


