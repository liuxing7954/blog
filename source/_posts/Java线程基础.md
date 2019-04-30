---
title: Java线程基础
---
## 线程的基本状态

1. *初始化*(New)

   `调用线程的start方法`进入**阶段2**。

1. *可运行*(Runnable)

   `获得cpu时间片`进入**阶段3**。

1. *运行中*

   逻辑阶段，`时间片暂时用完`或者`调用yield`主动让出cpu（非强制性）回到**阶段2。**`等待用户输入`，或者`sleep` 会进入**阶段4**，`主动调用wait方法`，会进入**阶段5**。

1. *阻塞阶段*(Blocked)

   阻塞的原因`解除`会回到可运行阶段

1. *等待队列*(Waiting)

   这是一个`不争取锁`资源的队列，`直到wait超时`，或者被`notify`和`notifyall`方法唤醒，进入到**阶段6**。

1. *锁池*

   `等待争取锁`，争取到锁就进入**阶段2**。

## 线程的主要方法使用

#### Thread

- start

  实际调用的native方法start0.此方法会`真正的新起`一个线程去执行run方法

- run

  仅仅是调用的runable对象的`run方法`，和线程一点关系都没有

- yield

  告诉调度器，此线程可以`让出cpu时间`，但是是否会切换执行线程，要看调度器的执行，所以这是个`非强制的方法`。

## 获取线程的返回值方案

线程一般新建的方法是`new Thread（new Runnable（）{}）;`

这种集成自Runnable的线程实现`不支持返回值`

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

- Join方法

  给继承自Runnable的实现类一个成员属性，然后run方法中给属性赋值，主线程中获取该值前调用线程的`join方法`去等待线程执行完毕

  ```java
  public static class MyThread implements Runnable {
      String name;
      @Override
      public void run() {
          try {
              Thread.currentThread().sleep(5000);
              name = "jxy";
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
      }
  }
  ```

  ```java
  public static void main(String[] args) throws InterruptedException {
      MyThread target = new MyThread();
      Thread thread = new Thread(target);
      thread.start();
      thread.join();
      String name = target.name;
      System.out.println(name);
  }
  ```

- Callable接口返回

  ```java
  public static class MyCallable implements Callable {
      @Override
      public Object call() throws Exception {
          String name = "jxy";
          Thread.currentThread().sleep(1000);
          return name;
      }
  }
  ```

  ```java
  public static void main(String[] args) throws InterruptedException, ExecutionException {
      MyCallable myCallable = new MyCallable();
      FutureTask<String> futureTask = new FutureTask<String>(myCallable);
      new Thread(futureTask).start();
      String name = futureTask.get();
      System.out.println(name);
  }
  ```

## wait和sleep的区别

- wait是Object的方法，sleep是Thread的方法

- sleep方法让出cpu执行时间，`不释放锁`，wait`让出cpu时间和锁`

- wait让出锁的特性——所以只能在`synchronized代码块或者方法`中使用。

## 锁池和等待池

线程中调用无参的`wait方法`，那就会进入`无限期等待中`(Waiting)也叫`等待池`（WaitSet）在这里的线程，释放了锁和让出cpu之后，`不会去争取cpu时间`,除非被notify和notifyall`唤醒`，重新进去锁池去争取锁。

## notify和notifyall的区别

这两个方法的区别在于notify`随机唤醒`一个等待池中的一个等待线程。notifyall`唤醒全部`等待线程

## stop和interrupt的区别

stop不安全已经废弃

interrupt是一个中断的通知，具体的中断由线程本身来执行

```java
public static void main(String[] args) throws InterruptedException, ExecutionException {
    Thread thread = new Thread() {
        @Override
        public void run() {
            try {
                while (!isInterrupted()) {
                    System.out.println("thread 1");
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };
    thread.start();
    thread.interrupt();
}
```