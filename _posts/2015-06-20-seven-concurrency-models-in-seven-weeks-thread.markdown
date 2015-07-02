---
layout: post
title: "并发模型学习 (1)"
date: 2015-06-20 09:20:27 +0800
comments: true
categories: 读书报告
tags:
- 并发
- 并行
- 读书报告
---
### 互斥和内存模型
互斥：用锁保证某一时间仅有一个线程可以访问数据；
 可能带来的麻烦：竞态条件和死锁。
####线程
并发的基本单元：线程，可以讲线程看做控制流；
线程间通信方式：共享内存。
```ruby
def say_hi(name)
  puts "Hi #{name}!"
end

def say_hi_to_folks(folks)
  folks.inject([]) do |threads_array, name|
    threads_array << Thread.new { say_hi(name) }
  end.each(&:join)
end

say_hi_to_folks %w(Larry Jack)
```

执行结果有可能是
```markdown
Hi Larry!
Hi Jack!
```
或者是
```markdown
Hi Jack!
Hi Larry!
```

多线程的运行结果依赖于时序，多次运行结果并不稳定。

*注：原书使用的Java例子中，两个线程分别为主线程和子线程，由于Ruby中没有相对应的让出线程方法`Thread.yield()`，而在Ruby中相接近的`Thread.pass`实验效果又很差，故而改由两个子线程举例（那你为啥要用Ruby？ 因为不用编译啊笨蛋，不过用的都是JRuby*
