---
title: <font color=#0099ff face="微软雅黑">阻塞队列DealayQueue</font>
date: 2017-06-13 09:42:23
categories: java多线程
tags: [java,多线程,阻塞队列,DelayQUeue]
---

阻塞队列
===========
阻塞队列（BlockingQueue）是那些支持持阻塞的插入和移除的队列。
1）支持阻塞的插入方法：意思是当队列满时，队列会阻塞插入元素的线程，直到队列不满。
2）支持阻塞的移除方法：意思是在队列为空时，获取元素的线程会等待队列变为非空。

阻塞队列常用于生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。阻塞队列就是生产者用来存放元素、消费者用来获取元素的容器。

|方法/处理方式|抛出异常|返回特殊值|一直阻塞|超时退出|
|:------------|--------|----------|--------|--------|
|插入方法|add(e)|offer(e)|put(e)|ofer(e,time,unit)|
|移除方法|remove()|poll()|take()|poll(time,unit)|
|检查方法|element()|peek()|不可用|不可用|

·抛出异常：当队列满时，如果再往队列里插入元素，会抛出IllegalStateException（"Queuefull"）异常。当队列空时，从队列里获取元素会抛出NoSuchElementException异常。
·返回特殊值：当往队列插入元素时，会返回元素是否插入成功，成功返回true。如果是移除方法，则是从队列里取出一个元素，如果没有则返回null。
·一直阻塞：当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，直到队列可用或者响应中断退出。当队列空时，如果消费者线程从队列里take元素，队列会阻塞住消费者线程，直到队列不为空。
·超时退出：当阻塞队列满时，如果生产者线程往队列里插入元素，队列会阻塞生产者线程一段时间，如果超过了指定的时间，生产者线程就会退出。

**注意:**如果是无界阻塞队列，队列不可能会出现满的情况，所以使用put或offer方法永远不会被阻塞，而且使用offer方法时，该方法永远返回true。

**JDK 7中的阻塞队列：**

- ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。
- LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。
- PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。
- DelayQueue：一个使用优先级队列实现的无界阻塞队列。
- SynchronousQueue：一个不存储元素的阻塞队列。
- LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
- LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

DelayQUeue
===============
DelayQueue是一个无界的BlockingQueue，用于放置实现了Delayed接口的对象，队列使用PriorityQueue来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。这种队列是有序的，即队头对象的延迟到期时间最长。注意：不能将null元素放置到这种队列中。DelayQueue非常有用，可以将DelayQueue运用在以下应用场景：

- 缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素时，表示缓存有效期到了。
- 定时任务调度：使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，比如TimerQueue就是使用DelayQueue实现的。

Delayed一种混合风格的接口，用来标记那些应该在给定延迟时间之后执行的对象。此接口的实现必须定义一个 compareTo方法，该方法提供与此接口的getDelay方法一致的排序。

使用DelayQueue
--------------

**一、实现Delayed接口：**
DelayQueue队列的元素必须实现Delayed接口。我们可以参考ScheduledThreadPoolExecutor里ScheduledFutureTask类的实现,首先，在对象创建的时候，初始化基本数据。使用time记录当前对象延迟到什么时候可以使用，使用sequenceNumber来标识元素在队列中的先后顺序。代码如下：
```java
private static final AtomicLong sequencer = new AtomicLong(0);
Scheduled FutureTask(Runnable r, V result, long ns, long period) {
Scheduled FutureTask(Runnable r, V result, long ns, long period) {
            super(r, result);
            this.time = ns;
            this.period = period;
            this.sequence Number = sequencer.getAndIncrement();
}
```
第二步：实现getDelay方法，该方法返回当前元素还需要延时多长时间，单位是纳秒，代码如下。
```java
public long getDelay(TimeUnit unit) {
            return unit.convert(time - now(), Time Unit.NANOSECONDS);
        }
```
通过构造函数可以看出延迟时间参数ns的单位是纳秒，自己设计的时候最好使用纳秒，因为实现getDelay()方法时可以指定任意单位，一旦以秒或分作为单位，而延时时间又精确不到纳秒就麻烦了。使用时请注意当time小于当前时间时，getDelay会返回负数。
第三步：实现compareTo方法来指定元素的顺序。例如，让延时时间最长的放在队列的末尾。实现代码如下:
```java
public int compareTo(Delayed other) {
            if (other == this)// compare zero ONLY if same object
                return 0;
            if (other instanceof Scheduled FutureTask) {
                Scheduled FutureTask<> x = (Scheduled FutureTask<>)other;
                long diff = time - x.time;
                if (diff < 0)
                    return -1;
                else if (diff > 0)
                    return 1;
                else if (sequence Number < x.sequence Number)
                    return -1;
                else
                    return 1;
            }
            long d = (getDelay(Time Unit.NANOSECONDS) -
                        other.getDelay(Time Unit.NANOSECONDS));
            return (d == 0)  0 : ((d < 0)  -1 : 1);
        }
```
**二、实现延时阻塞队列**
延时阻塞队列的实现很简单，当消费者从队列里获取元素时，如果元素没有达到延时时间，就阻塞当前线程。
```java
long delay = first.getDelay(Time Unit.NANOSECONDS);
if (delay <= 0)
    return q.poll();
else if (leader != null)
        available.await();
else {
    Thread thisThread = Thread.currentThread();
    leader = thisThread;
        try {
                available.awaitNanos(delay);
            } finally {
                if (leader == thisThread)
                leader = null;
            }
    }
```
代码中的变量leader是一个等待获取队列头部元素的线程。如果leader不等于空，表示已经有线程在等待获取队列的头元素。所以，使用await()方法让当前线程等待信号。如果leader等于空，则把当前线程设置成leader，并使用awaitNanos()方法让当前线程等待接收信号或等待delay时间。

下面实现两种具体场景：
--------------
1、模拟一个考试的日子，考试时间为120分钟，30分钟后才可交卷，当时间到了，或学生都交完卷了考试结束。这个场景中几个点需要注意：

- 考试时间为120分钟，30分钟后才可交卷，初始化考生完成试卷时间最小应为30分钟
- 对于能够在120分钟内交卷的考生，如何实现这些考生交卷
- 对于120分钟内没有完成考试的考生，在120分钟考试时间到后需要让他们强制交卷
- 在所有的考生都交完卷后，需要将控制线程关闭

实现思想：用DelayQueue存储考生（Student类），每一个考生都有自己的名字和完成试卷的时间，Teacher线程对DelayQueue进行监控，收取完成试卷小于120分钟的学生的试卷。当考试时间120分钟到时，先关闭Teacher线程，然后强制DelayQueue中还存在的考生交卷。每一个考生交卷都会进行一次countDownLatch.countDown()，当countDownLatch.await()不再阻塞说明所有考生都交完卷了，而后结束考试。

Student类实现Runnable和Delayed接口，之后就可以存入DelayQueue中去了：
```java
class Student implements Runnable,Delayed{

    private String name;
    private long workTime;
    private long submitTime;
    private boolean isForce = false;
    private CountDownLatch countDownLatch;
    
    public Student(){}
    
    public Student(String name,long workTime,CountDownLatch countDownLatch){
        this.name = name;
        this.workTime = workTime;
        this.submitTime = TimeUnit.NANOSECONDS.convert(workTime, 
                                        TimeUnit.NANOSECONDS)+System.nanoTime();
        this.countDownLatch = countDownLatch;
    }
    
    @Override
    public int compareTo(Delayed o) {
        // TODO Auto-generated method stub
        if(o == null || ! (o instanceof Student)) return 1;
        if(o == this) return 0; 
        Student s = (Student)o;
        if (this.workTime > s.workTime) {
            return 1;
        }else if (this.workTime == s.workTime) {
            return 0;
        }else {
            return -1;
        }
    }

    @Override
    public long getDelay(TimeUnit unit) {
        // TODO Auto-generated method stub
        return unit.convert(submitTime - System.nanoTime(),  TimeUnit.NANOSECONDS);
    }

    @Override
    public void run() {
        // TODO Auto-generated method stub
        if (isForce) {
            System.out.println(name + " 交卷, 希望用时" + workTime + "分钟"+" ,实际用时 120分钟" );
        }else {
            System.out.println(name + " 交卷, 希望用时" + workTime + 
                                                "分钟"+" ,实际用时 "+workTime +" 分钟");  
        }
        countDownLatch.countDown();
    }

    public boolean isForce() {
        return isForce;
    }

    public void setForce(boolean isForce) {
        this.isForce = isForce;
    }
    
}

```
Teacher类用来收取DelayQueue中时间到了的学生的试卷。也就是说一个学生如果用时大于30分钟小于120分钟，那么当时间到了的时候Teacheer类就会从QelayQueue中取出这个学生。
```java
class Teacher implements Runnable{

    private DelayQueue<Student> students;
    public Teacher(DelayQueue<Student> students){
        this.students = students;
    }
    
    @Override
    public void run() {
        // TODO Auto-generated method stub
        try {
            System.out.println(" test start");
            while(!Thread.interrupted()){
                students.take().run();
            }
        } catch (Exception e) {
            // TODO: handle exception
            e.printStackTrace();
        }
    }
    
}
```
EndExam类是强制交卷类，当考生用时超过120分钟就会强制从DelayQueue中取出来。
```java
class EndExam extends Student{

    private DelayQueue<Student> students;
    private CountDownLatch countDownLatch;
    private Thread teacherThread;
    
    public EndExam(DelayQueue<Student> students, long workTime, 
                        CountDownLatch countDownLatch,Thread teacherThread) {
        super("强制收卷", workTime,countDownLatch);
        this.students = students;
        this.countDownLatch = countDownLatch;
        this.teacherThread = teacherThread;
    }
    
    @Override
    public void run() {
        // TODO Auto-generated method stub
        
        teacherThread.interrupt();
        Student tmpStudent;
        for (Iterator<Student> iterator2 = students.iterator(); iterator2.hasNext();) {
            tmpStudent = iterator2.next();
            tmpStudent.setForce(true);
            tmpStudent.run();
        }
        countDownLatch.countDown();
    }
    
}
```
Exam是考试主类,包含一个main方法：
```java
public class Exam {

    public static void main(String[] args) throws InterruptedException {
        // TODO Auto-generated method stub
        int studentNumber = 20;
        CountDownLatch countDownLatch = new CountDownLatch(studentNumber+1);
        DelayQueue< Student> students = new DelayQueue<Student>();
        Random random = new Random();
        for (int i = 0; i < studentNumber; i++) {
            students.put(new Student("student"+(i+1), 30+random.nextInt(120),countDownLatch));
        }
        Thread teacherThread =new Thread(new Teacher(students)); 
        students.put(new EndExam(students, 120,countDownLatch,teacherThread));
        teacherThread.start();
        countDownLatch.await();
        System.out.println(" 考试时间到，全部交卷！");  
    }

}
```

**2、具有过期时间的缓存**
向缓存添加内容时，给每一个key设定过期时间，系统自动将超过过期时间的key清除。这个场景中几个点需要注意：

- 当向缓存中添加key-value对时，如果这个key在缓存中存在并且还没有过期，需要用这个key对应的新过期时间。
- 为了能够让DelayQueue将其已保存的key删除，需要重写实现Delayed接口可添加到DelayQueue的DelayedItem的hashCode函数和equals函数。
- 当缓存关闭，监控程序也应关闭，因而监控线程应当用守护线程。

Cache主类：
```java
public class Cache<K, V> {

    public ConcurrentHashMap<K, V> map = new ConcurrentHashMap<K, V>();
    public DelayQueue<DelayedItem<K>> queue = new DelayQueue<DelayedItem<K>>();
    
    public void put(K k,V v,long liveTime){
        V v2 = map.put(k, v);
        DelayedItem<K> tmpItem = new DelayedItem<K>(k, liveTime);
        if (v2 != null) {
            queue.remove(tmpItem);
        }
        queue.put(tmpItem);
    }
    
    public Cache(){
        Thread t = new Thread(){
            @Override
            public void run(){
                dameonCheckOverdueKey();
            }
        };
        t.setDaemon(true);
        t.start();
    }
    
    public void dameonCheckOverdueKey(){
        while (true) {
            DelayedItem<K> delayedItem = queue.poll();
            if (delayedItem != null) {
                map.remove(delayedItem.getT());
                System.out.println(System.nanoTime()+" remove "+
                                delayedItem.getT() +" from cache");
            }
            try {
                Thread.sleep(300);
            } catch (Exception e) {
                // TODO: handle exception
            }
        }
    }
    
    public static void main(String[] args) throws InterruptedException {
        Random random = new Random();
        int cacheNumber = 10;
        int liveTime = 0;
        Cache<String, Integer> cache = new Cache<String, Integer>();
        
        for (int i = 0; i < cacheNumber; i++) {
            liveTime = random.nextInt(3000);
            System.out.println(i+"  "+liveTime);
            cache.put(i+"", i, random.nextInt(liveTime));
            if (random.nextInt(cacheNumber) > 7) {
                liveTime = random.nextInt(3000);
                System.out.println(i+"  "+liveTime);
                cache.put(i+"", i, random.nextInt(liveTime));
            }
        }

        Thread.sleep(3000);
        System.out.println();
    }

}
```
DelayedItem类：
```java
class DelayedItem<T> implements Delayed{

    private T t;
    private long liveTime ;
    private long removeTime;
    
    public DelayedItem(T t,long liveTime){
        this.setT(t);
        this.liveTime = liveTime;
        this.removeTime = TimeUnit.NANOSECONDS.convert(liveTime, TimeUnit.NANOSECONDS) + 
                                        System.nanoTime();
    }
    
    @Override
    public int compareTo(Delayed o) {
        if (o == null) return 1;
        if (o == this) return  0;
        if (o instanceof DelayedItem){
            DelayedItem<T> tmpDelayedItem = (DelayedItem<T>)o;
            if (liveTime > tmpDelayedItem.liveTime ) {
                return 1;
            }else if (liveTime == tmpDelayedItem.liveTime) {
                return 0;
            }else {
                return -1;
            }
        }
        long diff = getDelay(TimeUnit.NANOSECONDS) - o.getDelay(TimeUnit.NANOSECONDS);
        return diff > 0 ? 1:diff == 0? 0:-1;
    }

    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(removeTime - System.nanoTime(), unit);
    }

    public T getT() {
        return t;
    }

    public void setT(T t) {
        this.t = t;
    }
    @Override
    public int hashCode(){
        return t.hashCode();
    }
    
    @Override
    public boolean equals(Object object){
        if (object instanceof DelayedItem) {
            return object.hashCode() == hashCode() ?true:false;
        }
        return false;
    }
    
}
```

