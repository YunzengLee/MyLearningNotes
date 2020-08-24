# 目录

1. 线程简介
2. 线程实现（重点）
3. 线程状态
4. 线程同步（重点）
5. 通信问题
6. 高级主题

# 线程简介

- 一边吃饭一边玩手机，看起来很多个任务同时进行，实际上大脑在同一时间依旧只做一件事情。
- 进程是执行程序的一次执行过程，是一个动态概念，是系统资源分配的单位。
- 一个进程可以包含多个线程，至少有一个线程，线程是cpu调度和执行的单位。
- 注意：很多多线程是模拟出来的，真正的多线程是指有多个cpu，即多核，如服务器。如果是模拟出来的多线程即在一个cpu的情况下，同一时间cpu只执行一个代码，只是切换快，就有同时执行的错觉。

核心：

- 线程就是独立的执行路径
- 程序运行时，后台会有多个线程如main线程，GC线程
- main称为主线程，是系统的入口，用于执行整个程序
- 如果一个进程开辟了多个线程，线程的运行由调度器（CPU）安排调度，调度器是与操作系统密切相关的，先后顺序是不能人为干预
- 对同一资源操作时，需要加入并发控制
- 线程会带来额外的开销，比如cpu调度时间，并发控制开销
- 每个线程在自己的工作内存交互，内存控制不当会造成数据不一致

# 线程创建

- 继承Thread
- 实现Runnable
- 实现Callable

## 1 Thread

- 自定义类继承Thread
- 重写run方法
- 创建线程对象，调用start方法启动线程

简单使用：

```java
package com.kuang.demo01;
public class TestThread extends Thread{
    public void run(){
        //run方法线程体
        for(int i=0 ;i<200;i++){
            System.out.println("我在看代码"+i)
        }
    }
    public static void main(String[] args){
        //main线程 主线程
        
        TestThread testThread1 = new TestThread();
        
        testThread.start();//启动线程。如果此处使用run方法，那么就是普通的方法调用，不是两个线程同时跑
        for(int i=0;i<1000;i++){
            System.out.println("我在学习多线程"+i);
        }
        //电脑是单核，所以main线程和另一个线程是交替执行的。
    }
}
```

注意：线程开启不一定马上执行，是由cpu调度。



使用多线程下载图片：

```java
package com.kuang.demo01;
import org.apache.commons.io.FIleUtils;//这个来自一个jar包，需要手动下载并导入工程
public class TestThread2 extends Thread{
    private String url;//图片地址
    private String name;//保存的文件名
    public TestThread2(String url, String name){
        this.url=url;
        this.name=name;
    }
    public void run(){
        WebDownloader.webDownloader = new WebDownloader();
        webDownloader.downloader(url,name);
        System.out.println("下载了图片名为"+name);
    }
    public static void main(String[] args){
        TestThread2 t1 = new TestThread2("https://bkimg.cdn.bcebos.com/pic/9f2f070828381f308bf309a3ae014c086f06f0d6?x-bce-process=image/resize,m_lfit,w_480,limit_1","1.jpg");
        TestThread2 t2 = new TestThread2("https://bkimg.cdn.bcebos.com/pic/9f2f070828381f308bf309a3ae014c086f06f0d6?x-bce-process=image/resize,m_lfit,w_480,limit_2","2.jpg");
        TestThread2 t3 = new TestThread2("https://bkimg.cdn.bcebos.com/pic/9f2f070828381f308bf309a3ae014c086f06f0d6?x-bce-process=image/resize,m_lfit,w_480,limit_3","3.jpg");
        t1.start();
        t2.start();
        t3.start();
    }
    
}
//下载器
class WebDownloader{
    //
    public void downloader(String url,String name){
        FileUtils.copyURLToFile(new URL(url), new File(name));
        
    }
}
```

## 2 Runnable

```java
package com.kuang.demo01;
public class TestThread3 implements Runnable{
    @Override
    public void run(){
        //run方法线程体
        for(int i=0 ;i<200;i++){
            System.out.println("我在看代码"+i)
        }
    }
    public static void main(String[] args){
        //main线程 主线程
        
        TestThread3 t3 = new TestThread3();
        Thred thread = new Thread(t3);
        thread.start();
        
        for(int i=0;i<1000;i++){
            System.out.println("我在学习多线程"+i);
        }
        //电脑是单核，所以main线程和另一个线程是交替执行的。
    }
}
```

## 比较

- 继承Thread
  - 子类继承Thread
  - 子类对象.start()启动线程
  - 不建议使用：**避免oop单继承局限性**
- 实现Runnable
  - 实现Runnable
  - 启动：传入目标对象+Thread对象.start()
  - 推荐使用：**避免单继承局限性，灵活方便，方便同一个对象被多个线程使用**

## 多线程操作同一对象

```java
//多线程操作同一对象
//买火车票的例子
public class TestThread4 implements Runnable{
    
    private int ticketNums=10;
    @Override
    public void run(){
        while(true){
            if(ticketNums<=0){
                break;
            }
            Thread.sleep(200);//200毫秒 ，模拟延时
            System.out.println(Thread.currentThread().getName()+"--》拿到了"+ticketNums--+"票");
        }
    }
    
    public static void main(String[] args){
        TestThread ticket = new TestThread4();
        new Thread(ticket,"小米").start();
        new Thread(ticket,"老师").start();
        new Thread(ticket,"黄牛").start();
    }
}
```

- **发现问题**：多个线程操作同一资源的情况下，线程不安全，数据混乱。（上个代码运行可以看到两个线程可能拿到同一张票）

## 龟兔赛跑

```java
public class Race implements Runnable{
    private static String winner;
    @Override
    public void run(){
        for(int i=0;i<=100;i++){
            //模拟兔子休息
            if (Thread.currentThread().getName=="兔子"&& i%10==0){
                Thread.sleep(10);
            }
            //判断比赛是否结束
            boolean flag = gameOver(i);
            if (flag){
                break;//比赛结束就停止程序
            }
            System.out.println(Thread.currentThread().getName()+"跑了"+i+"步");
        }
    }
    private boolean gameOver(int steps){
        if (winner!=null){//存在胜利者了
            return true;
        }{
            if(steps>=100){
                winner = Thread.currentThread().getName();
                System.out.println("winner is "+winner);
            }
        }
        return false;
    }
    
    public static void main(String[] args){
        Race r = new Race();
        new Thread(r,"兔子").start();
        new Thread(r,"乌龟").start();
    }
}
```

## 3 实现Callable

```java
package com.kuang.demo02;
public class TestCallable implements Callable<Boolean>{
    
    private String url;//图片地址
    private String name;//保存的文件名
    public TestCallable(String url, String name){
        this.url=url;
        this.name=name;
    }
    public Boolean call(){
        WebDownloader.webDownloader = new WebDownloader();
        webDownloader.downloader(url,name);
        System.out.println("下载了图片名为"+name);
        return true;
    }
    public static void main(String[] args){
        TestCallable t1 = new TestCallable("https://bkimg.cdn.bcebos.com/pic/9f2f070828381f308bf309a3ae014c086f06f0d6?x-bce-process=image/resize,m_lfit,w_480,limit_1","1.jpg");
        TestCallable t2 = new TestCallable("https://bkimg.cdn.bcebos.com/pic/9f2f070828381f308bf309a3ae014c086f06f0d6?x-bce-process=image/resize,m_lfit,w_480,limit_2","2.jpg");
        TestCallable t3 = new TestCallable("https://bkimg.cdn.bcebos.com/pic/9f2f070828381f308bf309a3ae014c086f06f0d6?x-bce-process=image/resize,m_lfit,w_480,limit_3","3.jpg");
        
        //创建执行服务
        ExectorService ser = Executors.newFixedThreadPool(3);//new一个池子，里面放三个线程
        
        //提交执行
        Future<Boolean> r1 = ser.submit(t1);
        Future<Boolean> r2 = ser.submit(t2);
        Future<Boolean> r3 = ser.submit(t3);
        //获取结果
        boolean rs1 = r1.get();
        boolean rs2 = r2.get();
        boolean rs3 = r3.get();
        //关闭服务
        ser.shutdownNow();
    }
    
}
//下载器
class WebDownloader{
    //
    public void downloader(String url,String name){
        FileUtils.copyURLToFile(new URL(url), new File(name));
        
    }
}
```

- Callable好处：
- 可以定义返回值
- 可以抛出异常



## 静态代理模式

```java
//总结：真实对象和代理对象都要实现同一个接口，代理对象要代理真实角色
//好处：
	//代理对象可以做很多真实对象做不了的事情
	//真实对象就关注自己的事情
//例子如下

public class StaticPoxy{	
    public static void main(String[] args){
       
        You you = new You();
        WeddingCompany weddingCompany = new WeddingCompany(you);
        weddingCompany.HappyMarry();
    		
        
        //与多线程比较写法,可以发现，实现Runnable接口的对象放入Thread类，这就是代理模式
        new Thread(()->System.out.println("爱你")).start();
        new WeddingCompany(new You()).HappyMarry();
    }
}

interface Marry{
    void HappyMarry();
}
//真实角色
class You implements Marry{
    public void HappyMarry(){
        System.out.println("marry!")
    }
}
//代理角色
class WeddingCompany implements Marry{
    //代理谁，也就是真实角色
    private Marry target;
    
    public WeddingCompany(Marry target){
        this.target= target;
    }
    public void HappyMarry(){
        before();
        this.target.HappyMarry();
        after();
    }
    private void before(){
        System.out.println("布置场地");
    }
    private void after(){
        System.out.println("收尾款");
    }
}
```

## Lambda表达式



- 避免内部类定义过多
- 使代码看起来简洁
- 去掉没有意义的代码，只留下核心逻辑

函数式接口：

- 定义：任何接口，如果只包含唯一一个抽象方法，就是一个函数式接口

  ```java
  public interface Runnable{
      public abstract void run();
  }
  ```

- 对于函数式接口，可以使用lambda表达式来创建该接口的对象



```java
public class TestLambda{
    
    //3.静态内部类
    static class Like2 implements ILike{
        public void lambda(){
        	System.out.println("lambda2");
    	}
    }
    
    public static void main(String[] args){
        ILike like = new Like();
        like.lambda();
        
        like = new Like2();
        like.lamdba();
        
        //4.局部内部类
        class Like3 implements ILike{
        	public void lambda(){
        		System.out.println("lambda3");
    		}
   		}
        like= new Like3();
        like.lambda();
        
        //5.匿名内部类,没有类的名称，必须借助接口或父类
        like = new ILike(){
            public void lambda(){
                System.out.println("lambda4");
            }
        }
        like.lambda();
        
        //6. jdk8：用lambda简化,由于函数式接口仅一个方法，所以省略方法名
        like = ()->{
                System.out.println("lambda5");
            };
        like.lambda();
    }
}
//1.定义函数式接口
interface ILike{
    void lambda(); 
}
//2.实现类
class Like implememnts ILike{
    public void lambda(){
        System.out.println("lambda1");
    }
}
```

```java
//lambda的再简化
ILove love = (int a)->{
    System.out.println("sss"+a);
}
love.方法名();

//简化参数类型
love = (a)->{
    System.out.println("sss"+a);
}
//简化括号
love = a->{
    System.out.println("sss"+a);
}

//简化大括号（前提是代码只有一行，多行代码不能去掉大括号）
love = a-> System.out.println("sss"+a);

//总结：使用lambda的前提是函数式接口。多个参数也可以去掉参数类型，要去掉就全部去掉，多参数情况下不能去掉小括号。



```

# 线程状态

创建、就绪、运行、阻塞、死亡

new->创建

start->就绪但不一定立刻执行

就绪状态下调度->运行状态

运行下调用sleep wait或同步锁定时->阻塞状态（代码不往下执行，阻塞状态解除后，重新进入就绪状态，等待cpu调度）

运行下中断或结束->死亡状态

事实上，就绪和运行在官方说法是合并为一个状态RUNNABLE

WAITING：等待另一个线程执行特定动作的线程处于此状态

TIMED_WAITING：等待另一个线程执行动作达到指定等待时间的线程处于此状态

| 方法                           | 说明                           |
| ------------------------------ | ------------------------------ |
| setProperty(int newPriority)   | 更改线程优先级                 |
| static void sleep(long millis) | 让当前线程休眠xx毫秒           |
| void join()                    | 等待该线程终止                 |
| static void yield()            | 暂停当前线程，并执行其他线程   |
| void interrupt()               | 中断线程（不推荐使用这种方式） |
| boolean isAlive()              | 测试线程是否处于活动状态       |

## 线程停止

推荐线程自己停止下来

建议使用一个标志位进行终止变量，当flag=false，则终止线程运行

```java
//1. 建议线程正常停止--》利用次数，不建议死循环
//2、建议使用标志位---》设置一个标志位
//3. 不要使用stop destroy等过时或jdk不建议的方法
public class TestStop implements Runnable{
    //1 设置标志位
    private boolean flag=true;
    @Override
    public void run(){
        int i=0;
        while(flag){
            System.out.println("run."+i++);
            
        }    
    }
    
    //2 设置一个公开方法停止线程
    public void stop(){
        this.flag=false
    }
    
    public static void main(String[] args){
        TestStop testStop = new TestStop();
        new Thread(testStop).start();//启动线程
        
        for (int i=0;i<1000;i++){
            if (i==900){  //当这个主线程到达900时，更改另一个线程的标志位
                testStop.stop();//调用方法切换标志位
                System.out.println("stop");
            }
        }
    }
}
```

## 线程休眠

- sleep指定当前线程阻塞的毫秒数
- 时间达到后进入就绪状态
- 每一个对象都有一个锁，sleep不会释放锁

```java
//模拟网络延时
public class TestSleep implements Runnable{
    private int ticketNums=10;
    @Override
    public void run(){
        while(true){
            if(ticketNums<=0){
                break;
            }
            Thread.sleep(200);//200毫秒 ，模拟延时,是为了放大问题的发生性，会发生线程不安全的问题
            System.out.println(Thread.currentThread().getName()+"--》拿到了"+ticketNums--+"票");
        }
    }
    
    public static void main(String[] args){
        TestThread ticket = new TestThread4();
        new Thread(ticket,"小米").start();
        new Thread(ticket,"老师").start();
        new Thread(ticket,"黄牛").start();
    }
}
```

```java
//模拟计时
public TestSleep2{
    
    public static void main(String[] args){
        tenDown();
        
        //打印当前系统时间
        Date startTime = new Date(System.currentTimeMillis());//获取当前系统时间
        while(true){
            Thread.sleep(1000);
            System.out.println(new SimpleDateFormat("HH:mm:ss").format(startTime))
            startTime = new Date(System.currentTimeMillis());//更新当前系统时间
        }
    }
    //模拟倒计时
    public void tenDown(){
        int num=10;
        
        
        
        while(true){
            Thread.sleep(1000);
            System.out.println(num--);
            if(num<=0){
                break;
            }
        }
    }
}
```

## 线程礼让

- 当前线程暂停但不阻塞
- 从运行转为就绪态
- **让cpu重新调度；礼让不一定成功！**

```java
//测试礼让线程
public class TestYield{
    public static void main(){
        MyYield myYield = new MyYield();
        new Thread(myYield,"a").start();
        new Thread(myYield,"b").start();
    }
}
class MyYield implements Runnable{
    public void run(){
        System,out.println(Thread.currentThread().getName()+"开始");
        Thread.yield();//礼让
        System,out.println(Thread.currentThread().getName()+"结束");
        
    }
}
```

## 线程强制执行

- join合并线程，待此线程执行完后，再执行其他线程，其他线程阻塞
- 想象成插队
- 一个霸道的方法，x.join()强制执行线程x，直到x线程结束

```java
//测试join方法
public class TestJoin implements Runnable{
    public void run(){
       for(int i=0;i<100;i++){
           System.out.prrintln("线程vip来了"+i);
       }
    }
    
    public static void main(){
        //启动我们的线程
        TestJoin testJoin = new TestJoin();
        Thread t = new Thread(testJoin);
        t.start();
        
        //主线程
        for(int i =0;i<1000;i++){
            if(i==200){
                t.join();//插队
            }
            System.out.println("main"+i);
        }
    }
}

```

## 查看线程状态

```java
public class TestState{
    
    public static void main(String[] args){
        Thread t = new Thread(
            ()->{
                for(int i=0;i<5;i++){
                    Thread.sleep(1000);//sleep时线程处于TIMED_WAITING状态
                }
                System.out.println("...");
            }
        );
        //观察状态
        Thread.State state = t.getState();
        System.out.println(state);
        
        t.start();
        state = t.getState();
        System.out.println(state);
        
        while(state!=Thread.State.TERMINATED){
            Thread.sleep(100);
            state = t.getState();
        	System.out.println(state);
        }
    }
    
}
```

## 线程优先级

- Java提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程。调度器按照优先级决定哪个线程执行
- 优先级只是提高了执行的概率，不是一定先执行
- 优先级用数字表示，1-10
  - Thread.MIN_PRIORITY =1;
  - Thread.MAX_PRIORITY =10;
  - Thread.NORM_PRIORITY =5;
- 获取或改变优先级
  - getPriority()  setPriority(int xx)

```java
public class TestPriority{
    public static void main(){
        //主线程
        System.out.println(Thread.currentThread().getName()+"-->"+Thread.currentThread().getPriority());
        
        MyPriority myPriority = new MyPriority();
        Thread t1 = new Thread(myPriority);
        Thread t2 = new Thread(myPriority);
        Thread t3 = new Thread(myPriority);
        Thread t4 = new Thread(myPriority);
        Thread t5 = new Thread(myPriority);
        //先设置优先级 再启动
        t1.start();
        
        t2.setPriority(1);
        t2.start();
        t3.setPriority(4);
        t3.start();
        t4.setPriority(10);
        t4.start();
        t5.setPriority(-1);//报错
        t5.start();
    }
}

class MyPriority implements Runnable{
    public void run(){
        System.out.println(Thread.currentThread().getName()+"-->"+Thread.currentThread().getPriority());
    }
}
```

## 守护（daemon）线程

- 线程分为用户线程和守护线程
- 虚拟机必须确保用户线程执行完毕，不必等待守护线程执行完毕（比如gc线程，监控内存）

```java
//测试守护线程
public class Test{
    public static void main(String[] args){
        God god = new God();
        You you =new You();
        Thread thread new Thread(god);
        thread.setDaemon(true);//设为守护线程，默认为用户线程（false）
        thread.start();//守护线程启动
        new Thread(you).start();//用户线程启动
    }
}

class God implements Runnable{
    public void run(){
        while(true){
            System.out.println("上帝保佑着你");
        }
    }
}
//你
class You implements Runnable{
    public void run(){
        for(int i=0;i<36500;i++){
            System.out.println("一天过去");
        }
        System.out.println("====bye===");
    }
}
```

以上代码测试发现用户线程结束后程序就结束，程序不会等待守护线程执行完毕。

# 线程同步

- 并发：同一个对象被多个线程同时操作，比如上万人同时抢100张票
- 解决的办法就是排队，就像多个人去食堂同一窗口打饭
- 处理多线程问题时，多线程同时访问同一资源，就需要线程同步，线程同步本质上就是一种等待机制，多个需要同时访问某对象的线程进入该对象的等待池形成队列，等待前面的线程处理完毕。

- 队列和锁：光排队还不够，还需啊哟锁。每个对象都有一把锁，

由于同一进程的所有线程共享同一存储空间，带来了访问冲突问题，为了保证数据在方法中被访问时的正确性，在访问时加入了锁机制（synchronized），当一个线程获得对象的排他锁，其他线程必须等待，使用完释放即可。存在以下问题。

- 其他需要此锁的线程挂起
- 加锁释放锁会导致较多的上下文切换，和调度延时，引起性能问题
- 如果优先级高的线程等待一个优先级低线程的锁，会导致优先级倒置，引起性能问题。

## 不安全案例

1. 

```java
public class Unsafe{
    public static void main(){
    	BuyTicket station = new BuyTicket();
        new Thread(station,"我").start();
        new Thread(station,"你").start();
        new Thread(station,"黄牛").start();
    }
     
}

class BuyTicket implements Runnable{
    private int ticketnum = 10;
    boolean flag = true
    public void run(){
        //买票
        while(flag){
            buy();
        }
        
    }
    private void buy(){   //private后面加个synchronized就可以解决不安全问题
        if (ticketnum<=0){
            return;
        }
       Thread.sleep(100);//模拟延时
 //买票
        System.out.println(Thread.currentThread().getName()+"拿到"+ticketnum--);
    }
 	           
}
```

- 运行发现有负数，线程不安全（负数怎么来的？当多个线程都看到有一张票时，都去拿，结果一共拿走了3张，就出现负数）

2. 

```java
public class UnsafeBank{
    public static void main(){
        //账户
        Account a = new Account(100,"结婚基金");
        
        Drawing you = new Drawing(account,50,"你");
        Drawing girl = new Drawing(account,100,"girlFriend");
        you.start();
        girl.start();
        
    }
}
//账户
class Account{
    int money;
    String name;
    //有参构造
    public Account(int money,String name){
        this.name=name;
        this.money=money;
    }
    //getter  setter
}
//银行;模拟取款
class Drawing extends Thread{
    Account account;
    int drawingMoney;
    int nowMoney;
    
    public Drawing(Account account, int drawingMoney,String name){
        super(name);//调用父类Thread的方法给线程一个名字
        this.account=account;
        this.drawingMoney=drawingMoney;
        
    }
    
    //取钱
    public void run(){ //
        if(account.money-drawingMoney<0){
            System,out.println(Thread.currentThread().getName()+"钱不够取");
            return;
        }
        //模拟延时
        Thread.sleep(100);
        
        //卡内余额减去取的钱
        account.money -= drawingMoney;
        //手里的钱
        nowMoney=nowMoney+drawingMoney;
        
        
        System.out.println(account.name+"余额为"+account.money);
        //Thread.currentThread()=this
        System.out.println(this.getName()+"手里的钱："+nowMoney);
    }
    
}
```

- 运行发现余额会变负数。（sleep可以放大问题的发生性）

3. 

```java
//线程不安全的集合
public class UnsafeList{
    public static void main(){
        List<String> list = new ArrayList<>
();
        for(int i =0;i<10000;i++){
            new Thread(
            ()->{
                list.add(Thread.currentThread().getName());
            }
            ).start();
        }
        Thread.sleep(100);
        
    	System.out.println(list.size());
        
    }
}
```

- 结果发现list的size小于10000，这是因为多个线程同时对同一个位置存放元素，导致覆盖。

## 同步方法和同步块

- 由于可以通过private保证数据对象只能通过方法访问，只需要对方法提出一套机制，这套机制就是synchronized，它有两种用法：synchronized方法和synchronized代码块
  - public synchronized void method(){}
- synchronized方法控制对对象的访问，每个对象有一把锁，每个synchronized方法都必须获得该所才能执行，否则线程阻塞。方法一旦执行就独占该锁直到方法返回才释放，后面的线程才能获得这个锁，继续执行。
  - 缺陷：将一个大的方法申明为synchronized会影响效率

- 方法里面需要修改的内容才需要锁（读内容不需要锁），锁的太多浪费资源。
- 同步块：synchronized (Obj){}
  - Obj称为同步监视器，可以是任何对象，但是推荐使用共享资源作同步监视器
  - 同步方法无需指定同步监视器，因为同步方法的监视器就是this，也就是该方法所属的这个对象本身，或者是class（反射中讲解）
  - 同步监视器执行过程
    - 第一个线程访问，锁定同步监视器，执行代码
    - 第二线程访问，发现同步监视器被锁定，无法访问
    - 第一个线程执行完毕释放同步监视器
    - 第二个线程访问，发现同步监视器没有锁，然后锁定并访问

### 解决以上三个不安全案例：

1. 第一个只需在方法上加synchronized关键字

2. 第二个案例中扣款操作是在run方法里，但这个方法属于对象Drawing，所以在这个方法上加synchronized不管用;需要使用synchronized代码块，代码块传入的对象是account，表示锁的是account这个对象

   ```java
   public class UnsafeBank{
       public static void main(){
           //账户
           Account a = new Account(100,"结婚基金");
           
           Drawing you = new Drawing(account,50,"你");
           Drawing girl = new Drawing(account,100,"girlFriend");
           you.start();
           girl.start();
           
       }
   }
   //账户
   class Account{
       int money;
       String name;
       //有参构造
       public Account(int money,String name){
           this.name=name;
           this.money=money;
       }
       //getter  setter
   }
   //银行;模拟取款
   class Drawing extends Thread{
       Account account;
       int drawingMoney;
       int nowMoney;
       
       public Drawing(Account account, int drawingMoney,String name){
           super(name);//调用父类Thread的方法给线程一个名字
           this.account=account;
           this.drawingMoney=drawingMoney;
           
       }
       
       //取钱
       public void run(){ //
           //锁的对象是变化的量
           synchronized (account){
           	if(account.money-drawingMoney<0){
                   System,out.println(Thread.currentThread().getName()+"钱不够取");
                   return;
               }
               //模拟延时
               Thread.sleep(100);
   
               //卡内余额减去取的钱
               account.money -= drawingMoney;
               //手里的钱
               nowMoney=nowMoney+drawingMoney;
   
   
               System.out.println(account.name+"余额为"+account.money);
               //Thread.currentThread()=this
               System.out.println(this.getName()+"手里的钱："+nowMoney);    
               }
           
       }
       
   }
   ```

3. ```java
   //线程不安全的集合解决办法
   public class UnsafeList{
       public static void main(){
           List<String> list = new ArrayList<>
   ();
           for(int i =0;i<10000;i++){
               new Thread(
               ()->{
                   synchronized (list){//将修改操作放到synchronized代码块中，同时锁住list对象
                   	list.add(Thread.currentThread().getName());    
                   }
                   
               }
               ).start();
           }
           Thread.sleep(100);
           
       	System.out.println(list.size());
           
       }
   }
   ```

## JUC安全类型的集合CopyOnWriteArrayList

juc就是java.util.concurrent包

````java
import java.util.concurrent.CopyOnWriteArrayList;
public class TestJUC{
    public void main(){
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
        for(int i=0;i<1000;i++){
            new Thread(
            	()->{
                    list.add(Thread.currentThread().getName());
                }
                
            ).start();
        }
        
        Thread,sleep(3000);
        System.out.println(list.size());
    }
}
````

## 死锁

多个线程都在等待对方释放资源，都停止执行。某一个同步块**同时拥有两个以上对象的锁**时，可能发生死锁。

```java
public class DeadLock{
    public static void main(){
        Makeup g1 = new Makeup(0,"灰姑娘");
        Makeup g2 = new Makeup(1,"白雪公主");
        g1.start();
        g2.start();
    }
}

//口红
class Lipstick{
    
}
//镜子
class Mirror{
    
}
//
class Makeup extends Thread{
    
    //需要的资源只有一份，用static保证只有一份
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();
    
    int choice;
    String girlName;
    Makeup(int choice, String girlName){
        this.choice= choice;
        this.girlName=girlName;
    }
    public void run(){
        makeup();
    }
    
    //化妆，互相持有对方所需的资源
    private void makeup(){
        if(choice==0){
            synchronized (lipstick){//获得口红的锁
                System.out.println("获得口红的锁");
                Thread.sleep(1000);
            	
                synchronized(mirror){
                    System.out.println("一秒后获得镜子的锁");
                }
            }
            
        }else{
            synchronized (mirror){//获得镜子的锁
                System.out.println("获得镜子的锁");
                Thread.sleep(1000);
            	
                synchronized(lipstick){
                    System.out.println("一秒后获得口红的锁");
                }
            }
            
        }
    }
}
```



运行发现程序会卡住不动。

改进：只需要把内部的syn代码块拿到外部syn代码块的同级即可：

```java
public class DeadLock{
    public static void main(){
        Makeup g1 = new Makeup(0,"灰姑娘");
        Makeup g2 = new Makeup(1,"白雪公主");
        g1.start();
        g2.start();
    }
}

//口红
class Lipstick{
    
}
//镜子
class Mirror{
    
}
//
class Makeup extends Thread{
    
    //需要的资源只有一份，用static保证只有一份
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();
    
    int choice;
    String girlName;
    Makeup(int choice, String girlName){
        this.choice= choice;
        this.girlName=girlName;
    }
    public void run(){
        makeup();
    }
    
    //化妆，互相持有对方所需的资源
    private void makeup(){
        if(choice==0){
            synchronized (lipstick){//获得口红的锁
                System.out.println("获得口红的锁");
                Thread.sleep(1000);
            }
            synchronized(mirror){
                System.out.println("一秒后获得镜子的锁");
            }
            
        }else{
            synchronized (mirror){//获得镜子的锁
                System.out.println("获得镜子的锁");
                Thread.sleep(1000);
            }
            synchronized(lipstick){
               System.out.println("一秒后获得口红的锁");
            }
            
        }
    }
}
```

## 死锁条件

1. 互斥条件
2. 请求保持
3. 不剥夺条件
4. 循环等待

## Lock

- jdk5开始，Java提供了更强大的线程同步机制——通过显式定义同步锁对象来实现同步，同步锁使用Lock对象充当。
- java.util.concurrent.locks.Lock接口是控制多个线程对共享资源进行访问的工具。锁提供了对象共享资源的独占访问，每次只能有一个线程对Lock对象加锁，线程开始访问共享资源之前应先获得Lock对象
- ReentrantLock（可重入锁）实现了Lock，拥有与synchronized相同的并发性和内存语义，比较常用，可以显式加锁和释放

```java
public class TestLock{
    TestLock2 test = new TestLock2();
    new Thread(test).start();
    new Thread(test).start();
    new Thread(test).start();
}
class TestLock2 implements Runnable{
    
    int ticketnum =10;
    
    //定义lock锁
    private final ReentrantLock lock = new ReentrantLock();//final保证它是个常量
    
    public void run(){
        while(true){
            lock.lock();
            if(ticketnum>0){
                Thread.sleep(1000);
                System.out.println(ticketnum--);
            }else{
                break;
            }
            lock.unlock();//显式的加锁解锁
        }
    }
}
```

### synchronized与Lock对比

- 后者是显式锁，手动，前者是隐式锁，出了作用域自动释放
- 后者只有代码块锁，前者有代码块锁和方法锁
- 使用后者，java将花费较少的时间来调度线程，性能更好，且具有更好的扩展性（提供更多的子类）
- 优先使用顺序：
  - lock>同步代码块>同步方法

# 线程通信（协作）

## 生产者消费者模型



- 线程同步问题，二者共享一个资源，又相互依赖互为条件
  - 对于生产者，没有生产产品之前通知消费者等待，生产之后又要通知消费者消费
  - 对于消费者，消费之后要通知生产者已经结束消费，需生产新的产品
  - 仅有synchronized不够的
    - 可以阻止并发更新同一个资源，实现同步
    - 但无法实现不同线程之间的消息传递

java提供了一些通信方法

| 方法               | 作用                                                         |
| ------------------ | ------------------------------------------------------------ |
| wait()             | 表示线程一直等待，直到其他线程通知，与sleep不同的是，wait会释放锁 |
| wait(long timeout) | 指定等待时间                                                 |
| notify()           | 唤醒一个等待的线程                                           |
| notifyAll()        | 唤醒同一个对象上所有调用wait方法的 线程，优先级高的线程优先调度 |



### 管程法

**生产者-缓冲区-消费者**

```java
//利用缓冲区解决：管程法
public class TestPc{
    public static void main(){
        SynContainer container = new SynContainer();
        new Productor(container).start();
        new Consumer(container).start();
        
    }
}

//
class Productor extends Thread{
    SynContainer container;
    public Productor(SynContainer container){
        this.container = container;
    }
    
    //生产
    public void run(){
        for (inti=0;i<100;i++){
            
            container.push(new Chicken(i));
            System.out.println("生产了第"+i+"只鸡");
        }
    }
}
class Consumer extends Thread{
    SynContainer container;
    public Productor(SynContainer container){
        this.container = container;
    }
    
    //消费
    public void run(){
        for (int i=0;i<100;i++){
            System.out.ptrintln("消费了第"+container.pop().id+"只鸡");
        }
    }
}
//产品
class Chicken{
    int id;
    piublic Chicken(int id){
        this.id = id;
    }
}

//缓冲区
class SynContainer{
    Chicken[] chickens = new Chicken[10];
    int count = 0;
    
    //生产者放入产品
    public synchronized void push(Chicken chicken){
        //如果满了就需要等待消费者消费
        if(count == chickens.length){
            //通知消费者消费，生产等待
            this.wait(); //注意此处的等待是指线程在这里暂停，直到收到通知后，才从此处继续向下进行
        }
        //
        chickens[count] = chicken;
        count++;

        //可以通知消费者消费了
        this.notifyAll();
    }
    //消费者消费产品
    public synchronized Chicken pop(){
        //判断能否消费
        if(count==0){
            //等待生产者生产
            this.wait();
        }
        //如果可以消费
        count--;
        Chicken c = chickens[count];
        //吃完了通知生产者生产
        this.notifyAll();
        return chicken;
    }
}
```



### 信号灯法

```java
//生产者消费者问题：信号灯法：标志位解决
public class TestPc2{
    public static void main(){
        Tv t = new Tv();
        new Player(t).start();
        new Watcher(t).start();
    }
}
//生产者  -演员
class Player extends Thread{
    Tv tv;
    public Player(Tv t){
        this.tv=t;
    }
    
    public void run(){
        for (int i =0;i<20;i++){
            if(i%2==0){
                this.tv.play("快乐大本营");
            }else{
                this.tv.play("抖音");
            }
        }
    }
}
//消费者  -观众
class Watcher extends Thread{
    Tv tv;
    public Watcher(Tv t){
        this.tv=t;
    }
    
    public void run(){
        for (int i =0;i<20;i++){
            tv.watch();
        }
    }
}
//产品--节目
class TV{
    //演员表演，观众等待 true
    //观众观看，演员等待 false
    String voice; //表演的的节目
    boolean flag = true;
    
    //表演
	public synchronized void play(String voice){
        if(!flag){
            this.wait();
        }
        System.out.println("演员表演了"+voice);
        //通知观众观看
        this.notifyAll(); //此处唤醒了另一个线程，但目前还没有释放对象，所以接下来的voice flag的修改是没问题的
        this.voice = voice;
        this.flag = !this.flag;
    }
    
    //观看
    public synchronized void watch(){
        if(flag){
            this.wait();
        }
        System.out.println("观看了"+voice);
        //通知演员表演
        this.notifyAll();
        this.flag=!this.flag;
    }
}
```

# 线程池

- 提前创建多个线程，放入线程池，使用时获取，使用完放回，避免反复创建销毁，实现重复利用。
- 好处：
  - 提升响应速度（减少了创建新线程的时间）
  - 降低资源消耗（重复利用线程池中的资源）
  - 便于线程管理
    - corePoolSize：核心池的大小
    - maximumPoolSize：最大线程数
    - keepAliveTime：线程没有任务时保持多久终止

## 使用线程池

- jdk5提供了线程池相关API ：ExecutorService和Executors
- ExecutorService：真正的线程池接口。常见子类ThreadPoolExecutor
  - void execute(Runnable command):执行任务/命令
  - <T> Future<T> submit(Callable<T> task):执行任务，有返回值
  - void shutdown():关闭连接池
- Executors：工具类，线程池的工厂类，用于创建并返回不同类型的线程池

```java
public class TestPool{
    //1 创建线程池,参数为线程池大小
    ExecutorService service = Executors.newFixedThreadPool(10);
    service.execute(new MyThread());
    service.execute(new MyThread());
    service.execute(new MyThread());
    service.execute(new MyThread());
    service.execute(new MyThread());
    //2 关闭
    service.shutdown();
}

class MyThread implements Runnable{
    public void run(){
        
        System.out.println(Thread.currentThread().getName());
        
    }
}
```

