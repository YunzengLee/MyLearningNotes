

# 1 什么是JUC

是java.util （工具包）下的concurrent包

在 Java 5.0 提供了 `java.util.concurrent`(简称JUC)包,在此包中增加了在并发编程中很常用的工具类,
用于定义类似于线程的自定义子系统,包括线程池,异步 IO 和轻量级任务框架;还提供了设计用于多线程上下文中
的 Collection 实现等;

普通的线程代码 使用Thread 

Runnable：没有返回值，效率比callable低

# 2 线程和进程

> 线程和进程

进程：一个程序，QQ.exe, Music.exe 程序的集合；

一个进程包含多个线程。java默认有几个线程？2个，main线程和gc线程。

线程：开了一个进程Typora，可以写字，自动保存（都是线程负责的）

java可以开启线程吗？

答：不可以，Thread的start方法是调用了一个名为start0的方法，start0是一个native方法，c++，java无法直接操作硬件。

```java
public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this);

        boolean started = false;
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }
//start0是一个native方法， c++写的
    private native void start0();
```



> 并发、并行

并发编程：并发、并行

并发：多线程操作一个资源

- CPU 只有一核，模拟出多线程，就在多个线程之间快速交替执行

并行：多个人一起行走

- CPU多核，多个线程同时执行

```java
package com.kuang.demo01;

public class Test1 {
    public static void main(String[] args) {
        //获取cpu核数
        System.out.println(Runtime.getRuntime().availableProcessors());
//        new Thread().start();
    }
}
```

并发编程的本质：**充分利用CPU的资源**



> 线程状态

6个状态：

```java
public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,

        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,

        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * 阻塞
         */
        BLOCKED,

        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,

        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }
```

> wait/sleep区别

1. 来自不同的类（wait来自Object类，sleep来自Thread类）,企业当中不会用sleep，而是用java.util.concurrent.TimeUnit.DAYS.sleep(x);

2. 锁的释放

   wait会释放，sleep不会

3. 使用的范围不同

   wait：必须在同步代码块中

   sleep：可以在任何地方睡

4. sleep必须传入时间参数，wait不一定。

5. 是否需要捕获异常

   wait不需要，sleep必须要捕获异常，因为可能发生超时等待的情况

# 3 lock锁

> synchronized：排队 锁

java.util.concurrent.locks 

## Interface Lock

- 所有已知实现类： 

  [ReentrantLock](../../../../java/util/concurrent/locks/ReentrantLock.html)，  [ReentrantReadWriteLock.ReadLock](../../../../java/util/concurrent/locks/ReentrantReadWriteLock.ReadLock.html)，  [ReentrantReadWriteLock.WriteLock](../../../../java/util/concurrent/locks/ReentrantReadWriteLock.WriteLock.html)

  用法：

  ```java
   Lock l = ...;
   l.lock();
   try {
     // access the resource protected by this lock
   } finally {
     l.unlock();
   }
  ```

- 公平锁：必须先来后到（所有因为该锁阻塞的线程，重新调度时按照先后顺序进行调度）

- **非公平锁：可以插队（默认）**（因为该锁阻塞的若干线程，重新调度时与阻塞的先后顺序无关，任何线程都可能先调度执行）

## synchronized与Lock的区别

1. synchronized是内置的java关键字，Lock是一个java类
2. synchronized基于JVM实现，Lock基于API层面实现
3. synchronized无法判断锁的状态，Lock可以判断是否拿到了锁
4. synchronized会自动释放锁，Lock必须手动释放，如果不释放，就可能会死锁
5. synchronized：线程1获得锁且阻塞的话，线程2会一直等；Lock就不一定会等待下去，Lock有个方法tryLock()（tryLock：可以传入参数设置等待锁的时间）
6. synchronized是可重入锁，非公平锁，不可以中断；Lock，可重入锁，可以中断锁的等待，非公平（可以自己设置）
7. synchronized：适合锁少量的代码同步问题，Lock适合锁大量的代码。（why？）

区别在于：中断等待锁，非公平锁设置，精准唤醒，判断是否拿到了锁，关键字和类

> 锁是什么，如何判断锁的是谁

#  4 生产者和消费者

面试手写：单例模式，排序算法，生产者消费者，死锁

## synchronized版：   wait  notify

```java
package com.kuang.pc;

/**
 * 线程之间的通信问题：生产者和消费者问题   等待唤醒 通知唤醒
 * 线程交替执行  A B同时操作同一个变量
 */
public class A {
    public static void main(String[] args) {
        Data data = new Data();

        new Thread(()->{for (int i =0;i<10;i++){
            try{
                data.increment();
            }catch (Exception e){
                e.printStackTrace();
            }
        }},"A").start();
        new Thread(()->{for (int i =0;i<10;i++){
            try{
                data.decrement();
            }catch (Exception e){
                e.printStackTrace();
            }
        }},"B").start();
    }
}

//判断等待 业务 通知
class Data{
    private int num = 0;

    //+1
    public synchronized void increment() throws InterruptedException {
        if(num!=0){
            //wait
            this.wait(); //synchronized会锁住当前对象，（当一个线程执行increment方法时，其他线程无法执行decrement方法）
                        // 但是wait方法将会使线程阻塞，并释放锁
        }
        num++;
        System.out.println(Thread.currentThread().getName()+"=>"+num);
        //notify other thread: work is done
        this.notifyAll();

    }
    //-1
    public synchronized void decrement() throws InterruptedException {
        if(num==0){
            //wait
            this.wait();
        }
        num -- ;
        System.out.println(Thread.currentThread().getName()+"=>"+num);
        //notify other thread:work is done
        this.notifyAll();
    }
}

```

- 问题：如果多加几个线程，比如两个线程调用increment，两个线程调用decrement，就会出现虚假唤醒问题。解决办法：if判断改为while判断（[什么是虚假唤醒？](https://blog.csdn.net/qq_39455116/article/details/87101633)）
- 虚假唤醒就是：假设两个消费者线程都进行了if判断发现num==0然后进入wait，某个生产者进行num+1后就把两个消费者都唤醒了，结果两个消费者线程都会从if判断处，继续往下执行（尽管只有一个线程会获得锁进行num-1操作，但是释放锁之后，另一个线程就会 获得锁，接着又进行num-1操作，这样num就减了两次变为负数）
- 如果将if改为while循环判断，那么被唤醒的第一个消费者线程拿到锁，while循环判断发现num不为0，就会进行了num-1，之后第二个消费者拿到锁，不会直接向下进行num-1操作，而是仍处于while循环中，就会发现num已经被减为0了，就会再次进入wait状态。

## JUC版的生产者和消费者

synchronized：wait，notify配套

Lock：await，signal配套

代码实现

```java
package com.kuang.pc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class B {
    public static void main(String[] args) {
        Data2 data = new Data2();
        new Thread(()->{for (int i =0;i<10;i++){
            try{
                data.increment();
            }catch (Exception e){
                e.printStackTrace();
            }
        }},"A").start();
        new Thread(()->{for (int i =0;i<10;i++){
            try{
                data.decrement();
            }catch (Exception e){
                e.printStackTrace();
            }
        }},"B").start();

        new Thread(()->{for (int i =0;i<10;i++){
            try{
                data.increment();
            }catch (Exception e){
                e.printStackTrace();
            }
        }},"C").start();
        new Thread(()->{for (int i =0;i<10;i++){
            try{
                data.decrement();
            }catch (Exception e){
                e.printStackTrace();
            }
        }},"D").start();


    }
}

//判断等待 业务 通知
class Data2{
    private int num = 0;
    Lock l = new ReentrantLock();
    Condition condition = l.newCondition();

    //+1
    public void increment() throws InterruptedException {
        //上锁
        l.lock();

        //业务代码
        while(num!=0){
            //wait
            condition.await();
        }
        num++;
        System.out.println(Thread.currentThread().getName()+"=>"+num);
        //notify other thread: work is done
        condition.signalAll();

        //解锁
        l.unlock();
    }
    //-1
    public void decrement() throws InterruptedException {
        l.lock();

        while(num==0){
            //wait
            condition.await();
        }
        num -- ;
        System.out.println(Thread.currentThread().getName()+"=>"+num);
        //notify other thread:work is done
        condition.signalAll();

        l.unlock();
    }
}

```

### condition的优势：精准唤醒

需求：使线程按顺序唤醒，比如A->B->C->D->A

代码实现精准唤醒：

```java
package com.kuang.pc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class C {
    public static void main(String[] args) {
        Data3 data = new Data3();
        new Thread(()->{
            for (int i=0;i<10;i++){
                data.printA();
            }
        },"A").start();
        new Thread(()->{
            for (int i=0;i<10;i++){
                data.printB();
            }
        },"B").start();
        new Thread(()->{
            for (int i=0;i<10;i++){
                data.printC();
            }
        },"C").start();
    }
}

/**
 * A执行完调用B，B执行完调用C，C执行完调用A
 */
class Data3{
    private Lock l = new ReentrantLock();
    private Condition condition1 = l.newCondition();
    private Condition condition2 = l.newCondition();
    private Condition condition3 = l.newCondition();
    private int num = 1; //1A 2B  3C
    public void printA(){
        l.lock();
        try {
            //业务 ： 判断》执行》通知
            while (num!=1){
                //等待
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName()+"--AAAAAA");
            num=2;
            //唤醒指定的线程
            condition2.signal();  //为什么condition2对应B呢
            //signalAll是唤醒全部，signal是精准唤醒

        }catch (Exception e){
                e.printStackTrace();
        }finally{
            l.unlock();
        }
    }
    public void printB(){
        l.lock();
        try {
            //业务 ： 判断》执行》通知
            while (num!=2){
                //等待
                condition2.await();
            }
            System.out.println(Thread.currentThread().getName()+"---BBBBB");
            num=3;
            //唤醒指定的线程
            condition3.signal();  //为什么condition3对应C呢

        }catch (Exception e){
            e.printStackTrace();
        }finally{
            l.unlock();
        }
    }
    public void printC(){
        l.lock();
        try {
            //业务 ： 判断》执行》通知
            while (num!=3){
                //等待
                condition3.await();
            }
            System.out.println(Thread.currentThread().getName()+"----CCCCC");
            num = 1;
            //唤醒指定的线程
            condition1.signal();  //为什么condition1对应A呢

        }catch (Exception e){
            e.printStackTrace();
        }finally{
            l.unlock();
        }
    }
    //生产线  下单-  支付 --交易
}

```

- 解疑：代码中的资源类中的三个condition，分别代表三个条件，或者代表三个标记。.await()方法将会将对应的condition“失效”，就可以使运行到await方法的线程就需要等该条件生效才能继续往下运行，等待其他线程中将该条件生效。而conditionx.signal()方法可以精准地使某一个条件“生效”，这样的等待该条件的线程可以继续运行
- 可以理解为，所有线程都是在执行的，只是有的线程将某一个条件condition对象失效了，该线程就暂时停止，直到另一个线程中将该条件生效，该线程才继续运行。

# 5 8锁现象

简化了一下，没有将全部8个问题整理下来，省略了一些简单的。

## 锁的是什么？如何理解锁？

synchronized加在static方法上，锁的是class，类模板

加在普通方法上，锁的是调用该方法的实例对象。

注意的是类模板和实例对象是相互不关联的，即，有一个线程锁了class，执行static方法，并不妨碍另一个线程获得该类的某个实例对象的锁，执行普通方法。

例子1：

```java
package com.kuang.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 锁的8个问题
 * 
 * 两个线程操作同一个对象，先拿到锁的线程先执行
 */
public class Test1 {
    public static void main(String[] args) throws InterruptedException {
        Phone phone = new Phone();
        new Thread(()->{
            try {
                phone.sendMsg();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            phone.call();
        }).start();
    }
}

class Phone{
    public synchronized void sendMsg() throws InterruptedException {
        Thread.sleep(4000);
        System.out.println("发短信");

    }

    public synchronized void call(){
        System.out.println("打电话");
    }
}

class Solution{
    //最长有效括号  dp问题
    public int logest(String s){
        int n =s.length();
        if(n<2){
            return 0;
        }
        int res=0;
        int [] dp= new int[n];
        for (int i=n-2;i>=0;i--){
            if (s.charAt(i)=='('){
                int j=i+dp[i+1]+1;
                if(j<n && s.charAt(j)==')'){
                    dp[i] = dp[i+1]+2;
                    if(j+1<n){
                        dp[i]+=dp[j+1];
                    }
                }
                res=Math.max(res,dp[i]);
            }
        }
        return res;
    }
}
```

例子2：

```java
package com.kuang.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 锁的8个问题
 *
 * synchronized锁的是对象，两个线程分别操作两个对象，就是互不干扰的，就不会存在等待锁释放的问题。
 */
public class Test2 {
    public static void main(String[] args) throws InterruptedException {
        Phone2 phone = new Phone2();
        Phone2 phone2 = new Phone2();
        new Thread(()->{
            try {
                phone.sendMsg();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            phone2.call();
        }).start();
    }
}

class Phone2{
    public synchronized void sendMsg() throws InterruptedException {
        Thread.sleep(4000);
        System.out.println("发短信");

    }

    public synchronized void call(){
        System.out.println("打电话");
    }

    public void hello(){  //不受锁的影响
        System.out.println("hello");
    }
}

```

例子3：

```java
package com.kuang.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 锁的8个问题
 * 如果某个类有两个static的同步方法，但是该类有两个实例化对象，分别通过这两个对象来调用方法，会不会被锁？
 * 事实说明，synchronized加在static方法上，锁的就是class模板，而不是具体对象，即使通过两个不同对象来调用方法，也是只有先拿到锁的先执行。
 */
public class Test3 {
    public static void main(String[] args) throws InterruptedException {
        Phone3 phone = new Phone3();
        Phone3 phone2 = new Phone3();

        new Thread(()->{
            try {
                phone.sendMsg();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            phone2.call();
        }).start();
    }
}

class Phone3{
    /**
     * 如果没有static 那么方法是对象，加了static，该方法就是类的，不实例化就可以调用。
     * 没有static的话，那么synchronized锁的是对象，锁住的对象是调用该方法的对象
     * 如果加了static，那么该方法就在类加载的时候就有了，此时synchronized锁的是class，而不是对象
     * @throws InterruptedException
     */
    public static synchronized void sendMsg() throws InterruptedException {
        Thread.sleep(4000);
        System.out.println("发短信");

    }

    public static synchronized void call(){
        System.out.println("打电话");
    }

    public void hello(){  //不受锁的影响
        System.out.println("hello");
    }
}

```

例子4：

```java
package com.kuang.lock8;

import java.util.concurrent.TimeUnit;

/**
 * 锁的8个问题
 * 如果某个类有一个static的同步方法，一个普通同步方法，两个线程操作同一个对象，会怎样？
 * 事实证明，synchronized锁住static方法时，锁的是class 类模板，而普通的synchronized方法锁的是对象，二者相互不影响！
 * */
public class Test4 {
    public static void main(String[] args) throws InterruptedException {
        Phone4 phone = new Phone4();
        Phone4 phone2 = new Phone4();

        new Thread(()->{
            try {
                phone.sendMsg();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        TimeUnit.SECONDS.sleep(1);

        new Thread(()->{
            phone.call();
        }).start();
    }
}

class Phone4{
    /**
     * 如果没有static 那么方法是对象，加了static，该方法就是类的，不实例化就可以调用。
     * 没有static的话，那么synchronized锁的是对象，锁住的对象是调用该方法的对象
     * 如果加了static，那么该方法就在类加载的时候就有了，此时synchronized锁的是class，而不是对象
     * @throws InterruptedException
     */
    public static synchronized void sendMsg() throws InterruptedException {
        Thread.sleep(4000);
        System.out.println("发短信");

    }

    public synchronized void call(){
        System.out.println("打电话");
    }

    public void hello(){  //不受锁的影响
        System.out.println("hello");
    }
}

```



# 6 不安全的集合类

## 1 ArrayList

```java
package com.kuang.unsafeCollections;


import java.util.*;
import java.util.concurrent.CopyOnWriteArrayList;

//java.util.ConcurrentModificationException 并发修改异常
public class ListTest {
    public static void main(String[] args) {

        //并发下，ArrayList不安全
        /**
         * 解决方案：
         * 1. List<String> list = new Vector<>();
         * 2. 数组的工具类叫Arrays， 集合的所有工具类叫Collections,利用Collections里的一些方法 List<String> list = Collections.synchronizedList(new ArrayList<>());
         * 3. List<String> list = new CopyOnWriteArrayList<>(); 使用JUC下的一个ArrayList类
         * CopyOnWrite：写入时复制，用到了COW：计算机设计领域的一种策略
         * 具体来说 就是，多线程调用时，在读取时是固定的，在写入时先复制一份，插入新元素，最后放回去，这样避免写入覆盖
         * Vector使用的是synchronized，写入效率低，而CopyOnWriteArrayList使用的是可重入锁，
         */

        List<String> list = new Vector<>();
//        List<String> list = Collections.synchronizedList(new ArrayList<>());
//        List<String> list = new CopyOnWriteArrayList<>();
        for(int i=0;i<10;i++){
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(list);
            },String.valueOf(i)).start();

        }
    }
}

```

### [CopyOnWrite容器的理解](https://ifeve.com/java-copy-on-write/)

- 是什么：

  CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的**好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁**，因为当前容器不会添加任何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。

- 应用场景：

  CopyOnWrite并发容器用于读多写少的并发场景。比如白名单，黑名单，商品类目的访问和更新场景，假如我们有一个搜索网站，用户在这个网站的搜索框中，输入关键字搜索内容，但是某些关键字不允许被搜索。这些不能被搜索的关键字会被放在一个黑名单当中，黑名单每天晚上更新一次。当用户搜索时，会检查当前关键字在不在黑名单当中，如果在，则提示不能搜索。

## 2 HashSet

```java
package com.kuang.unsafeCollections;

import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import java.util.UUID;
import java.util.concurrent.CopyOnWriteArraySet;

/**
 * 也会出现并发修改异常 java.util.ConcurrentModificationException
 * 解决：
 * 1. Set<String> set = Collections.synchronizedSet(new HashSet<>());
 * 2. Set<String> set = new CopyOnWriteArraySet<>();
 */
public class SetTest {
    public static void main(String[] args) {
//        Set<String> set = new HashSet<>();
        Set<String> set =new CopyOnWriteArraySet<>();
//                Set<String> set = Collections.synchronizedSet(new HashSet<>());
        for (int i =0;i<30;i++){
            new Thread(()->{
                set.add(UUID.randomUUID().toString().substring(0,5));
                System.out.println(set);
            },String.valueOf(i)).start();

        }
    }
}

```

### HashSet的底层

就是HashMap

```java
//构造函数
public HashSet() {
    map = new HashMap<>();
}
//add方法
public boolean add(E e) {
   return map.put(e, PRESENT)==null;
}
//PRESENT是个常量
private static final Object PRESENT = new Object();

```

## 3 HashMap

```java
package com.kuang.unsafeCollections;

import java.util.Collections;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 并发修改异常：java.util.ConcurrentModificationException
 * 1. Map<String,String> map = new ConcurrentHashMap<>();
 * 2. Map<String,String> map = Collections.synchronizedMap(new HashMap<>());
 */
public class MapTest {
    public static void main(String[] args) {
        //工作中不用HashMap
        //默认等价于new HashMap<>(16,0.75)
//        Map<String,String> map = new HashMap<>();
//        Map<String,String> map = Collections.synchronizedMap(new HashMap<>());
        Map<String,String> map = new ConcurrentHashMap<>();
        for (int i=0;i<30;i++){
            new Thread(()->{
                map.put(Thread.currentThread().getName(), UUID.randomUUID().toString().substring(0,5));
                System.out.println(map);
            },String.valueOf(i)).start();
        }
    }
}

```

# 7 Callable(简单)

类似于Runnable，但是Runnable不会返回值，也不会抛出异常

1. 可以返回值
2. 可以抛出异常
3. 方法不同，Runnable中的run方法对应Callable 的call方法

代码测试：

```java
package com.kuang.callable;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class CallableTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        new Thread(new MyThreadOld()).start();//Thread只能接收Runnable
        
        MyThread myThread = new MyThread();
        FutureTask futureTask = new FutureTask(myThread);
        new Thread(futureTask,"A").start();//FutueTask是Runnable的实现类
        new Thread(futureTask,"B").start();//注意，虽然又创建一个线程B执行，但是call只调用一次，原因是：结果会被缓存，效率高
        String res = (String) futureTask.get();//获取callable的返回值,这个get方法可能产生阻塞，因为call方法里可能是一个耗时操作，无法很快返回（主线程可能会在此处等待子线程执行完call方法），因此这一句最好写在最后一行；或者使用异步通信去处理
    }
}

class MyThreadOld implements Runnable{
    @Override
    public void run() {
        
    }
}
class MyThread implements Callable<String>{
    @Override
    public String call() throws Exception {
        System.out.println("call()");
        return "null";
    }
}

```

细节：

- 有缓存
- get方法获得结果可能需要等待，会阻塞！

# 8 常用辅助类（必会）

## CountDownLatch

```java
package com.kuang.helperClass;
//计数器
public class CountDownLatch {
    public static void main(String[] args) throws InterruptedException {
        //倒计时，总数是6
        java.util.concurrent.CountDownLatch countDownLatch = new java.util.concurrent.CountDownLatch(6);
        for (int i =0 ;i<6;i++){
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+" go out.");
                countDownLatch.countDown();//-1操作
            },String.valueOf(i)).start();
        }
        countDownLatch.await();//等待计数器归零，然后再继续向下执行，如果没有这句，就可能"lock door"出现在 "xxx go out"之前
        System.out.println("lock door");

    }
}

```

原理：

- countDownLatch.countDown();//-1操作
- countDownLatch.await();//在技术起点 值大于0之前，这个方法会使当前线程阻塞，等待计数器归零，然后再继续向下执行

每次有线程调用countDown()，数量就-1，假设计数器归零，countDownLatch.await()就会被唤醒，继续执行。

第二个例子：

```java
import java.util.concurrent.CountDownLatch;
public class cD implements Runnable{
  private   CountDownLatch begin ;
  private   CountDownLatch end;
  private  int s;
  public cD(CountDownLatch begin,CountDownLatch end){
	  this.begin=begin;
	  this.end=end;
	  this.s=10;
  }
  @Override  
  public synchronized void run(){  //线程同步用了synchronized否则无法保证s的正确性
	try{                       //注意：await用在synchronized可能导致死锁，如果换成CyclicBarrier类会导致死锁
		begin.await();    //等待begin统一
		System.out.println(s+" 次");
		s--;
		//do something
	}
	catch(Exception e){
		e.printStackTrace();
	}
	finally{
		end.countDown();
	} 
  }
  public static void main(String[] args) throws Exception {
	  CountDownLatch begin = new CountDownLatch(1);
	  CountDownLatch end = new CountDownLatch(10);
	  cD juc=new cD(begin,end);   
	  for(int j=0;j<10;j++){
		 new Thread(juc).start();   //开启10个线程，10个线程分别倒数1次，并让end减一次
	  }
	  //开始倒数计时，开启begin的countDown()方法
	  System.out.println("开始倒数计数!。。。10次");
	  begin.countDown();  //begin减到0，则开始10个线程里面的await()方法后面的程序
	  end.await(); 		  //阻塞程序，知道end减为0
	  System.out.println("倒数结束。。。后面的程序开始运行");
	  //do others
  }
}
```



## CyclicBarrier

该类从字面理解为循环屏障，它可以协同多个线程，让多个线程在这个屏障前等待，直到所有线程都到达了这个屏障时，每个线程再继续执行各自后面的操作。假如每个线程各有一个await，任何一个线程运行到await方法时就阻塞，直到最后一个线程运行到await时才同时返回。和之前的CountDownLatch相比，它只有await方法，而CountDownLatch是使用countDown()方法将计数器减到0，它创建的参数就是countDown的数量；CyclicBarrier创建时的int参数是await的数量。

简单理解为：加法计数器

```java
package com.kuang.helperClass;

import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {
    public static void main(String[] args) {
        /**
         * 集齐七颗龙珠召唤神龙
         */
        //召唤龙珠的线程
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("召唤神龙成功！");
        });
        for (int i = 1;i<=7;i++){
            final int temp = i;
            //lambda能操作到i吗？不能，因为lambda本质是new了一个class的实例，而这个class是一个实现了函数式接口的类
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"收集到第"+temp+"颗龙珠");
                try {
                    cyclicBarrier.await(); //等待。
                    // 只有此处与cyclicBarrier有关，所以，也就是需要await()执行七次，才能继续执行？
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
}

```

第二个例子：

```java
import java.util.concurrent.CyclicBarrier;
public class cD implements Runnable{
  private   CyclicBarrier end;
  private  int s;
  public cD(CyclicBarrier end,int i){
	  this.end=end;
	  s=i;
  }
  @Override  
  public  void run(){  //线程同步用了synchronized否则无法保证s的正确性
	try{
		System.out.println(s+"队准备完毕");
		s--;
		end.await(); 
	}
	catch(Exception e){
		e.printStackTrace();
	}
  }
  public static void main(String[] args) throws Exception {
	  CyclicBarrier end = new CyclicBarrier(11);  //await为10+1=11个
	  for(int j=0;j<10;j++){
		 new Thread(new cD(end,j)).start();   //开启10个线程
	  }
	  end.await(); 		  //阻塞程序，直到10个子线程全部运行到await处
	  System.out.println("\n10队全部准备结束。。。后面的程序开始运行");
	  //do others
  }
}
```



## Semaphore

抢车位：六个车，三个位置

```java
package com.kuang.helperClass;

import java.util.concurrent.TimeUnit;

public class Semaphore {
    public static void main(String[] args) {
        //线程数量，停车位
        java.util.concurrent.Semaphore semaphore = new java.util.concurrent.Semaphore(3);
        for (int i=0;i<6;i++){
            new Thread(()->{
                //acquire()得到  release()释放
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName()+"抢到车位");
                    TimeUnit.SECONDS.sleep(2);
                    System.out.println(Thread.currentThread().getName()+"离开车位");

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            },String.valueOf(i)).start();
        }
    }
}

```

原理：

- acquire()  :获得，如果已经满了，就等待释放为止
- release()：释放，将当前的信号量释放（+1），然后唤醒等待的线程

作用：

- **多个共享资源互斥的使用** 
- 并发限流

# 9 读写锁

java.util.concurrent下有三个接口，Condition，Lock，ReadWriteLock。

ReadWriteLock只有一个实现类就是ReentrantReadWriteLock

ReadWriteLock维护一对锁，读锁和写锁，允许多个线程读，只允许一个线程写

```java
package com.kuang.rw;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
独占锁：就类似于写锁，一次只能被一个线程持有
共享锁：就类似于读锁，可以同时被多个线程持有
读写锁：
读-读 可以共存
读-写：不可共存
写-写：不可共存
*/
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCacheLock myCache = new MyCacheLock();
        //写入
        for (int i=0;i<5;i++){
            final int temp = i;
            new Thread(()->
            {
                myCache.put(temp+"",temp+"");
            },String.valueOf(i)).start();
        }
        //读取
        for (int i=0;i<5;i++){
            final int temp = i;
            new Thread(()->
            {
                myCache.get(temp+"");
            },String.valueOf(i)).start();
        }


    }
}
//自定义缓存
class MyCache{
    private volatile Map<String,Object> map = new HashMap<>();
    //存
    public void put(String s,Object obj){
        System.out.println(Thread.currentThread().getName()+"写入"+s);
        map.put(s, obj);
        System.out.println(Thread.currentThread().getName()+"写入完毕");

    }
    //取
    public void get(String s){
        System.out.println(Thread.currentThread().getName()+"读取"+s);
       Object object = map.get(s);
       System.out.println(Thread.currentThread().getName()+"读取ok");

    }
}
//加锁的自定义缓存
class MyCacheLock{
    private volatile Map<String,Object> map = new HashMap<>();
    //读写锁，更加细粒度的控制：写入时只允许一个线程写，读取时允许多个线程读
    private ReadWriteLock lock =  new ReentrantReadWriteLock();
//    private Lock lock = new ReentrantLock();//普通锁 
    //存
    public void put(String s,Object obj){
        lock.writeLock().lock();
        try{
            System.out.println(Thread.currentThread().getName()+"写入"+s);
            map.put(s, obj);
            System.out.println(Thread.currentThread().getName()+"写入完毕");    
        } finally {
            lock.writeLock().unlock();    
        }
    }
    //取
    public void get(String s){
        lock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName()+"读取"+s);
            Object object = map.get(s);
            System.out.println(Thread.currentThread().getName()+"读取ok");    
        }finally {
            lock.readLock().unlock();    
        }
    }
}

```

## try catch-finally-面试题

[参考：](https://blog.csdn.net/weixin_42924812/article/details/105210908)

- catch和finally可以省略其一，但不能同时省略
- 无论何种情况，finally一定会执行，即使try-catch中return了
- try或catch中有return的话，在return之前finally仍会执行（如果finally还有return，那么会覆盖其他return）
- 对于基本数据类型的数据，在finally块中改变return的值对返回值没有影响，而对引用数据类型的数据会有影响。

注意：finally也不是一定会被执行，如果在try代码块中，System.exit()强制退出程序,或者在执行try代码块中报错抛出异常（例如5/0），finally代码块就不会执行了。

## 锁和try-catch面试题

- 为什么加锁在try外
- 为什么解锁在finally中
- 为什么使用try包裹业务代码

[答案：](https://blog.csdn.net/u013568373/article/details/98480603)

- try包裹业务代码，解锁在finally中，是为了在发生异常时仍然能执行finally代码，保证锁的释放
- 加锁在try外，那么加锁失败就不会执行finally里的解锁，如果放在try中，一旦加锁失败，执行finally里的解锁，就会因为没有得到锁而解锁出现异常。

# 10 阻塞队列 Blocking Queue

与List，Set是同级的，是JUC下的一个接口，实现类有：ArrayBlockingQueue，LinkedBlockingQueue, SynchronizedBlockingQueue，PriorityBlockingQueue等

什么时候使用：多线程并发处理；线程池

**使用：**添加和移出

## **四组API**

1. 抛出异常
2. 不抛出异常
3. 阻塞等待
4. 超时等待

| 方式             | 抛出异常        | 有返回值（t或f）不抛出异常 | 阻塞等待      | 超时等待                       |
| ---------------- | --------------- | -------------------------- | ------------- | ------------------------------ |
| 添加             | add(Object obj) | offer(Object o)            | put(Object o) | offer(Object o,时间，时间单位) |
| 移除             | remove()        | poll()                     | take()        | pull(时间，时间单位)           |
| 返回队列首部元素 | element()       | peek()                     |               |                                |

代码演示：

```java
package com.kuang.bq;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.TimeUnit;

public class Test {
    public static void main(String[] args) {
        //Collection接口  java.util
        //List实现Collection接口  java.util
        //Set实现Collection接口  java.util
        //Queue实现Collection，BlockingQueue是实现了Queue的接口
        test1();

    }

    /**
     * 抛出异常
     */
    public  static void test1(){
        ArrayBlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.add("A"));
        System.out.println(blockingQueue.add("B"));
        System.out.println(blockingQueue.add("C"));
        //异常，超出容量
        System.out.println(blockingQueue.add("A"));

        System.out.println(blockingQueue.element());//查看队首元素

        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        System.out.println(blockingQueue.remove());
        //异常， 已经没有元素了
        System.out.println(blockingQueue.remove());
    }

    /**
     * 有返回值，不抛出异常
     */
    public static void test2(){
        ArrayBlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        System.out.println(blockingQueue.offer("A"));
        System.out.println(blockingQueue.offer("B"));
        System.out.println(blockingQueue.offer("C"));
        //队列已满，不会抛出异常，会返回一个false
        System.out.println(blockingQueue.offer("C"));

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        //队列已空，不会抛出异常，返回null
        System.out.println(blockingQueue.poll());
    }

    /**
     * 等待 阻塞，一直等待
     */

    public void test3() throws InterruptedException {
        ArrayBlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        blockingQueue.put("A"); //没有返回值
        blockingQueue.put("B");
        blockingQueue.put("C");
        //队列已满，会一直等待
        blockingQueue.put("D");

        System.out.println(blockingQueue.take());//取出，返回元素
        System.out.println(blockingQueue.take());
        System.out.println(blockingQueue.take());
        //队列已空，会一直阻塞，处于等待状态
        System.out.println(blockingQueue.take());
    }

    /**
     * 等待 阻塞，等待超时
     */
    public void test4() throws InterruptedException {
        ArrayBlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(3);
        blockingQueue.offer("A");
        blockingQueue.offer("A");
        blockingQueue.offer("A");
        ////队列已满，就会等待2秒，如果2秒后还没有位置，就退出
        blockingQueue.offer("A",2, TimeUnit.SECONDS);

        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        System.out.println(blockingQueue.poll());
        //超过2秒没取出就不取了
        System.out.println(blockingQueue.poll(2, TimeUnit.SECONDS));
    }
}

```



## SynchronizedBlockingQueue

没有容量，进去一个元素，只有取出来才能再放，相当于容量为1.

put()  take()

```java
package com.kuang.bq;

import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.TimeUnit;

public class SynchronizedBlockingQueue {
    public static void main(String[] args) {
        //和其他BlockingQueue不一样，不存储元素，
        //put了一个元素，就必须先取出后再put
        SynchronousQueue<String> synchronizedBlockingQueue = new SynchronousQueue<String>();
        new Thread(()->{

            try {
                System.out.println(Thread.currentThread().getName()+"put 1");
                synchronizedBlockingQueue.put("1");
                System.out.println(Thread.currentThread().getName()+"put 2");
                synchronizedBlockingQueue.put("2");
                System.out.println(Thread.currentThread().getName()+"put 3");
                synchronizedBlockingQueue.put("3");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T1").start();
        new Thread(()->{
            try {
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+synchronizedBlockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+synchronizedBlockingQueue.take());
                TimeUnit.SECONDS.sleep(3);
                System.out.println(Thread.currentThread().getName()+synchronizedBlockingQueue.take());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"T2").start();
    }
}

```

# 11 线程池（重点）

**面试掌握：三大方法，7大参数，4大拒绝策略**

> 池化技术

程序运行本质就是占用系统资源，所以要优化资源使用，就有了池化技术

线程池，连接池，内存池，对象池。。。

池化技术：事先准备好一些资源，有人要用就来这些拿，用完还回

## 线程池的好处：

- 降低资源消耗（创建和销毁十分浪费资源）
- 提高响应速度（节省了创建时间）
- 方便管理

线程复用，可以控制最大并发数，管理线程

> 线程池三大方法

```java
package com.kuang.pool;

import oracle.jrockit.jfr.jdkevents.ThrowableTracer;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
//Executors就是个工具类，有三大方法 都是用来new一个线程池的
//
public class Demo01 {
    public static void main(String[] args) {
//        ExecutorService threadPool = Executors.newSingleThreadExecutor();//创建一个单个线程的线程池
//        ExecutorService threadPool = Executors.newFixedThreadPool(5);//创建一个固定大小的线程池
        ExecutorService threadPool = Executors.newCachedThreadPool();//可伸缩的

        try{
            for (int i=0;i<10;i++){
                threadPool.execute(()->{ //执行
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        }finally {
            //需要关闭线程池
            threadPool.shutdown();
        }


    }
}

```

> 七大参数

以上三个方法都是new了一个ThreadPoolExecutor并返回

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```

那么什么是ThreadPoolExecutor

```java
//此为ThreadPoolExecutor的构造方法，可以看到有七个参数
public ThreadPoolExecutor(int corePoolSize, //核心线程池大小
                              int maximumPoolSize, //最大线程池的大小
                              long keepAliveTime, //线程超过一定时间无人调用就释放
                              TimeUnit unit,//时间单位
                              BlockingQueue<Runnable> workQueue,//阻塞队列
                              ThreadFactory threadFactory, //创建线程的线程工厂
                              RejectedExecutionHandler handler) { //拒绝策略
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

阿里建议使用ThreadPoolExecutor创建线程池，而不用Executor，更有利于理解底层原理和运行规则，避免资源耗尽的风险

Executor的弊端：

- FixedThreadPool和SingleThreadPool

  允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量请求导致OOM

- CachedThreadPool和ScheduledThreadPool

  允许的创建的线程数为Integer.MAX_VALUE，可能会堆积大量请求导致OOM

七大参数：

核心线程数：一直运行的线程数量

最大线程数：当阻塞队列满时，能够开启的最大线程数量

如果已经开启了最大线程数量，且阻塞队列满了，这时候如何处理新来的请求，就是拒绝策略

除了几个核心线程之外，其他线程如果一段时间没有业务就释放

最大承载请求：队列长+最大线程数，一旦超出，新请求就执行拒绝策略

> ## 手动创建线程池

```java
package com.kuang.pool;

import oracle.jrockit.jfr.jdkevents.ThrowableTracer;

import java.util.concurrent.*;

//Executors就是个工具类，有三大方法 都是用来new一个线程池的
//
public class Demo01 {
    public static void main(String[] args) {

        System.out.println(Runtime.getRuntime().availableProcessors());//获取cpu核数
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                3,
                TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(3),
                Executors.defaultThreadFactory(),//这个工厂一般不动
                new ThreadPoolExecutor.AbortPolicy()); //这是默认拒绝策略：不处理，抛出异常

        try{
            for (int i=0;i<8;i++){
                threadPool.execute(()->{ //执行
                    System.out.println(Thread.currentThread().getName()+"ok");
                });
            }
        }finally {
            //需要关闭线程池
            threadPool.shutdown();
        }


    }
}
```

四个拒绝策略：

- ```java
  new ThreadPoolExecutor.AbortPolicy();//丢弃任务，抛出异常
  new ThreadPoolExecutor.CallerRunsPolicy();//哪来的回哪去，将任务交给调用线程池的线程处理，如果该线程已终止，就丢弃任务。
  new ThreadPoolExecutor.DiscardPolicy();//不抛出异常，直接丢掉任务
  new ThreadPoolExecutor.DiscardOldestPolicy();//放弃队列中最早的未处理请求，然后重试execute，如果执行程序已关闭，则丢弃任务，不抛出异常。
  ```

## CPU密集型和IO密集型

最大线程数到底该如何定义：

CPU密集型：几核就定义为几，保证CPU效率最高，运行时用代码获取cpu核数

```java
System.out.println(Runtime.getRuntime().availableProcessors());//获取cpu核数
```

IO密集型：判断你的程序中十分耗IO的线程有多少，只要大于这个数就可以，一般设2倍。

# 12 四大函数式接口（必会）

泛型 枚举 反射 注解是旧时代的

新时代必会：lamdba表达式，链式编程，函数式接口，stream流式计算（jdk8刚出现的 ）

- 函数式接口：只有一个方法的接口，比如Runnable接口

  ```java
  @FunctionalInterface
  public interface Runnable {
      /**
       * When an object implementing interface <code>Runnable</code> is used
       * to create a thread, starting the thread causes the object's
       * <code>run</code> method to be called in that separately executing
       * thread.
       * <p>
       * The general contract of the method <code>run</code> is that it may
       * take any action whatsoever.
       *
       * @see     java.lang.Thread#run()
       */
      public abstract void run();
  }
  //
  ```

  只要是函数式接口就可以用lambda简化

- lambda的格式就是()->{} 小括号写参数，大括号写代码，用法就是：比如new Thread() 需要传入一个实现了Runnable接口的类的实例，那么new Thread(()->{System.out.println("");}).start();即可

```java
都来自package java.util.function
//1. 函数型接口
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
}
//2. 断定型接口，有一个输入参数，返回值是布尔值
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
}
//3. 消费型接口 只有输入，无返回值
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
}
//4. 供给型接口
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
//测试一下：
Supplier<Integer> supplier = ()->{return 1;};
System.out.println(supplier.get());
```

# 13 Stream流式计算

是什么

集合、MySQL本质就是存东西的

计算交给流来做

代码实现：

```java

package com.kuang.stream;

import java.util.Arrays;
import java.util.List;

/**
 * 筛选出满足以下条件的用户：
 * 1. id为偶数
 * 2. 年龄大于23
 * 3. 名称转为大写字母
 * 4. 用户名字母倒着排序
 * 5. 只输出一个用户！
 */
public class Test {
    public static void main(String[] args) {
        User u1 = new User(1,"a",21);
        User u2 = new User(0,"b",23);
        User u3 = new User(4,"c",24);
        User u4 = new User(6,"d",22);
        User u5 = new User(8,"e",26);
        //集合就是存储
        List<User> users = Arrays.asList(u1,u2,u3,u4,u5);
        //计算交给流
        users.stream()  //stream下的各个方法（filter，map，sorted）里的参数都是函数式接口
                .filter((user)->{ return user.getId()%2==0; })
                .filter(u->{return u.getAge()>23;})
                .map(u->{return u.getName().toUpperCase();}) //转大写
                .sorted((uu1,uu2)->{return uu2.compareTo(uu1);})//倒序
                .limit(1)//只显示一个
                .forEach(System.out::println);
    }
}

```

- 该例子也体现了链式编程，也叫函数式编程。

# 14 ForkJoin

什么是ForkJoin

出现在jdk7，并行执行任务，提高效率！

本质上说，就是把一个大任务分成若干小任务并行处理，每个小任务返回一个结果，最后Join成最终结果

ForkJoin特点：**工作窃取**（线程A，B同时执行各自的若干个任务，B先处理完，B就拿过A的任务来处理。对每个线程来说，各自的子任务都存在一个双边队列里，自己可以从一端拿任务执行，其他线程可以从另一端获取任务执行）

```java
package com.kuang.forkjoin;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.stream.LongStream;

public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        test1();
        test2();
        test3();
    }
    public static void test1(){
        long start = System.currentTimeMillis();
        long sum = 0L;
        for (long i =1L;i<=10_0000_0000L;i++){
            sum+=1;
        }

        long end = System.currentTimeMillis();
        System.out.println("sum=时间："+(end-start));
    }
    //forkjoin
    public static void test2() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Long> task = new ForkJoinDemo(0L,10_0000_0000L);
//        forkJoinPool.execute(task);//执行任务，没有结果
        ForkJoinTask<Long> submit = forkJoinPool.submit(task);//提交任务
        long sum = submit.get();//获得结果，此处会阻塞等待
        long end = System.currentTimeMillis();
        System.out.println("sum=时间："+(end-start));  //这个结果怎么是最慢的？？？？
    }
    //stream并行流
    public static void test3(){
        long start = System.currentTimeMillis();
        long sum = LongStream.rangeClosed(0L,10_0000_0000L).parallel().reduce(0, Long::sum);
        long end = System.currentTimeMillis();
        System.out.println("sum=时间："+(end-start));//这个结果怎么也没快到哪去？？？？
    }
}

```

```java
package com.kuang.forkjoin;

import java.util.concurrent.RecursiveTask;

/**
 * 普通->ForkJoin-> Stream并行流
 */
public class ForkJoinDemo extends RecursiveTask<Long> {
    private long start;
    private long end;
    //临界值
    private long temp=10000L;
    public ForkJoinDemo(Long start,Long end){
        this.start = start;
        this.end=end;

    }
            //如何使用ForkJoin
            //1. forkjoinPool
            //2. 计算任务 forkjoinPool.execute(ForkJoinTask task)
            // ForkJoinTask有两个子类，一个是适用有返回值任务的RecursiveTask<返回值类型>，一个是适用无返回值任务的RecursiveAction
            //只需写一个类继承这两个类之一并重写其方法即可

    @Override
    protected Long compute() {
        if ((end-start)<temp){
            long sum = 0L;
            for (Long i = start; i <= end; i++){
                sum+=i;
            }
//            System.out.println(sum);
            return sum;
        }else{
            //分支合并计算  很像递归
            long middle = (start+end)/2; //中间值
            ForkJoinDemo task1 = new ForkJoinDemo(start,middle);
            task1.fork();//把线程压入队列
            ForkJoinDemo task2 = new ForkJoinDemo(middle+1,end);
            task2.fork();
            return task1.join()+task2.join();

        }

    }
    public static void main(String[] args) {

    }
}
```



- 上述代码结果有一些问题,forkjoin用时最长，stream流也没有很快。为什么？？

# 15 异步回调

Future：设计的初衷：对将来某个事件 的结果进行建模

Future是一个接口，CompletableFuture是它的一个实现类

```java
package com.kuang.future;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;

/**
 * 异步调用：相当于ajax
 * 异步执行
 * 成功回调
 * 失败回调
 */
public class Demo1 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //发起请求
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(()->{ //runAsync是没有返回值的异步回调
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"runAync=>void");
        });
        System.out.println("1111");
        completableFuture.get();//可能会阻塞等待 来获取执行结果

        System.out.println("===================================");
        
        //有返回值的异步回调:supplyAsync
        CompletableFuture<Integer> completableFuture1 = CompletableFuture.supplyAsync(()->{
            System.out.println(Thread.currentThread().getName()+"supplyAsync=》Integer");
            int i =10/0;//故意制造异常
            return 1024;
        });
        System.out.println(
                completableFuture1.whenComplete((t, u) -> {
            System.out.println("t=" + t); //t是正常运行时返回的1024，若有异常则为null
            System.out.println("u=" + u);// u在无异常情况下为null，有异常则为异常的类型
        }).exceptionally((e) -> {
            System.out.println(e.getMessage());//有异常则会执行这一段，无异常则不执行
            return 233;
        }).get());

    }
}
```

# 16 JMM

volatile：java虚拟机提供的轻量级同步机制，没有Synchronized那么强大。它有三个重要点（特征）：

1. 保证可见性
2. **不保证原子性**
3. 禁止指令重排

## **什么是JMM**

 参考：https://blog.csdn.net/u011080472/article/details/51337422

JMM：java内存模型，不存在的东西，一种概念，一种约定。

**关于JMM的一些同步的约定：**

1. 线程解锁前：必须把共享变量**立刻**刷回主内存
2. 线程加锁前：必须读取主内存中的最新值到工作内存中
3. 加锁跟解锁是同一把锁

### 什么是JMM

JMM屏蔽掉不同硬件和操作系统对内存访问的差异，实现让java程序在各种平台下都能达到一致的并发效果。

JMM规定所有变量都存储在主内存中，（包括实例变量，静态变量）但不包括局部变量和方法参数。线程自己的工作内存保存了该线程用到的变量和主内存副本拷贝，线程对变量的操作都是在工作内存中进行，不能直接读写主内存的变量。

不同线程也无法访问对方工作内存中的变量。线程之间的变量值的传递均需要通过主内存来完成。

### JMM定义了什么

JMM实际上是围绕三个特征建立起来的。分别是原子性、可见性、有序性。这三个特征可谓是整个java开发的基础。



## **8种操作：**

每个线程有自己的执行引擎，自己的工作内存。线程在操作共享资源时，有8个操作，分为4组：

- 加锁，解锁
- 从主内存read到缓冲区，再load进工作内存（相当于从主内存到工作内存复制一份）
- 执行引擎use工作内存中的共享变量，操作后再assign回工作内存
- 工作内存将变量store到缓冲区，再write到主内存



关于主内存与工作内存之间的具体交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步到主内存之间的实现细节，Java内存模型定义了以下八种操作来完成：

1. lock（锁定）：作用于主内存的变量，把一个变量标识为一条线程独占状态。
2. unlock（解锁）：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
3. read（读取）：作用于主内存变量，把一个变量值从主内存传输到线程的工作内存中，以便随后的load动作使用
4. load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
5. use（使用）：作用于工作内存的变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
6. assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收到的值赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
7. store（存储）：作用于工作内存的变量，把工作内存中的一个变量的值传送到主内存中，以便随后的write的操作。
8. write（写入）：作用于主内存的变量，它把store操作从工作内存中一个变量的值传送到主内存的变量中。

如果要把一个变量从主内存中复制到工作内存，就需要按顺寻地执行read和load操作，如果把变量从工作内存中同步回主内存中，就要按顺序地执行store和write操作。Java内存模型只要求上述两个操作必须按顺序执行，而没有保证必须是连续执行。也就是read和load之间，store和write之间是可以插入其他指令的，如对主内存中的变量a、b进行访问时，可能的顺序是read a，read b，load b， load a。Java内存模型还规定了在执行上述八种基本操作时，必须满足如下规则：

- 不允许read和load、store和write操作之一单独出现
- 不允许一个线程丢弃它的最近assign的操作，即变量在工作内存中改变了之后必须同步到主内存中。

- 不允许一个线程无原因地（没有发生过任何assign操作）把数据从工作内存同步回主内存中。

- 一个新的变量只能在主内存中诞生，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量。即就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。

- 一个变量在同一时刻只允许一条线程对其进行lock操作，lock和unlock必须成对出现
  如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前需要重新执行load或assign操作初始化变量的值

- 如果一个变量事先没有被lock操作锁定，则不允许对它执行unlock操作；也不允许去unlock一个被其他线程锁定的变量。

- 对一个变量执行unlock操作之前，必须先把此变量同步到主内存中（执行store和write操作）。



# 17 volatile

## 保证可见性

问题：

```java
package com.kuang.volatileTest;

import java.util.concurrent.TimeUnit;

public class JMMDemo {
    private  static  int num = 0;
    public static void main(String[] args) {  //主线程
        
        new Thread(()->{  //线程1
            while(num==0){
                //
            }
        }).start();
        
        try{
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        num=1;
        System.out.println(num);  //运行发现，主线程将num改为1，但是线程A并没有停止，
        // 因为线程A将num复制到自己的工作内存中，其他线程对num进行修改，主内存发生变化，是线程A看不到的，也就是不可见。
        // 所以num变化之后，需要去通知线程A才可以让线程A知道发生了变化
        //那能不能让线程A看到共享内存中值的变化呢？？？？
    }
}

```

- 解决：
  1. 是否可以加Synchronized来同步代码块（可行吗？有待验证）
  2. volatile来保证可见性

```java
private volatile static  int num = 0; //修改此行代码即可
```

## 不保证原子性

线程A在执行任务时，不能被打扰，也不能被分割，要么同时成功，要么同时失败

```java
package com.kuang.volatileTest;

public class VDemo2 {
    private static int num=0;
    private static void add(){
        num++;
    }
    public static void main(String[] args) {
        for (int i=0;i<20;i++){
            new Thread(()->{
                for (int j=0;j<1000;j++){
                    add();
                }
            }).start();
        }//理论上num结果为2w,但由于多线程都把num读取到了自己的工作内存再执行加法操作，所以会导致结果小于2w
        //解决方法是在add方法上加Synchronized，保证只有一个线程执行add方法
        // （volatile只能加在变量上，不能加在方法上）volatile不保证原子性，所以在num上加volatile不能解决这个问题
        while (Thread.activeCount()>2){//如果还有两个线程就说明还没执行完
            Thread.yield();//那么就让主线程礼让
        }
        System.out.println(Thread.currentThread().getName()+" "+num);
    }
}

```

**如果不加Lock和synchronized怎么样保证原子性？？**

为什么上例子中只有num++一行代码还不能保证原子性？（分析字节码文件可知，这行代码具体还要分三步：获取static值，执行操作，写回static值）

如下修改即可：使用AtomicXxx包装类做共享变量。这个方法十分高效，因为都是用的底层native方法，用到了CAS原理

```java
package com.kuang.volatileTest;

import java.util.concurrent.atomic.AtomicInteger;

public class VDemo2 {
    private volatile static AtomicInteger num=new AtomicInteger(0);
    private static void add(){
        num.getAndIncrement();//AtomicInteger的加1方法；使用了底层（都是native方法）的CAS，效率极高
    }
    public static void main(String[] args) {
        for (int i=0;i<20;i++){
            new Thread(()->{
                for (int j=0;j<1000;j++){
                    add();
                }
            }).start();
        }//理论上num结果为2w,但由于多线程都把num读取到了自己的工作内存再执行加法操作，所以会导致结果小于2w
        //解决方法是在add方法上加Synchronized，保证只有一个线程执行add方法
        // （volatile只能加在变量上，不能加在方法上）volatile不保证原子性，所以加volatile不能解决这个问题
        while (Thread.activeCount()>2){//如果还有两个线程就说明还没执行完
            Thread.yield();//那么就让主线程礼让
        }
        System.out.println(Thread.currentThread().getName()+" "+num);
    }
}

```



- 这些类（AtomicXxx）的底层都直接和操作系统挂钩，在内存中修改值，效率极高。（涉及到Unfase类，一个很特殊的存在）

## 可避免指令重排

是什么？

- 你写的程序，计算机并不是按照你写的那样执行的。
- 源代码->编译器优化->指令并行也可能重排->内存系统也会重排->执行

**处理器在执行指令重排时，会考虑数据的依赖性**

```java
int x = 1;//1
int x = 2;//2
x = x + 5;//3
y = x + x;//4
//以上代码，并不是按顺序执行，可能顺序是2134，1324，但考虑数据依赖性，不会是4123
```

假设x y a b都是0

| 线程A执行代码 | 线程B执行代码 |
| ------------- | ------------- |
| x=a           | y=b           |
| b=1           | a=2           |

正常的结果是：x = 0;y = 0;

但由于指令重排，可能线程A先执行b=1;因为这两个语句的顺序对线程A无关紧要；线程B也可能先执行a=2；

这样导致的结果就是：x=2;y=1;

这个结果在理论上 逻辑上是会产生的。

**volatile可以避免指令重排。**

小结：volatile可以保证可见性，不保证原子性，由于内存屏障，可以避免指令重排现象的产生

# 18 深入单例模式

内存屏障在单例模式里使用最多，

饿汉式的单例模式中有个DCL懒汉式就用到了内存屏障

## 饿汉式单例：

```java
package com.kuang.singleton;
//饿汉单例
public class Hungry {
    //这个对象有这四组数据，饿汉式一上来就会初始化一个对象，把这些数据加载进来，而这个对象还没有被使用，因此会浪费空间
    private byte[] data1= new byte[1024*1024];
    private byte[] data2= new byte[1024*1024];
    private byte[] data3= new byte[1024*1024];
    private byte[] data4= new byte[1024*1024];
    private Hungry(){

    }
    private final static Hungry HUNGRY = new Hungry();
    public static Hungry getInstance(){
        return HUNGRY;
    }
}

```

## 懒汉式单例

```java
package com.kuang.singleton;

//懒汉式单例
public class Lazy {
    private Lazy(){
        //查看哪个线程执行了构造方法。执行了构造方法就说明new了一个实例对象
        System.out.println(Thread.currentThread().getName()+"ok");
    }

    private volatile static Lazy lazy;
    //双重检测锁模式的懒汉式称为DCL懒汉式
    public static Lazy getInstance(){
        if (lazy == null){
            synchronized (Lazy.class){//1. 锁住类模板  因为这个getInstance方法和lazy都是static的
                if(lazy==null) { //2. 双重检测 为什么？因为可能2个线程发现lazy==null，然后其中一个线程拿到锁new出了一个实例
                                //然后第二个线程接着获得锁，此时就需要再一次判断来验证第一个线程已经创建了实例
                    lazy = new Lazy();
                }
                //3. 以上还可能出现问题，因为lazy = new Lazy();不是原子性操作，具体分三步完成：1）分配空间 2）在空间中初始化对象 3）将lazy指向该内存空间
                //由于指令重排的存在，可能线程A拿到了锁并进行new操作，先分配了空间，然后就指向了这个空间，还没有初始化，
                // 此时线程B执行getInstance时执行第一处判断就会发现lazy不为空，就会直接返回，那么返回的就是一个不正确的结果。所以还需要在lazy上加volatile避免指令重排。
            }

        }
        return lazy;
    }

    //多线程并发的情况下，可能会创建多个对象，也就是说，有若干个线程都见到lazy==null 然后都创建实例，就导致不再是单例模式了,解决方法就是加锁
    //
    public static void main(String[] args) {
        for (int i = 0;i<10;i++){
            new Thread(()->{
                Lazy.getInstance();
            }).start();
        }
    }
}

```



## 内部类单例

```java
package com.kuang.singleton;

//静态内部类实现
public class Holder {
    private Holder(){//只要是单例模式就是构造器私有,构造器私有了，其他类中就无法直接new这个类的实例

    }
    public static Holder getInstance(){
        return InnerClass.HOLDER;
    }
    private static class InnerClass{
        private static final Holder HOLDER = new Holder();
    }
}
//所有单例模式创建方式都有一个漏洞，就是可以通过反射来将构造器取消私有化，然后就可以直接new了。
```

以上来自狂神Java的JUC并发编程视频下的彻底玩转单例模式，一些东西（如枚举类分析）太深入就没有整理。

# 19 深入理解CAS

同样来自狂神的JUC编程视频，看不懂。

## CAS是什么

```java
package com.kuang.cas;

import java.util.concurrent.atomic.AtomicInteger;

public class CASDemo {

    //CAS
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(2020);
        /**
         * public final boolean compareAndSet(int expect, int update) {
         *         return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
         *     }
         *     如果期望的值达到了,就更新，否则不更新
         */
        boolean ifsuccess = atomicInteger.compareAndSet(2020,2021);//比较并交换

        boolean ifsuccess2 = atomicInteger.compareAndSet(2020,2023);//比较并交换

        System.out.println(atomicInteger.get());

    }
}

```

```java
public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }
```

```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

CAS：比较当前工作内存中的值和主内存中的值，如果这个值是期望的，那么执行操作，不是则一直循环

缺点：

- 循环会耗时
- 一次只能保证一个共享变量的原子性
- ABA问题



# 20 原子引用解决ABA问题

**什么是ABA问题？**

两个线程A、B操作同一个共享资源C，A将C的值由期望1改为2（CAS(1,2)），B将C的值由期望1改为3，又将该值由期望3改为了1（CAS(1,3)，CAS(3,1)），此时，变量已经被动过了，但是对于A来说并没有察觉到。

**如何解决**？答：**带版本号的原子操作**！使用原子引用AtomicStampedReference，思想是乐观锁。



```java
package com.kuang.cas;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicStampedReference;

public class CASDemo {

    //CAS
    public static void main(String[] args) {
//        AtomicInteger atomicInteger = new AtomicInteger(2020);
//        /**
//         * public final boolean compareAndSet(int expect, int update) {
//         *         return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
//         *     }
//         *     如果期望的值达到了,就更新，否则不更新
//         */
//        boolean ifsuccess = atomicInteger.compareAndSet(2020,2021);//比较并交换
//
//        boolean ifsuccess2 = atomicInteger.compareAndSet(2020,2023);//比较并交换
//
//        System.out.println(atomicInteger.get());

        //注意如果泛型是包装类，注意对象的引用问题！！！
        AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(1,1);
        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();//获取版本号
            System.out.println("a1="+stamp);

            //加延时是为了程序开始时让两个线程拿到相同的版本号
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            //第一次修改
            atomicStampedReference.compareAndSet(1,2,
                    atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);//多两个参数，分别是旧版本和新版本号
            System.out.println("a2="+atomicStampedReference.getStamp());

            //第二次修改，改回去
            atomicStampedReference.compareAndSet(2,1,
                    atomicStampedReference.getStamp(),atomicStampedReference.getStamp()+1);//多两个参数，分别是旧版本和新版本号
            System.out.println("a3="+atomicStampedReference.getStamp());

        },"a").start();

        new Thread(()->{
            int stamp = atomicStampedReference.getStamp();//获取版本号
            System.out.println("b1="+stamp);

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            atomicStampedReference.compareAndSet(1,6,stamp,stamp+1);
            //版本号已经变了，所以修改不会成功。这种记录版本的思想与乐观锁类似。
            System.out.println("b2="+atomicStampedReference.getStamp());

        },"b").start();
    }
}
```

- 如果把上述代码中的原子引用的值使用-128到127之外的值，就会出现修改失败的情况（即使是单线程操作）。这涉及到一个坑

坑：Integer使用对象缓存机制，默认范围是 -128-127，推荐使用静态工厂方法valueOf获取对象实例，而不是new，因为valueOf使用缓存，而new一定会创建新的对象分配新的内存空间。

相同类型的包装类之间的值比较尽量使用equals，因为对于Integer var = ?在-128到127之间的赋值，Integer对象是在IntegerCache.cache中产生，会复用已有对象，这个区间的Integer值可以直接使用==进行判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象。这是个大坑。



# 21 锁的理解

## 1 公平锁、非公平锁

公平锁：非常公平，不能插队，必须先来后到，获取不到锁的时候，会自动加入队列，等待线程释放后，队列的第一个线程获取锁

非公平锁：首先是检查并设置锁的状态，这种方式会出现即使队列中有等待的线程，**但是新的线程仍然会与排队线程中的队头线程竞争（但是排队的线程是先来先服务的），所以新的线程可能会抢占已经在排队的线程的锁，这样就无法保证先来先服务，但是已经等待的线程们是仍然保证先来先服务的。**

```java
Lock lock = new ReentantLock();//默认非公平锁
Lock lock = new ReentantLock(true);//改为公平锁
```

### Synchronized是公平还是非公平？

是非公平锁，可重入。

## 2 可重入锁

[重要易懂的补充材料](https://www.cnblogs.com/gxyandwmm/p/9387833.html)

可重入锁：该锁支持同一个线程对同一资源的重复加锁。也就是一个线程获得锁之后可以再次获得该锁。

同一个线程可以反复获取同一把锁多次，然后需要释放多次。

所有的锁都是可重入锁，也叫递归锁

### Lock版：

```java
package com.kuang.lock;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Demo02 {
    public static void main(String[] args) {
        Phone2 phone = new Phone2();
        new Thread(()->{
            phone.sms();
        },"a").start();

        new Thread(()->{
            phone.sms();
        },"b").start();
    }
}
class Phone2{
    Lock lock = new ReentrantLock();
    public synchronized void sms(){
        lock.lock();// 锁必须配对（也就是一个lock.lock()对应一个lock.unlock()），否则就会死在里面，
        try {
            System.out.println(Thread.currentThread().getName()+"sms");
            call(); //这里也有锁    
        }finally {
            lock.unlock();
        }
        
        
    }
    public synchronized void call(){
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName()+"call");    
        }finally {
            lock.unlock();
        }
        
    }
}
```

### Synchronized版：

```java
package com.kuang.lock;

public class Demo01 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(()->{
            phone.sms();
        },"a").start();

        new Thread(()->{
            phone.sms();
        },"b").start();
    }
}
class Phone{
    public synchronized void sms(){
        System.out.println(Thread.currentThread().getName()+"sms");
        call(); //这里也有锁
    }
    public synchronized void call(){
        System.out.println(Thread.currentThread().getName()+"call");
    }
}
```

## 3 自旋锁

应用实例：

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

不断尝试直到成功。

自己实现自旋锁：

```java
package com.kuang.lock;

import sun.security.provider.ConfigFile;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.atomic.AtomicStampedReference;


public class Demo3{
    public static void main(String[] args) throws InterruptedException {

        SpinLock lock = new SpinLock(); //自己使用CAS实现的锁

        new Thread(()->{
            lock.myLock();
            try{
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.myUnlock();
            }

        },"t1").start();

        TimeUnit.SECONDS.sleep(1);//主线程休眠一秒，保证线程t1在t2之前先获取到锁

        new Thread(()->{
            lock.myLock();
            try{
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.myUnlock();
            }
        },"t2").start();
    }
    //解析：t1线程先执行myLock（），将atomicReference更改了值，此时t2线程也执行myLock（），
    // 但是由于t1已经修改了atomicReference，expect不是null，因此线程t2会一直在myLock（）的while处因为执行失败而循环，也就是自旋
    //直到t1执行unMylock（），修改了atomicReference为null，此时线程t2才可以从while循环处执行成功，继续执行。这就是自旋锁
}
/**
 * 自旋锁
 */
class SpinLock {

    AtomicReference<Thread> atomicReference = new AtomicReference<>(); //此时atomicRefernece的初始值为null。
    //加锁
    public void myLock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"==>myLock");

        //自旋锁
        while(!atomicReference.compareAndSet(null,thread)){  //如果这个操作不成功就一直重复尝试

        }
    }
    //解锁
    public void myUnlock(){
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName()+"==>myUnlock");
        atomicReference.compareAndSet(thread,null);
    }
}

```

## 4 死锁

> 是什么

```java
package com.kuang.deadlock;

import java.util.concurrent.TimeUnit;

public class Demo01 {
    public static void main(String[] args) {
        String la= "locka";
        String lb = "lockb";
        new Thread(new MyThread(la,lb),"线程A").start();
        new Thread(new MyThread(lb,la),"线程B").start();

    }
}

class MyThread implements Runnable{
    private String lockA;
    private String lockB;

    public MyThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA){  //锁住lockA对象
            System.out.println(Thread.currentThread().getName()+"获得"+lockA);
            System.out.println("想获取"+lockB);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lockB){  //锁住lockB对象
                System.out.println(Thread.currentThread().getName()+"获得"+lockB);

            }
        }
    }
}
```



[怎么排查？](https://blog.csdn.net/u013250071/article/details/80496623)

1. 使用  <u>jps -l</u>  定位进程号
2. 使用 <u>jstack 进程号</u> 找到死锁问题



