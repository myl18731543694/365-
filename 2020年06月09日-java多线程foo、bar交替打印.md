## java多线程foo、bar交替打印

#### **1. 问题：**

提供如下的类

```java
class FooBar {
    private int n;

    public FooBar(int n) {
        this.n = n;
    }
    
    public void foo() {
        for (int i = 0; i < n; i++) {
            System.out.println("foo");
        }
    }

    public void bar() {
        for (int i = 0; i < n; i++) {
            System.out.println("bar");
        }
    }
}
```

两个不同的线程将会共用一个 FooBar 实例。其中一个线程将会调用 foo() 方法，另一个线程将会调用 bar() 方法。

请设计修改程序，以确保 "foobar" 被输出 n 次。

示例 1:

```
输入: n = 1
输出: "foobar"
解释: 这里有两个线程被异步启动。其中一个调用 foo() 方法, 另一个调用 bar() 方法，"foobar" 将被输出一次。
```


示例 2:

```
输入: n = 2
输出: "foobarfoobar"
解释: "foobar" 将被输出两次。
```



#### **2. 解决思路：**

- 使用condition实现精确唤醒，实现交替打印



#### **3. 代码：**

```java
package com.learn.othercode.learn.jdbcspace;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class FooBar {
    Lock lock = new ReentrantLock();
    Condition fooCondition = lock.newCondition();
    Condition barCondition = lock.newCondition();

    /**
     * 标识是不是foo最先开始，用来保证foo一定最先启动
     */
    Boolean isFooFirst = false;

    private int n;

    public FooBar(int n) {
        this.n = n;
    }

    public void foo() {
        lock.lock();
        try {
            // 标识foo已经是最先开始启动了
            isFooFirst = true;

            for (int i = 0; i < n; i++) {
                System.out.print("foo");
                barCondition.signal();
                fooCondition.await();
            }

            barCondition.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void bar() {
        lock.lock();
        try {
            while (!isFooFirst) {
                // 只要不是foo最开始，就等待
                fooCondition.signal();
                barCondition.await();
            }

            for (int i = 0; i < n; i++) {
                System.out.println("bar");
                fooCondition.signal();
                barCondition.await();
            }

            fooCondition.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

public class Test {
    public static void main(String[] args) throws InterruptedException {

        FooBar fooBar = new FooBar(5);
        new Thread(() -> fooBar.foo()).start();

        new Thread(() -> fooBar.bar()).start();
    }

}
```



#### **4. 疑难答案：**

**为什么要使用 `Boolean isFooFirst ` 保证foo在前面？**

因为线程启动是无序的，有可能是t2先启动，如果没有变量保证，那么顺序将会颠倒变成 bar、foo交替打印。

