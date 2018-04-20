---
title: <font color=#0099ff face="微软雅黑">阻塞队列 PriorityBlockingQueue</font>
date: 2017-06-14 09:37:23
categories: java多线程
tags: [java,多线程,阻塞队列,PriorityBlockingQueue]
---

PriorityBlockingQueue是一个基于优先级堆的无界的并发安全的优先级队列（FIFO），队列的元素按照其自然顺序进行排序，或者根据构造队列时提供的Comparator进行排序，具体取决于所使用的构造方法。PriorityBlockingQueue里面存储的对象必须是实现Comparable接口。队列通过这个接口的compare方法确定对象的priority。

PriorityBlockingQueue通过使用堆这种数据结构实现将队列中的元素按照某种排序规则进行排序，从而改变先进先出的队列顺序，提供开发者改变队列中元素的顺序的能力。队列中的元素必须是可比较的，即实现Comparable接口，或者在构建函数时提供可对队列元素进行比较的Comparator对象。

PriorityBlockingQueue通过内部组合PriorityQueue的方式实现优先级队列（private final PriorityQueue q;），另外在外层通过ReentrantLock实现线程安全，同时通过Condition实现阻塞唤醒。

具体实现请去看JDK源码。这里给出一篇源码分析的博客：[优先级对列PriorityBlockingQueue][1]

要说明的一点是：
**PriorityBlockingQueue中若多个元素的优先级相同，则其顺序是不固定的，可以采用二级比较方法来进一步排序。**

有没有人在使用PriorityBlockingQueue时，发现将添加在PriorityBlockingQueue的一系列元素打印出来，队列的元素其实并不是全部按优先级排序的，但是队列头的优先级肯定是最高的？
回复：这就是因为PriorityBlockingQueue使用了堆来进行排序。只保证头元素是优先级最高的。

利用PriorityBlockingQueue实现基于优先级的Executor类：
==========
使用Executor框架，只需要实现任务并将他们传递到执行器中，然后执行器将负责创建执行任务的线程，并执行这些线程。执行器内部使用一个阻塞队列存放等待执行的任务，并按任务到达执行器时的顺序进行存放。但是如果使用优先级队列存放任务，就可以使高优先级的任务先到达执行器，它会先被执行。

具体实现例子：
创建一个MyPriorityTask类，实现Runnable和Comparable类。
```java
import org.jetbrains.annotations.NotNull;
import java.util.concurrent.TimeUnit;

/**
 * Created by Catyee on 2017/6/12.
 */
public class MyPriorityTask implements Runnable, Comparable<MyPriorityTask>{

    private int priority;
    private String name;

    public MyPriorityTask(int priority, String name) {
        this.priority = priority;
        this.name = name;
    }

    public int getPriority() {
        return priority;
    }

    @Override
    public int compareTo(@NotNull MyPriorityTask o) {
        if(this.getPriority() < o.getPriority()){
            return 1;
        }
        if(this.getPriority() > o.getPriority()){
            return -1;
        }
        return 0;
    }

    @Override
    public void run() {
        System.out.printf("MyPriorityTask: %s Priority : %d\n", name, priority);
        try{
            TimeUnit.SECONDS.sleep(2);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}

```
这个类中重写了Comparable接口中的compareTo()方法，它接收一个MyPriorityTask对象作为参数，然后比较当前和参数对象的优先级值。让高优先级的任务先于低优先级的任务执行。同时重写了Runnable接口中的Run()方法，打印出信息并休眠2s。

创建一个Main主类，在里面声明TreadPoolExecutor，去执行任务：
```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.PriorityBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * Created by Catyee on 2017/6/12.
 */
public class Main {
    public static void main(String[] args){
        ThreadPoolExecutor executor = new ThreadPoolExecutor(2,2,1, TimeUnit.SECONDS, new PriorityBlockingQueue<Runnable>());
        for(int i = 0; i < 4; i++){
            MyPriorityTask task = new MyPriorityTask(i, "Task "+i);
            executor.execute(task);
        }
        try{
            TimeUnit.SECONDS.sleep(1);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        for(int i = 4; i < 8; i++){
            MyPriorityTask task = new MyPriorityTask(i, "Task "+i);
            executor.execute(task);
        }
        executor.shutdown();
        try{
            executor.awaitTermination(1, TimeUnit.DAYS);
        }catch (InterruptedException e){
            e.printStackTrace();
        }
        System.out.printf("Main: end of the program.\n");
    }
}
```
这里创建TreadPollExecutor对象的时候，要传入带Runnable泛型参数的PriorityBlockingQueue。先后创建了八个任务并赋予了不同的优先级，可以看到低优先级的先传入进线程池，但是会后于高优先级的任务执行。
执行结果：
```java
MyPriorityTask: Task 0 Priority : 0
MyPriorityTask: Task 1 Priority : 1
MyPriorityTask: Task 7 Priority : 7
MyPriorityTask: Task 6 Priority : 6
MyPriorityTask: Task 5 Priority : 5
MyPriorityTask: Task 4 Priority : 4
MyPriorityTask: Task 3 Priority : 3
MyPriorityTask: Task 2 Priority : 2
Main: end of the program.

Process finished with exit code 0
```
这里我们只创建了两个线程池执行器，当执行器空闲并等待任务时，第一批任务到达，它们将立即被执行。接下来，剩余任务基于他们优先级被依次执行。

  [1]: http://blog.sina.com.cn/s/blog_6145ed8101010q1y.html