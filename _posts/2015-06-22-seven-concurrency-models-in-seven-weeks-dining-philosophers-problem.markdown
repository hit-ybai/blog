---
layout: post
title: "并发模型学习 (4)"
date: 2015-06-22 11:20:27 +0800
comments: true
categories: 读书报告
tags:
- 并发
- 并行
- 读书报告
---
####[哲学家用餐问题（Dining philosophers problem）](https://zh.wikipedia.org/zh-cn/%E5%93%B2%E5%AD%A6%E5%AE%B6%E5%B0%B1%E9%A4%90%E9%97%AE%E9%A2%98)
简单来说：有五个哲学家坐在一张圆桌上，每个人之间放着一只餐叉，这样桌上就有五只餐叉。哲学家只会做两件事，吃饭，或者思考。吃东西的时候，他们就停止思考，思考的时候也停止吃东西。
![来自Wikipedia](http://7xjra1.com1.z0.glb.clouddn.com/Dining_philosophers.png)

是的，他们每个人都会用到别人用过的餐叉，开不开心。

这个例子一般用来说明死锁问题，经典的场景之一：一名哲学家拿起了自己左手的餐叉，并为其加锁（以免同时被自己左边的哲学家拿到），而后等待自己右手的餐叉锁的释放。

然而，如果五个哲学家同时处于这个状态，就会死锁。

举个栗子：
`Chopstick.java`
```java
class Chopstick {
}
```
`Philosopher.java`
```java
import java.util.Random;

class Philosopher extends Thread {
    private int id;
    private Chopstick left, right;
    private Random random;

    public Philosopher(Chopstick left, Chopstick right, int id) {
        this.left  = left; this.right = right; this.id = id;
        random = new Random();
    }

    public void run() {
        try {
            while (true) {
                Thread.sleep( random.nextInt(1000) );     // Think for a while
                synchronized (left) {                    // Grab left chopstick
                    System.out.println("Philosopher#" + id + " take left Chopstick");

                    synchronized (right) {                 // Grab right chopstick
                        System.out.println("Philosopher#" + id + " take right Chopstick");
                        Thread.sleep( random.nextInt(1000) ); // Eat for a while
                    }
                    System.out.println("Philosopher#" + id + " put right chopsticks");
                }
                System.out.println("Philosopher#" + id + " put left chopsticks");
            }
        } catch (InterruptedException e) {}
    }
}
```
`DiningPhilosophers.java`
```java
import java.util.Random;

public class DiningPhilosophers {

    public static void main(String[] args) throws InterruptedException {
        Philosopher[] philosophers = new Philosopher[5];
        Chopstick[] chopsticks     = new Chopstick[5];

        for (int i = 0; i < 5; ++i)
            chopsticks[i] = new Chopstick();

        for (int i = 0; i < 5; ++i) {
            philosophers[i] = new Philosopher(chopsticks[i], chopsticks[(i + 1) % 5], i);
            philosophers[i].start();
        }

        for (int i = 0; i < 5; ++i)
            philosophers[i].join();
    }
}
```
这个栗子属于可以锁得死死的那种。

*未完待续*