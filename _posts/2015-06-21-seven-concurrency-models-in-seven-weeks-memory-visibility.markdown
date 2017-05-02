---
layout: post
title: "并发模型学习 (3)"
date: 2015-06-21 11:20:27 +0800
comments: true
categories: 读书报告
tags:
- 并发
- 并行
- 读书报告
---
#### 优化的副作用
![What is the meaning of my life?](http://7xjra1.com1.z0.glb.clouddn.com/the thinker.jpg)

由于没有通读过Ruby源码，无法确定这个Bug是否能用Ruby来复现，先用Java：

```java
public class Puzzle {
    static boolean answerReady = false;
    static int answer = 0;
    static Thread t1 = new Thread() {
        public void run() {
            answer = 42;
            answerReady = true;
        }
    };
    static Thread t2 = new Thread() {
        public void run() {
            if (answerReady)
                System.out.println("The meaning of life is: " + answer);
            else
                System.out.println("I don't know the answer");
        }
    };
    public static void main(String[] args) throws InterruptedException {
        t1.start(); t2.start();
        t1.join(); t2.join();
    }
}
```

根据线程执行的时序，这段代码的输出可能是：
```markdown
The meaning of life is: 42
```
或者是
```markdonw
I don't know the answer
```

但是还有一种结果可能是：
```markdown
The meaning of life is: 0
```

这说明了，当`answerReady`为`true`时`answer`可能为0！

就好像第六行和第七行颠倒了执行顺序。但是乱序执行是完全可能发生的：
1. 编译器的静态优化可以打乱代码的执行顺序（编译原理）
2. JVM的动态有话也会打乱代码的执行顺序（JVM）
3. 硬件可以通过乱序执行来优化其性能（计算机体系结构）

比乱序执行更糟糕的时，有时一个线程产生的修改可能对另一个线程不可见，

如果讲`run()`写成：
```java
        public void run() {
            while (!answerReady)
                Thread.sleep(100);
            System.out.println("The meaning of life is: " + answer);
        }
```
`answerReady`可能不会变成`true`代码运行后无法退出。

显然，我们需要一个明确的标准来告诉我们，优化会产生什么副作用影响，这就是`Java`内存模型(其他语言应该也有类似的东西)。Btw，经过本天才的多次试验，Ruby这边可以复现啦！！！

复现代码：
```ruby
require 'singleton'  

class Puzzle
  include Singleton

  def initialize
    @answer_ready = false
    @answer       = 0
  end

  def thread1
    Thread.new do
      @answer       = 'eat'
      @answer_ready = true
    end
  end

  def thread2
    Thread.new do
      meaning_of_life = "The meaning of life is: #{@answer}"
      no_answer       = "I don't know the answer"
      puts @answer_ready ? meaning_of_life : no_answer
    end
  end

  def main
    [thread1, thread2].each(&:join)
  end
end

Puzzle.instance.main
```

实验过程：
```bash
#!/bin/bash

while [ "$result" != "The meaning of life is: 0" ]; do
  result="$(ruby test.rb)"
  echo $result
done
```
实验结果：
```markdown
➜  /tmp ruby:(system: ruby 2.1.5p273)
$ sh test.sh
The meaning of life is: eat
The meaning of life is: eat
I don't know the answer
The meaning of life is: eat
I don't know the answer
I don't know the answer
The meaning of life is: 0
```

#### 内存可见性
`Java`内存模型定义了何时一个线程对内存的修改对另一个线程可见。基本原则是，如果读线程和写线程不进行同步，就不能保证可见性。

除了`increment`之外，`count`的`getter`方法也需要进行同步。否则`count`方法可能获得一个失效的值：对于前面交互的两个线程，`conter`在`join`之后调用因此是线程安全的。但这种设计为其他调用`conter`的方法埋下了隐患。

所以，「竞态条件」和「内存可见性」都可能让多线程程序运行结果出错。除此之外，还有一类问题：「死锁」。

### 推荐阅读
*[深入理解Java内存模型系列](http://www.infoq.com/cn/articles/java-memory-model-1)*
*[内存可见性](http://www.domaigne.com/blog/computing/mutex-and-memory-visibility/)*
