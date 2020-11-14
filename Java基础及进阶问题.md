### 数组初始化

int [][] arr = new int [2] [3];

int [][] arr = new int [][]{{1,2,3}，{1,2}，{1,2,3,4}}；数组长度可以不相同。

int [][] arr = new int [2] []; 这个是只制定了第一维的长度，第二维不定义。

**注意：一维数组声明时必须声明长度，二维数组同样，但是二维数组是必须声明第一维的长度，第二维的长度无要求。**

**如果是定义为{},**就可以不声明长度，比如：

**int[] arr = {};** **是没有问题的。**



**类中的方法可以直接访问成员变量但是有**例外：**Static方法不能访问非static的成员变量!!!只能访问static的成员变量**

### 整数相除的结果问题

int类型的相除只能得到int类型的结果，即使用一个float类型变量接收，得到的也是int类型结果转化为float的，不包含小数。

比如

```java
int x = 12;
int y = 5;
float res = x/y; // res = 2.0 
float res2 = (float)x/y; //res2 = 2.4,即需要将其中一个数转为float类型才能得到float类型的结果
```



### Collection与Collections的区别

1. java.util.Collection 是集合类的一个顶级接口，提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供了最大化的统一操作方式，其直接继承接口有List与Set。

 Collection  
├List  
│├LinkedList  
│├ArrayList  
│└Vector  
│　└Stack  
└Set

2. Collections则是集合类的一个工具类/帮助类，其中提供了一系列静态方法，用于对集合中元素进行排序、取最值、搜索以及线程安全等各种操作。



### 构造器

- **子类的构造器必须先调用父类的构造器**

- **子类所有构造器默认自动调用父类空参数的构造器（所以，如果父类因为自定义了构造器而没有无参构造器时，子类就需要自己写构造器，并在构造器的首部调用父类的有参构造）**

- 由以上两点可知，若父类没有空参数构造器(可能由于自定义构造器使得无参构造器失效)，子类构造器必须显式的写至少一个构造器，且必须通过**this(参数列表)或者super(参数列表)语句**指定调用本类或父类的相应的构造器，并放在构造器第一行，否则编译出错。



### 实例初始化过程

1. 类加载检查：当JVM遇到new指令时先检查这个指令的参数是否能在常量池定位到这个类的符号引用，并检查这个类是否加载过

2. 分配内存：，从堆中划分一块空间。有两种方式，取决于Java堆是否规整，是否规整又由垃圾收集算法决定。

   - 指针碰撞：已使用的内存和未使用的内存之间有一个分界指针，将该指针向未使用内存方向移动对象内存大小的位置即可。（堆内存规整）
   - 空闲列表：虚拟机维护一个记录着可用内存块的列表，找出一块划分给对象实例，之后更新列表。（堆内存不规整）

   内存分配的并发问题解决（两种方式）：

   - CAS+失败重试
   - TLAB：为每个线程预先在Eden分配一块内存称为TLAB（ThreadLocalAllocationBuffer），首先在TLAB中为对象分配内存，TLAB内存不够时再进行CAS分配。

3. 初始化零值：将分配到的内存空间都初始化为零值（不包括对象头）

4. 设置对象头：对象头存储的是：这个对象属于哪个类，对象的哈希码，对象的GC年龄等。

5. 执行构造方法：把对象按照程序员的意愿进行初始化。

从方法区加载class（先加载父类后加载子类）；在栈内存开辟空间声明变量；在堆内存开辟空间，存储地址；在堆内存中，在对象空间中，将对象属性初始化为默认值（先父类后子类）；调用构造方法进行初始化，子类构造方法入栈内存执行，父类构造方法入栈执行并出栈；初始化结束后，将堆内存地址赋值给引用变量，子类构造方法出栈。

**顺序： 方法区加载class-堆栈中开辟空间- 对象属性初始化默认值- 代码块执行 -构造方法执行





### 面向对象三大特征

- 封装：

  隐藏就是加private，这样的方法或属性只能在自己的类方法中调用。再提供public方法实现对该属性的操作。

  - 隐藏不必对外提供的细节

  - 使用者只能通过定制好的方法访问数据，可以加入控制逻辑避免不合理操作

  - 便于修改，增强代码可维护性。

- 继承

  子类含有父类的所有方法和属性（包括private属性和private方法，只是这些属性或方法无法直接访问）。子类可以根据需要对父类的方法进行改造（重写）。子类可以在父类的基础上进行扩展。

  （子类重写的方法较于父类方法：访问权限只能更大，抛出的异常范围只能更小）

- 多态

  Person p = new Student(); 父类变量可以接收子类实例对象，引用变量p**可以访问子类Studennt中的重写方法，但无法访问子类Student中的重写的属性。方法声明时形参为父类类型，可以将子类实例作为实参传入。** 方法的重载和重写是多态的另一个体现。





### 接口和抽象类的区别

1. **抽象类是对类的抽象，接口是对行为的抽象**
2. 接口的方法默认是public，所有方法不能有实现（1.8开始有默认实现），抽象类可以有非抽象方法
3. 一个类可以**实现多个接口，但只能继承一个类**，接口本身可以通过extends扩展多个接口
4. 本质上来说，**抽象类就是一个普通类中多出了若干抽象方法，接口在java版本中有变化**（7 只有常量变量（final修饰）和抽象方法，8 有默认实现，9 可以有私有方法）
5. （以下两条不必答了）接口中只有static、final变量，而抽象类不一定。
6. 接口方法默认修饰符是public，抽象类的方法可以有public，protected，default三种（private修饰的无法被重写）



### 接口的变化

1. 7之前接口只能有常量变量和抽象方法，不能有实现（抽象方法要么public修饰，要么无修饰符）
2. 8可以有default方法和static方法（也就是使用default和static修饰的方法，必须要有方法体）。
   接口中的static方法实现类无法继承，default方法实现类可以继承。default方法必须由接口实现类实例来调用，default方法可以被重写，重写后权限必须改为public。static由接口直接调用（实现类调用不到）。
3. 9  **私有**  可以有私有方法和静态私有方法



### final关键字

1. 修饰变量，该变量必须显式地初始化，一旦被初始化后就不能更改，引用类型的变量不能更改指向另一个对象，相当于定义了一个常量。
2. 修饰类，该类无法被继承，所有成员方法都会指定为final
3. 修饰方法，继承类无法修改。

继承了Seriable接口的类可以被序列化，给其中的变量加transient关键字可以使该变量不被序列化。

### static关键字

**1.**    因为static方法不用实例化就能调用，因此static方法内部不能有this、super。也就是不能涉及实例变量。

**2.** **重载的方法必须同时为static或非static**。

static 方法不能被重写，因为static方法只能通过类名调用，不是属于对象的。

**Static修饰的初始化块只在第一次new对象时执行一次，而其他非static修饰的代码块和构造方法都是new一次就执行一次。**

1. 修饰成员变量和方法：被修饰的成员属于类，可以通过类名调用，被所有实例共享，存放在方法区。
2. 修饰代码块：该代码块在非静态代码块之前执行，非静态代码块在构造方法之前执行。静态代码块只执行一次（非静态代码块每次实例化都会执行），用于初始化静态变量。
3. 修饰内部类：非静态内部类编译后保存着指向外围类的引用，而静态内部类没有，这说明：**静态内部类创建不依赖于外围类的创建；不能使用外围类的非静态成员**。
4. 静态导包：import static连用可以导入指定类中的静态资源，而不需要使用类名调用，可以直接使用静态成员变量和方法。

### this

1. 指当前类的实例对象，可以调用该对象的属性和方法，或者是用来区分同名形参和自身的属性。
2. 后面加括号代表该类的构造器，根据传入的参数类型和顺序决定哪个构造器，放在某个构造器的首行。

### super

1. 指父类对象，可以调用父类的属性或方法（除了私有属性和方法）
2. 后面加括号代表调用父类的构造器，一定放在自己某个构造器的首行。

### 抽象

将子类的共同特征提取到公共父类中，父类不给出具体的方法实现，由子类根据自己的情况去实现，这就是抽象。

抽象有两种：

- 抽象方法
- 抽象类

有抽象方法的类一定是抽象类。

### String StringBuffer StringBuilder

**可变性**
简单来说，String类中使用final关键字修饰字符数组来保存字符串，private final char value[]，所以String对象不可变。
[issue675](https://github.com/Snailclimb/JavaGuide/issues/675)：Java9之后，String类改用byte数组存储字符串 private final byte[] value，可以节省内存空间。
StringBuilder与StringBuffer都继承自AbstractStringBuilder，在AbstractStringBuilder中使用字符数组保存字符串char[] value但是没有final关键字，所以这两种对象都是可变的。
StringBuilder与StringBuffer的构造方法都是调用父类AbstractStringBuilder的构造方法实现的。
**线程安全性**
String中的对象不可变，是线程安全的。AbstractStringBuilder是上述二者的父类，定义了一些基本操作（append、 insert、 indexOf等）。StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，线程安全；而StringBuilder没有对方法加锁，所以非线程安全。
**性能**
每次对String改变时都会生成新对象并指向新对象。StringBuffer每次都会对StringBuffer对象本身操作不会生成新对象。StringBuilder性能较高但线程不安全。
**三者总结**
1） 操作少量数据用String
2） 单线程操作字符串缓冲区下操作大量数据：适用StringBuilder
3） 多线程操作字符串缓冲区下操作大量数据：适用StringBuffer



### java处理输入输出

```java
package com.imooc;

import com.sun.org.apache.xpath.internal.operations.Bool;

import java.util.*;

import java.io.IOException;
/
public class Test {
    public static void main(String[] args) {
        Data d = new Data();
        new Thread(() -> {
            for (int i = 0; i <= 100; i++) {
                try {
                    d.playSingle();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }, "a").start();
        new Thread(() -> {
            for (int i = 0; i <= 100; i++) {
                try {
                    d.playDouble();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }, "b").start();
    }
}

class Data {
    private int num = 0;

    public synchronized void playSingle() throws InterruptedException {
        if (num % 2 == 0) {
            this.wait();
        }
        System.out.println(Thread.currentThread().getName() + "==" + num++);
        this.notifyAll();
    }

    public synchronized void playDouble() throws InterruptedException {
        if (num % 2 != 0) {
            this.wait();
        }
        System.out.println(Thread.currentThread().getName() + "==" + num++);
        this.notifyAll();
    }
}


class Test3 {
    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);
        System.out.println("请输入你的姓名：");
        String name = sc.nextLine();
        String[] ne = name.split(" ");
        for (int i = 0; i < ne.length; i++) {
            int k = Integer.parseInt(ne[i]);
            String s = String.valueOf(k);
            System.out.println(s);
        }
        char[] chara = name.toCharArray();
        for (int i = 0; i < chara.length; i++) {
            System.out.println(String.valueOf(chara.length));
            System.out.println(String.valueOf(chara[i]));
        }
        System.out.println("请输入你的年龄：");
        int age = sc.nextInt();
        System.out.println("请输入你的工资：");
        float salary = sc.nextFloat();
        System.out.println("请输入你的其他：");
        String desc = sc.next();
        System.out.println("你的信息如下：");
        System.out.println("姓名：" + name + "\n" + "年龄：" + age + "\n" + "工资：" + salary + "\ndesc:" + desc);

        LinkedList<String> l = new LinkedList<String>();
        for (String s : l) {

        }
        List<Integer> k = new ArrayList<>();
        k.add(1);
        for(int i:k){
            System.out.println(String.valueOf(i));
        }

        Map<String,String> m = new HashMap<>();
        m.put("1","2");
        for(Map.Entry<String,String> e : m.entrySet()){
        }
        Set<String> s = new HashSet<>();
        s.add("1");
        for(String st : s){
        }

    }
}
//区间型动态规划
class Solution{
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String input = sc.nextLine();
        String[] info = input.split(" ");
        int num1 = Integer.parseInt(info[0]);
        int num2 = Integer.parseInt(info[1]);
        String input2 = sc.nextLine();
        String[] numList = input2.split(" ");
        int[] nums = new int[num2];
        for(int idx =0; idx<num2;idx++){
            nums[idx] = Integer.parseInt(numList[idx]);
        }
        int res = 0;
        Boolean[][] dp = new Boolean[num2][num2];
        for(int i =0;i<num2;i++){
            if (nums[i]%num1==0){
                dp[i][i] = true;
                res+=1;
            }
        }
        for (int length = 2; length <= num2; length++){
            for(int startIdx = 0; startIdx <= num2 - 2; startIdx++){
                if (startIdx+length>num2){
                    break;
                }
                for(int k=startIdx;k<=startIdx+length-2;k++){
                    dp[startIdx][startIdx+length-1] =dp[startIdx][startIdx+length-1] || dp[startIdx][k] && dp[k+1][startIdx+length-1];
                    if(dp[startIdx][startIdx+length-1]){
                        res+=1;
                        break;
                    }
                }
            }
        }
    }
}
```

### java常用数据结构

```java
Stack<String> s = new Stack<>();
s.push("a");
s.pop();
s.peek();
s.empty();
int capacity = s.size();

LinkedList<Integer> l = new LinkedList<>();
l.addLast(1);
l.getFirst();
l.size();
l.isEmpty();

Queue<String> q = new LinkedList<>();
int size = q.size();
q.offer("a");
q.offer("b");
boolean ise = q.isEmpty();
String sss = q.poll();
String ss = q.peek();

List<String> l = new ArrayList<>();
l.add("a");
l.remove(l.size()-1); //用于深度优先搜索后的pop
l.size();

//深拷贝一个列表,用于深度优先搜索的最后一层，将子结果深拷贝后放到全局结果中
List<String> newList = new ArrayList<>(oldList);
```

### i=i++

无论运行多少次，i都不变，因为这句话运行是这样的，temp = i，i++，i=temp。

### ICMP协议

Internet控制消息协议

注解：该协议是TCP/IP协议集中的一个子协议，属于[网络层](http://www.so.com/s?q=网络层&ie=utf-8&src=internal_wenda_recommend_textn)协议，主要用于在主机与路由器之间传递[控制信息](http://www.so.com/s?q=控制信息&ie=utf-8&src=internal_wenda_recommend_textn)，包括报告错误、交换受限控制和[状态信息](http://www.so.com/s?q=状态信息&ie=utf-8&src=internal_wenda_recommend_textn)等。