---
title: Java CAS初识
date: 2018-07-22 14:45:10
tags:
---

多线程并发协同通讯一直都是不可避之的话题，尤其是现代多核处理器的发展更推动了关于这方面的研究。

在JAVA领域，JDK 5之前是靠synchronized关键字保证同步的，而这会导致产生比较重量级的锁，通常会导致线程阻塞、等待、唤醒。如果线程的这种状态切换比较频繁可能会加重CPU的负担，这样可能会阻碍真正有意义的处理。

而CAS这种算法的出现可以避免这些，CAS是从硬件层面作一些变量操作的原子性强控，这些并不会导致JAVA线程的阻塞，你也可以理解它其实也有一把锁，就是在比对更新的时候锁住，但是这把锁非常轻量化。 
关于CAS的一些详细原理、发展史、以及详细说明，这里不作解释，不懂可以度娘。

今天只是初识了下JAVA的**AtomicReference**

***以下是代码截图，同时附上了相关理解说***

```java
package test.atomic.reference;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicReference;

/**
 *

- 该用例意在说明用CAS原理基于一个预期的值作更新时（更新后的值与预期值不一样），

- 我们可以保证从读取判断到更新值这一个过程是原子方式进行操作的，

- 所以在多线程并发的时候只会有一个成功，因为后面的线程在作CAS时会发现预期值已经改变

- 这可以避免对同一个域进行多次写操作
  *

- 注意点：多线程并发做该操作只会有一个成功
  *
   */
  public class CompareAndSetTest {

  private static AtomicReference<Object> obj = new AtomicReference<>();

  public static void main(String[] args) {
      ExecutorService executorService = Executors.newFixedThreadPool(100);
      for (int i = 0; i < 100000; i++) {
          executorService.submit(new ObjReferenceHandler());
      }
      executorService.shutdown();
  }

  public static void printWrappedInfo(Object info, String type) {
      long id = Thread.currentThread().getId();
      System.out.println(id + ": [" + type + "] --> " + info);
  }

  public static void printBeforeObj(Object info) {
      printWrappedInfo(info, "before");
  }

  public static void printAfterObj(Object info) {
      printWrappedInfo(info, "after");
  }

  public static void printCASResult(Object info) {
      printWrappedInfo(info, "CAS");
  }


static class ObjReferenceHandler implements Runnable {
    @Override
    public void run() {
        Object beforeObj = new Object();
        printBeforeObj(beforeObj);
        boolean b = obj.compareAndSet(null, beforeObj);
        printCASResult(b);
        Object afterObj = obj.get();
        printAfterObj(afterObj);
    }
}

}
```



```java
package test.atomic.reference;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicReference;

/**
 *

- 该用例意在用CAS原理基于当前值作更新，

- 我们可以保证的是更新完成后，所作的更新是基于一个当前引用值作更新，而不是基于一个过期的引用值

- 事实上我们并不关心当前引用值的具体状态，我们能保证的是：并发更新时，从读取、到经过运算、再到更新引用变量，这个流程最终会通过CAS，

- 换名话说我们并不能保证这个流程一直是原子性的，但是终有一次通过了CAS，它一定是原子性的，而此时会中止尝试，因为已经成功以原子方式更新了引用值
  *

- 总的来说所有成功的操作才是有意义的，所有成功的操作在时间分片上一定是互不交叉的，最终保证了整体执行完成后，在最终效果上是同步的
  *

- 注意点：对当前值不作任何预测，只是作CAS直到成功
  *
   */
  public class UpdateAndGetTest {

  private static final AtomicReference<Integer> atomicInteger = new AtomicReference<>();

static {
    atomicInteger.set(0);
}

public static void main(String[] args) {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    for (int i = 0; i < 100; i++) {
        executorService.submit(new PlusNumber());
    }
    executorService.shutdown();
}

private static class PlusNumber implements Runnable {
    @Override
    public void run() {
        System.out.println(atomicInteger.updateAndGet(i -> i + 1));
    }
}


}
```

