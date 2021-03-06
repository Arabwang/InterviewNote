# 面试

[美团面试](https://github.com/Snailclimb/JavaGuide/tree/master/%E9%9D%A2%E8%AF%95%E5%BF%85%E5%A4%87)

 Java 面试问题列表包含的主题：

- 多线程，并发及线程基础
- 数据类型转换的基本原则
- 垃圾回收（GC）
- Java 集合框架
- 数组
- 字符串
- GOF 设计模式
- SOLID （单一功能、开闭原则、里氏替换、接口隔离以及依赖反转）设计原则
- 抽象类与接口
- Java 基础，如 equals 和 hashcode
- 泛型与枚举
- Java IO 与 NIO
- 常用网络协议
- Java 中的数据结构和算法
- 正则表达式
- JVM 底层
- Java 最佳实践
- JDBC
- Date, Time 与 Calendar
- Java 处理 XML
- JUnit
- 编程

## 多线程、并发及线程的基础问题

**1）Java 中能创建 volatile 数组吗？**

能，Java 中可以创建 volatile 类型数组，不过只是一个指向数组的引用，而不是整个数组。我的意思是，如果改变引用指向的数组，将会受到 volatile 的保护，但是如果多个线程同时改变数组的元素，volatile 标示符就不能起到之前的保护作用了。

**2）volatile 能使得一个非原子操作变成原子操作吗？**

一个典型的例子是在类中有一个 long 类型的成员变量。如果你知道该成员变量会被多个线程访问，如计数器、价格等，你最好是将其设置为 volatile。为什么？因为 Java 中读取 long 类型变量不是原子的，需要分成两步，如果一个线程正在修改该 long 变量的值，另一个线程可能只能看到该值的一半（前 32 位）。但是对一个 volatile 型的 long 或 double 变量的读写是原子。

**3）volatile 修饰符的有过什么实践？**

一种实践是用 volatile 修饰 long 和 double 变量，使其能按原子类型来读写。double 和 long 都是64位宽，因此对这两种类型的读是分为两部分的，第一次读取第一个 32 位，然后再读剩下的 32 位，这个过程不是原子的，但 Java 中 volatile 型的 long 或 double 变量的读写是原子的。volatile 修复符的另一个作用是提供内存屏障（memory barrier），例如在分布式框架中的应用。简单的说，就是当你写一个 volatile 变量之前，Java 内存模型会插入一个写屏障（write barrier），读一个 volatile 变量之前，会插入一个读屏障（read barrier）。意思就是说，在你写一个 volatile 域时，能保证任何线程都能看到你写的值，同时，在写之前，也能保证任何数值的更新对所有线程是可见的，因为内存屏障会将其他所有写的值更新到缓存。

**4）volatile 类型变量提供什么保证？**

volatile 变量提供顺序和可见性保证，例如，JVM 或者 JIT为了获得更好的性能会对语句重排序，但是 volatile 类型变量即使在没有同步块的情况下赋值也不会与其他语句重排序。 volatile 提供 happens-before 的保证，确保一个线程的修改能对其他线程是可见的。某些情况下，volatile 还能提供原子性，如读 64 位数据类型，像 long 和 double 都不是原子的，但 volatile 类型的 double 和 long 就是原子的。

**5) 10 个线程和 2 个线程的同步代码，哪个更容易写？**

从写代码的角度来说，两者的复杂度是相同的，因为同步代码与线程数量是相互独立的。但是同步策略的选择依赖于线程的数量，因为越多的线程意味着更大的竞争，所以你需要利用同步技术，如锁分离，这要求更复杂的代码和专业知识。

**6）你是如何调用 wait（）方法的？使用 if 块还是循环？为什么？**

wait() 方法应该在循环调用，因为当线程获取到 CPU 开始执行的时候，其他条件可能还没有满足，所以在处理前，循环检测条件是否满足会更好。下面是一段标准的使用 wait 和 notify 方法的代码：

```
// The standard idiom for using the wait method
synchronized (obj) {
while (condition does not hold)
obj.wait(); // (Releases lock, and reacquires on wakeup)
... // Perform action appropriate to condition
}

```

参见 Effective Java 第 69 条，获取更多关于为什么应该在循环中来调用 wait 方法的内容。

**7）什么是多线程环境下的伪共享（false sharing）？**

伪共享是多线程系统（每个处理器有自己的局部缓存）中一个众所周知的性能问题。伪共享发生在不同处理器的上的线程对变量的修改依赖于相同的缓存行，如下图所示：

![img](https://pic1.zhimg.com/80/v2-a9557dcd20f2d034d8f2a28c643e7974_hd.jpg)

伪共享问题很难被发现，因为线程可能访问完全不同的全局变量，内存中却碰巧在很相近的位置上。如其他诸多的并发问题，避免伪共享的最基本方式是仔细审查代码，根据缓存行来调整你的数据结构。

## 有经验程序员的 Java 面试题

12）用 wait-notify 写一段代码来解决生产者-消费者问题？

请参考答案中的示例代码。只要记住在同步块中调用 wait() 和 notify()方法，如果阻塞，通过循环来测试等待条件。

```
static class Producer implements Runnable{
        private Clerk clerk;
        public Producer(Clerk clerk)
        {
            this.clerk=clerk;
        }
        @Override
        public void run() {
            // TODO Auto-generated method stub
            System.out.println("生产者开始生产产品");
            while(true)
            {
                try {
                  Thread.sleep(1000);
                } catch (Exception e) {
                    // TODO: handle exception
                }
                clerk.addProduct();
            }

        }
    }

static class Consumer implements Runnable{
        private Clerk clerk;
        public Consumer(Clerk clerk)
        {
            this.clerk=clerk;
        }
        @Override
        public void run() {
            // TODO Auto-generated method stub
            System.out.println("消费者开始消费产品");
            while(true)
            {
                try {
                  Thread.sleep(1000);
                } catch (Exception e) {
                    // TODO: handle exception
                }
                clerk.getProduct();
            }

        }
```

```
 public static void main(String[] args)   
    {  
        Clerk clerk = new Clerk();  
        Thread producerThread = new Thread(new Producer(clerk));  
        Thread consumerThread = new Thread(new Consumer(clerk));  

        producerThread.start();  
        consumerThread.start();  
    }  
    static class Clerk {
        private static final int MAX_PRODUCT = 20;
        private static final int MIN_PRODUCT = 0;

        private int PRODUCT = 0;

        public synchronized void addProduct() {
            if (this.PRODUCT >= MAX_PRODUCT) {
                try {
                    wait();
                    System.out.println("产品已满,请稍候再生产");
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                return;
            }
            this.PRODUCT++;
            System.out.println("生产者生产了第"+this.PRODUCT+"个产品");
            notifyAll();
        }

        public synchronized void getProduct() {
            if(this.PRODUCT <= MIN_PRODUCT) {
                try {
                    wait();
                    System.out.println("产品处于缺货状态");
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                return;
            }
            System.out.println("消费者消费了第" + this.PRODUCT + "个产品");  
            this.PRODUCT--;
            notifyAll();
        }
    }
```

使用 BlockingQueue 实现生产者消费者问题**

```
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
}

public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
        Producer producer = new Producer();
        producer.start();
    }
    for (int i = 0; i < 5; i++) {
        Consumer consumer = new Consumer();
        consumer.start();
    }
    for (int i = 0; i < 3; i++) {
        Producer producer = new Producer();
        producer.start();
    }
}
```

**13) 用 Java 写一个线程安全的单例模式（Singleton）？**

当我们说线程安全时，意思是即使初始化是在多线程环境中，仍然能保证单个实例。Java 中，使用枚举作为单例类是最简单的方式来创建线程安全单例模式的方式。

不使用同步锁

```
public class Singleton {
    private static Singleton sin=new Singleton();    ///直接初始化一个实例对象
    private Singleton(){    ///private类型的构造函数，保证其他类对象不能直接new一个该对象的实例
    }
    public static Singleton getSin(){    ///该类唯一的一个public方法    
        return sin;
    }
}
```

使用同步锁

```
public class Singleton {
    private static Singleton sin=new Singleton();    ///直接初始化一个实例对象
    private Singleton(){    ///private类型的构造函数，保证其他类对象不能直接new一个该对象的实例
    }
    public static Singleton getSin(){    ///该类唯一的一个public方法    
        return sin;
    }
}
```

使用双重同步锁

```
public class Singleton {  
     private static Singleton instance;  
     private Singleton (){
     }   
     public static Singleton getInstance(){    //对获取实例的方法进行同步
       if (instance == null){
           synchronized(Singleton.class){
               if (instance == null)
                   instance = new Singleton(); 
           }
       }
       return instance;
     }
 }
```

**14）Java 中 sleep 方法和 wait 方法的区别？**

虽然两者都是用来暂停当前运行的线程，但是 sleep() 实际上只是短暂停顿，因为它不会释放锁，而 wait() 意味着条件等待，这就是为什么该方法要释放锁，因为只有这样，其他等待的线程才能在满足条件时获取到该锁。

**15）什么是不可变对象（immutable object）？Java 中怎么创建一个不可变对象？**

不可变对象指对象一旦被创建，状态就不能再改变。任何修改都会创建一个新的对象，如 String、Integer及其它包装类。

**16）我们能创建一个包含可变对象的不可变对象吗？**

是的，我们是可以创建一个包含可变对象的不可变对象的，你只需要谨慎一点，不要共享可变对象的引用就可以了，如果需要变化时，就返回原对象的一个拷贝。最常见的例子就是对象中包含一个日期对象的引用。

## **数据类型和 Java 基础面试问题**

**17）Java 中应该使用什么数据类型来代表价格？**

如果不是特别关心内存和性能的话，使用BigDecimal，否则使用预定义精度的 double 类型。

**18）怎么将 byte 转换为 String？**

可以使用 String 接收 byte[] 参数的构造器来进行转换，需要注意的点是要使用的正确的编码，否则会使用平台默认编码，这个编码可能跟原来的编码相同，也可能不同。

**19）Java 中怎样将 bytes 转换为 long 类型？**

String接收bytes的构造器转成String，再Long.parseLong

**20）我们能将 int 强制转换为 byte 类型的变量吗？如果该值大于 byte 类型的范围，将会出现什么现象？**

是的，我们可以做强制转换，但是 Java 中 int 是 32 位的，而 byte 是 8 位的，所以，如果强制转化是，int 类型的高 24 位将会被丢弃，byte 类型的范围是从 -128 到 127。

**21）存在两个类，B 继承 A，C 继承 B，我们能将 B 转换为 C 么？如 C = (C) B；**

可以，向下转型。但是不建议使用，容易出现类型转型异常.

**22）哪个类包含 clone 方法？是 Cloneable 还是 Object？**

java.lang.Cloneable 是一个标示性接口，不包含任何方法，clone 方法在 object 类中定义。并且需要知道 clone() 方法是一个本地方法，这意味着它是由 c 或 c++ 或 其他本地语言实现的。

**23）Java 中 ++ 操作符是线程安全的吗？**

不是线程安全的操作。它涉及到多个指令，如读取变量值，增加，然后存储回内存，这个过程可能会出现多个线程交差。

**24）a = a + b 与 a += b 的区别**

+= 隐式的将加操作的结果类型强制转换为持有结果的类型。如果两这个整型相加，如 byte、short 或者 int，首先会将它们提升到 int 类型，然后在执行加法操作。

```
byte a = 127;
byte b = 127;
b = a + b; // error : cannot convert from int to byte
b += a; // ok

```

（因为 a+b 操作会将 a、b 提升为 int 类型，所以将 int 类型赋值给 byte 就会编译出错）

**25）我能在不进行强制转换的情况下将一个 double 值赋值给 long 类型的变量吗？**

不行，你不能在没有强制类型转换的前提下将一个 double 值赋值给 long 类型的变量，因为 double 类型的范围比 long 类型更广，所以必须要进行强制转换。

**26）3\*0.1 == 0.3 将会返回什么？true 还是 false？**

false，因为有些浮点数不能完全精确的表示出来。

**27）int 和 Integer 哪个会占用更多的内存？**

Integer 对象会占用更多的内存。Integer 是一个对象，需要存储对象的元数据。但是 int 是一个原始类型的数据，所以占用的空间更少。

**28）为什么 Java 中的 String 是不可变的（Immutable）？**

Java 中的 String 不可变是因为 Java 的设计者认为字符串使用非常频繁，将字符串设置为不可变可以允许多个客户端之间共享相同的字符串。

**29）我们能在 Switch 中使用 String 吗？**

从 Java 7 开始，我们可以在 switch case 中使用字符串，但这仅仅是一个语法糖。内部实现在 switch 中使用字符串的 hash code。

**30）Java 中的构造器链是什么？**

当你从一个构造器中调用另一个构造器，就是Java 中的构造器链。这种情况只在重载了类的构造器的时候才会出现。

### **JVM 底层 与 GC（Garbage Collection） 的面试问题**

**31）64 位 JVM 中，int 的长度是多数？**

Java 中，int 类型变量的长度是一个固定值，与平台无关，都是 32 位。意思就是说，在 32 位 和 64 位 的Java 虚拟机中，int 类型的长度是相同的。

**32）Serial 与 Parallel GC之间的不同之处？**

Serial 与 Parallel 在GC执行的时候都会引起 stop-the-world。它们之间主要不同 serial 收集器是默认的复制收集器，执行 GC 的时候只有一个线程，而 parallel 收集器使用多个 GC 线程来执行。

**33）32 位和 64 位的 JVM，int 类型变量的长度是多数？**

32 位和 64 位的 JVM 中，int 类型变量的长度是相同的，都是 32 位或者 4 个字节。

**34）Java 中 WeakReference 与 SoftReference的区别？**

Java中一共有四种类型的引用。StrongReference、 SoftReference、 WeakReference 以及 PhantomReference。

StrongReference 是 Java 的默认引用实现, 它会尽可能长时间的存活于 JVM 内，当没有任何对象指向它时将会被GC回收

WeakReference，顾名思义, 是一个弱引用, 当所引用的对象在 JVM 内不再有强引用时, 将被GC回收

虽然 WeakReference 与 SoftReference 都有利于提高 GC 和 内存的效率，但是 WeakReference ，一旦失去最后一个强引用，就会被 GC 回收，而 SoftReference 会尽可能长的保留引用直到 JVM 内存不足时才会被回收(虚拟机保证), 这一特性使得 SoftReference 非常适合缓存应用

**35）WeakHashMap 是怎么工作的？**

WeakHashMap 的工作与正常的 HashMap 类似，但是使用弱引用作为 key，意思就是当 key 对象没有任何引用时，key/value 将会被回收。

**36）JVM 选项 -XX:+UseCompressedOops 有什么作用？为什么要使用？**

当你将你的应用从 32 位的 JVM 迁移到 64 位的 JVM 时，由于对象的指针从 32 位增加到了 64 位，因此堆内存会突然增加，差不多要翻倍。这也会对 CPU 缓存（容量比内存小很多）的数据产生不利的影响。因为，迁移到 64 位的 JVM 主要动机在于可以指定最大堆大小，通过压缩 OOP 可以节省一定的内存。通过 -XX:+UseCompressedOops 选项，JVM 会使用 32 位的 OOP，而不是 64 位的 OOP。

**37）怎样通过 Java 程序来判断 JVM 是 32 位 还是 64 位？**

你可以检查某些系统属性如 sun.arch.data.model 或 os.arch 来获取该信息。

**38）32 位 JVM 和 64 位 JVM 的最大堆内存分别是多数？**

理论上说上 32 位的 JVM 堆内存可以到达 2^32，即 4GB，但实际上会比这个小很多。不同操作系统之间不同，如 Windows 系统大约 1.5 GB，Solaris 大约 3GB。64 位 JVM允许指定最大的堆内存，理论上可以达到 2^64，这是一个非常大的数字，实际上你可以指定堆内存大小到 100GB。甚至有的 JVM，如 Azul，堆内存到 1000G 都是可能的。

**39）JRE、JDK、JVM 及 JIT 之间有什么不同？**

JRE 代表 Java 运行时（Java run-time），是运行 Java 应用所必须的。JDK 代表 Java 开发工具（Java development kit），是 Java 程序的开发工具，如 Java 编译器，它也包含 JRE。JVM 代表 Java 虚拟机（Java virtual machine），它的责任是运行 Java 应用。JIT 代表即时编译（Just In Time compilation），当代码执行的次数超过一定的阈值时，会将 Java 字节码转换为本地代码，如，主要的热点代码会被准换为本地代码，这样有利大幅度提高 Java 应用的性能。

![img](https://pic4.zhimg.com/80/v2-dd8309180a824d7c2176dce812b2d28f_hd.jpg)

### **3 年工作经验的 Java 面试题**

**40）解释 Java 堆空间及 GC？**

当通过 Java 命令启动 Java 进程的时候，会为它分配内存。内存的一部分用于创建堆空间，当程序中创建对象的时候，就从对空间中分配内存。GC 是 JVM 内部的一个进程，回收无效对象的内存用于将来的分配。

### **JVM 底层面试题及答案**

**41）你能保证 GC 执行吗？**

不能，虽然你可以调用 System.gc() 或者 Runtime.getRuntime().gc()，但是没有办法保证 GC 的执行。

**42）怎么获取 Java 程序使用的内存？堆使用的百分比？**

可以通过 java.lang.Runtime 类中与内存相关方法来获取剩余的内存，总内存及最大堆内存。通过这些方法你也可以获取到堆使用的百分比及堆内存的剩余空间。Runtime.freeMemory() 方法返回剩余空间的字节数，Runtime.totalMemory() 方法总内存的字节数，Runtime.maxMemory() 返回最大内存的字节数。

**43）Java 中堆和栈有什么区别？**

JVM 中堆和栈属于不同的内存区域，使用目的也不同。栈常用于保存方法帧和局部变量，而对象总是在堆上分配。栈通常都比堆小，也不会在多个线程之间共享，而堆被整个 JVM 的所有线程共享。 Difference between stack and heap memory in Java

### **关于内存的的面试问题和答案**

**Java 基本概念面试题**

**44）“a==b”和”a.equals(b)”有什么区别？**

如果 a 和 b 都是对象，则 a==b 是比较两个对象的引用，只有当 a 和 b 指向的是堆中的同一个对象才会返回 true，而 a.equals(b) 是进行逻辑比较，所以通常需要重写该方法来提供逻辑一致性的比较。例如，String 类重写 equals() 方法，所以可以用于两个不同对象，但是包含的字母相同的比较。

**45）a.hashCode() 有什么用？与 a.equals(b) 有什么关系？**

hashCode() 方法是相应对象整型的 hash 值。它常用于基于 hash 的集合类，如 Hashtable、HashMap、LinkedHashMap等等。它与 equals() 方法关系特别紧密。根据 Java 规范，两个使用 equal() 方法来判断相等的对象，必须具有相同的 hash code。

**46）final、finalize 和 finally 的不同之处？**

final 是一个修饰符，可以修饰变量、方法和类。如果 final 修饰变量，意味着该变量的值在初始化后不能被改变。Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的，但是什么时候调用 finalize 没有保证。finally 是一个关键字，与 try 和 catch 一起用于异常的处理。finally 块一定会被执行，无论在 try 块中是否有发生异常。

**47）Java 中的编译期常量是什么？使用它又什么风险？**

公共静态不可变（public static final ）变量也就是我们所说的编译期常量，这里的 public 可选的。实际上这些变量在编译时会被替换掉，因为编译器知道这些变量的值，并且知道这些变量在运行时不能改变。这种方式存在的一个问题是你使用了一个内部的或第三方库中的公有编译时常量，但是这个值后面被其他人改变了，但是你的客户端仍然在使用老的值，甚至你已经部署了一个新的jar。为了避免这种情况，当你在更新依赖 JAR 文件时，确保重新编译你的程序。

### **Java 集合框架的面试题**

这部分也包含数据结构、算法及数组的面试问题

**48) List、Set、Map 和 Queue 之间的区别(答案)**

List 是一个有序集合，允许元素重复。它的某些实现可以提供基于下标值的常量访问时间，但是这不是 List 接口保证的。Set 是一个无序集合。

**49）poll() 方法和 remove() 方法的区别？**

poll() 和 remove() 都是从队列中取出一个元素，但是 poll() 在获取元素失败的时候会返回空，但是 remove() 失败的时候会抛出异常。

**50）Java 中 LinkedHashMap 和 PriorityQueue 的区别是什么？(答案)**

PriorityQueue 保证最高或者最低优先级的的元素总是在队列头部，但是 LinkedHashMap 维持的顺序是元素插入的顺序。当遍历一个 PriorityQueue 时，没有任何顺序保证，但是 LinkedHashMap 课保证遍历顺序是元素插入的顺序。

**51）ArrayList 与 LinkedList 的区别？(答案)**

最明显的区别是 ArrrayList 底层的数据结构是数组，支持随机访问，而 LinkedList 的底层数据结构书链表，不支持随机访问。使用下标访问一个元素，ArrayList 的时间复杂度是 O(1)，而 LinkedList 是 O(n)。更多细节的讨论参见答案。

**52）用哪两种方式来实现集合的排序？(答案)**

你可以使用有序集合，如 TreeSet 或 TreeMap，你也可以使用有顺序的的集合，如 list，然后通过 Collections.sort() 来排序。

**53）Java 中怎么打印数组？(答案)**

你可以使用 Arrays.toString() 和 Arrays.deepToString() 方法来打印数组。由于数组没有实现 toString() 方法，所以如果将数组传递给 System.out.println() 方法，将无法打印出数组的内容，但是 Arrays.toString() 可以打印每个元素。

**54）Java 中的 LinkedList 是单向链表还是双向链表？(答案)**

是双向链表，你可以检查 JDK 的源码。在 Eclipse，你可以使用快捷键 Ctrl + T，直接在编辑器中打开该类。

**55）Java 中的 TreeMap 是采用什么树实现的？(答案)**

Java 中的 TreeMap 是使用红黑树实现的。

**56) Hashtable 与 HashMap 有什么不同之处？(答案)**

这两个类有许多不同的地方，下面列出了一部分：

 a) Hashtable 是 JDK 1 遗留下来的类，而 HashMap 是后来增加的。

 b）Hashtable 是同步的，比较慢，但 HashMap 没有同步策略，所以会更快。 

c）Hashtable 不允许有个空的 key，但是 HashMap 允许出现一个 null key。 

**57）Java 中的 HashSet，内部是如何工作的？(答案)**

HashSet 的内部采用 HashMap来实现。由于 Map 需要 key 和 value，所以所有 key 的都有一个默认 value。类似于 HashMap，HashSet 不允许重复的 key，只允许有一个null key，意思就是 HashSet 中只允许存储一个 null 对象。

**58）写一段代码在遍历 ArrayList 时移除一个元素？(答案)**

该问题的关键在于面试者使用的是 ArrayList 的 remove() 还是 Iterator 的 remove()方法。这有一段示例代码，是使用正确的方式来实现在遍历的过程中移除元素，而不会出现 ConcurrentModificationException 异常的示例代码。

**59）我们能自己写一个容器类，然后使用 foreach 循环吗？**

可以，你可以写一个自己的容器类。如果你想使用 Java 中增强的循环来遍历，你只需要实现 Iterable 接口。如果你实现 Collection 接口，默认就具有该属性。

**60）ArrayList 和 HashMap 的默认大小是多数？(答案)**

在 Java 7 中，ArrayList 的默认大小是 10 个元素，HashMap 的默认大小是16个元素（必须是2的幂）。这就是 Java 7 中 ArrayList 和 HashMap 类的代码片段：

```
// from ArrayList.java JDK 1.7
private static final int DEFAULT_CAPACITY = 10;

//from HashMap.java JDK 7
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

```

**61）有没有可能两个不相等的对象有相同的 hashcode？**

有可能，两个不相等的对象可能会有相同的 hashcode 值，这就是为什么在 hashmap 中会有冲突。相等 hashcode 值的规定只是说如果两个对象相等，必须有相同的hashcode 值，但是没有关于不相等对象的任何规定。

**62）两个相同的对象会有不同的的 hash code 吗？**

不能，根据 hash code 的规定，这是不可能的。

**63）我们可以在 hashcode() 中使用随机数字吗？(答案)**

不行，因为对象的 hashcode 值必须是相同的。参见答案获取更多关于 Java 中重写 hashCode() 方法的知识。

**64）Java 中，Comparator 与 Comparable 有什么不同？(答案)**

Comparable 接口用于定义对象的自然顺序，需要修改Class源代码；而 comparator 通常用于定义用户定制的顺序。Comparable 总是只有一个，但是可以有多个 comparator 来定义对象的顺序。

**65）为什么在重写 equals 方法的时候需要重写 hashCode 方法？(答案)**

因为有强制的规范指定需要同时重写 hashcode 与 equal 是方法，许多容器类，如 HashMap、HashSet 都依赖于 hashcode 与 equals 的规定。

### **Java IO 和 NIO 的面试题**

IO 是 Java 面试中一个非常重要的点。你应该很好掌握 Java IO，NIO，NIO2 以及与操作系统，磁盘 IO 相关的基础知识。下面是 Java IO 中经常问的问题。

**66）在我 Java 程序中，我有三个 socket，我需要多少个线程来处理？**

**67）Java 中怎么创建 ByteBuffer？**

**68）Java 中，怎么读写 ByteBuffer ？**

**69）Java 采用的是大端还是小端？**

**70）ByteBuffer 中的字节序是什么？**

**71）Java 中，直接缓冲区与非直接缓冲器有什么区别？(答案)**

**72）Java 中的内存映射缓存区是什么？(answer答案)**

**73）socket 选项 TCP NO DELAY 是指什么？**

**74）TCP 协议与 UDP 协议有什么区别？(answer答案)**

**75）Java 中，ByteBuffer 与 StringBuffer有什么区别？(答案)**

### **Java 最佳实践的面试问题**

包含 Java 中各个部分的最佳实践，如集合，字符串，IO，多线程，错误和异常处理，设计模式等等。

**76）Java 中，编写多线程程序的时候你会遵循哪些最佳实践？(答案)**

这是我在写Java 并发程序的时候遵循的一些最佳实践：

 a）给线程命名，这样可以帮助调试。 

b）最小化同步的范围，而不是将整个方法同步，只对关键部分做同步。 

c）如果可以，更偏向于使用 volatile 而不是 synchronized。

 d）使用更高层次的并发工具，而不是使用 wait() 和 notify() 来实现线程间通信，如 BlockingQueue，CountDownLatch 及 Semeaphore。

 e）优先使用并发集合，而不是对集合进行同步。并发集合提供更好的可扩展性。

**77）说出几点 Java 中使用 Collections 的最佳实践(答案)**

这是我在使用 Java 中 Collections类的一些最佳实践：

 a）使用正确的集合类，例如，如果不需要同步列表，使用 ArrayList 而不是 Vector。

 b）优先使用并发集合，而不是对集合进行同步。并发集合提供更好的可扩展性。

 c）使用接口代表和访问集合，如使用List存储 ArrayList，使用 Map 存储 HashMap 等等。

 d）使用迭代器来循环集合。

 e）使用集合的时候使用泛型。

**78）说出至少 5 点在 Java 中使用线程的最佳实践。(答案)**

这个问题与之前的问题类似，你可以使用上面的答案。对线程来说，你应该： a）对线程命名 b）将线程和任务分离，使用线程池执行器来执行 Runnable 或 Callable。 c）使用线程池

**79）说出 5 条 IO 的最佳实践(答案)**

IO 对 Java 应用的性能非常重要。理想情况下，你不应该在你应用的关键路径上避免 IO 操作。下面是一些你应该遵循的 Java IO 最佳实践： a）使用有缓冲区的 IO 类，而不要单独读取字节或字符。 b）使用 NIO 和 NIO2 c）在 finally 块中关闭流，或者使用 try-with-resource（Java7） 语句。 d）使用内存映射文件获取更快的 IO。

**80）列出 5 个应该遵循的 JDBC 最佳实践(答案)**

有很多的最佳实践，你可以根据你的喜好来例举。下面是一些更通用的原则： a）使用批量的操作来插入和更新数据 b）使用 PreparedStatement 来避免 SQL 异常，并提高性能。 c）使用数据库连接池 d）通过列名来获取结果集，不要使用列的下标来获取。

**81）说出几条 Java 中方法重载的最佳实践？(答案)**

下面有几条可以遵循的方法重载的最佳实践来避免造成自动装箱的混乱。 a）不要重载这样的方法：一个方法接收 int 参数，而另个方法接收 Integer 参数。 b）不要重载参数数量一致，而只是参数顺序不同的方法。 c）如果重载的方法参数个数多于 5 个，采用可变参数。

### **Date、Time 及 Calendar 的面试题**

**82）在多线程环境下，SimpleDateFormat 是线程安全的吗？(答案)**

不是，非常不幸，DateFormat 的所有实现，包括 SimpleDateFormat 都不是线程安全的，因此你不应该在多线程序中使用，除非是在对外线程安全的环境中使用，如 将 SimpleDateFormat 限制在 ThreadLocal 中。如果你不这么做，在解析或者格式化日期的时候，可能会获取到一个不正确的结果。因此，从日期、时间处理的所有实践来说，我强力推荐 joda-time 库。

**83）Java 中如何格式化一个日期？如格式化为 ddMMyyyy 的形式？(答案)**

Java 中，可以使用 SimpleDateFormat 类或者 joda-time 库来格式日期。DateFormat 类允许你使用多种流行的格式来格式化日期。参见答案中的示例代码，代码中演示了将日期格式化成不同的格式，如 dd-MM-yyyy 或 ddMMyyyy。

**84）Java 中，怎么在格式化的日期中显示时区？**

pattern中加z yyyy-MM-dd HH:mm:ss.SSS Z

**85）Java 中 java.util.Date 与 java.sql.Date 有什么区别？**

java.sql.Date是针对SQL语句使用的，它只包含日期而没有时间部分,它们都有getTime方法返回毫秒数，自然就可以直接构建。java.util.Date 是 java.sql.Date 的父类，前者是常用的表示时间的类，我们通常格式化或者得到当前时间都是用他，后者之后在读写数据库的时候用他，因为PreparedStament的setDate()的第2参数和ResultSet的getDate()方法的第2个参数都是java.sql.Date。

**86）Java 中，如何计算两个日期之间的差距？**

```
public static int dateDiff(Date d1, Date d2) throws Exception {
long n1 = d1.getTime();
long n2 = d2.getTime();
long diff = Math.abs(n1 - n2);

diff /= 3600 * 1000 * 24;
return diff;
}

```

**87）Java 中，如何将字符串 YYYYMMDD 转换为日期？**

SimpleDateFormat的parse方法

### **单元测试 JUnit 面试题**

**89）如何测试静态方法？(答案)**

可以使用 PowerMock 库来测试静态方法。

**90）怎么利用 JUnit 来测试一个方法的异常？(答案)**

**91）你使用过哪个单元测试库来测试你的 Java 程序？(答案)**

**92）@Before 和 @BeforeClass 有什么区别？(答案)**

### **编程和代码相关的面试题**

**93）怎么检查一个字符串只包含数字？(解决方案)**

**94）Java 中如何利用泛型写一个 LRU 缓存？(答案)**

**95）写一段 Java 程序将 byte 转换为 long？(答案)**

**95）在不使用 StringBuffer 的前提下，怎么反转一个字符串？(解决方案)**

**97）Java 中，怎么获取一个文件中单词出现的最高频率？(解决方案)**

**98）如何检查出两个给定的字符串是反序的？(解决方案)**

**99）Java 中，怎么打印出一个字符串的所有排列？(解决方案)**

**100）Java 中，怎样才能打印出数组中的重复元素？(解决方案)**

**101）Java 中如何将字符串转换为整数？(解决方案)**

**102）在没有使用临时变量的情况如何交换两个整数变量的值？(解决方案)**

## **关于 OOP 和设计模式的面试题**

这部分包含 Java 面试过程中关于 SOLID 的设计原则，OOP 基础，如类，对象，接口，继承，多态，封装，抽象以及更高级的一些概念，如组合、聚合及关联。也包含了 GOF 设计模式的问题。

**103）接口是什么？为什么要使用接口而不是直接使用具体类？**

接口用于定义 API。它定义了类必须得遵循的规则。同时，它提供了一种抽象，因为客户端只使用接口，这样可以有多重实现，如 List 接口，你可以使用可随机访问的 ArrayList，也可以使用方便插入和删除的 LinkedList。接口中不允许写代码，以此来保证抽象，但是 Java 8 中你可以在接口声明静态的默认方法，这种方法是具体的。

**104）Java 中，抽象类与接口之间有什么不同？(答案)**

Java 中，抽象类和接口有很多不同之处，但是最重要的一个是 Java 中限制一个类只能继承一个类，但是可以实现多个接口。抽象类可以很好的定义一个家族类的默认行为，而接口能更好的定义类型，有助于后面实现多态机制。关于这个问题的讨论请查看答案。

**105）除了单例模式，你在生产环境中还用过什么设计模式？**

这需要根据你的经验来回答。一般情况下，你可以说依赖注入，工厂模式，装饰模式或者观察者模式，随意选择你使用过的一种即可。不过你要准备回答接下的基于你选择的模式的问题。

**106）你能解释一下里氏替换原则吗?(答案)**

**107) 什么情况下会违反迪米特法则？为什么会有这个问题？(答案)**

迪米特法则建议“只和朋友说话，不要陌生人说话”，以此来减少类之间的耦合。

**108）适配器模式是什么？什么时候使用？**

适配器模式提供对接口的转换。如果你的客户端使用某些接口，但是你有另外一些接口，你就可以写一个适配去来连接这些接口。

**109）什么是“依赖注入”和“控制反转”？为什么有人使用？(答案)**

**110）抽象类是什么？它与接口有什么区别？你为什么要使用过抽象类？(答案)**

**111）构造器注入和 setter 依赖注入，那种方式更好？(答案)**

每种方式都有它的缺点和优点。构造器注入保证所有的注入都被初始化，但是 setter 注入提供更好的灵活性来设置可选依赖。如果使用 XML 来描述依赖，Setter 注入的可读写会更强。经验法则是强制依赖使用构造器注入，可选依赖使用 setter 注入。

**112）依赖注入和工厂模式之间有什么不同？(答案)**

虽然两种模式都是将对象的创建从应用的逻辑中分离，但是依赖注入比工程模式更清晰。通过依赖注入，你的类就是 POJO，它只知道依赖而不关心它们怎么获取。使用工厂模式，你的类需要通过工厂来获取依赖。因此，使用 DI 会比使用工厂模式更容易测试。关于这个话题的更详细讨论请参见答案。

**113）适配器模式和装饰器模式有什么区别？(答案)**

虽然适配器模式和装饰器模式的结构类似，但是每种模式的出现意图不同。适配器模式被用于桥接两个接口，而装饰模式的目的是在不修改类的情况下给类增加新的功能。

**114）适配器模式和代理模式之前有什么不同？(答案)**

这个问题与前面的类似，适配器模式和代理模式的区别在于他们的意图不同。由于适配器模式和代理模式都是封装真正执行动作的类，因此结构是一致的，但是适配器模式用于接口之间的转换，而代理模式则是增加一个额外的中间层，以便支持分配、控制或智能访问。

**115）什么是模板方法模式？(答案)**

模板方法提供算法的框架，你可以自己去配置或定义步骤。例如，你可以将排序算法看做是一个模板。它定义了排序的步骤，但是具体的比较，可以使用 Comparable 或者其语言中类似东西，具体策略由你去配置。列出算法概要的方法就是众所周知的模板方法。

**116）什么时候使用访问者模式？(答案)**

访问者模式用于解决在类的继承层次上增加操作，但是不直接与之关联。这种模式采用双派发的形式来增加中间层。

**117）什么时候使用组合模式？(答案)**

组合模式使用树结构来展示部分与整体继承关系。它允许客户端采用统一的形式来对待单个对象和对象容器。当你想要展示对象这种部分与整体的继承关系时采用组合模式。

**118）继承和组合之间有什么不同？(答案)**

虽然两种都可以实现代码复用，但是组合比继承共灵活，因为组合允许你在运行时选择不同的实现。用组合实现的代码也比继承测试起来更加简单。

**119）描述 Java 中的重载和重写？(答案)**

重载和重写都允许你用相同的名称来实现不同的功能，但是重载是编译时活动，而重写是运行时活动。你可以在同一个类中重载方法，但是只能在子类中重写方法。重写必须要有继承。

**120）Java 中，嵌套公共静态类与顶级类有什么不同？(答案)**

类的内部可以有多个嵌套公共静态类，但是一个 Java 源文件只能有一个顶级公共类，并且顶级公共类的名称与源文件名称必须一致。

**121) OOP 中的 组合、聚合和关联有什么区别？(答案)**

如果两个对象彼此有关系，就说他们是彼此相关联的。组合和聚合是面向对象中的两种形式的关联。组合是一种比聚合更强力的关联。组合中，一个对象是另一个的拥有者，而聚合则是指一个对象使用另一个对象。如果对象 A 是由对象 B 组合的，则 A 不存在的话，B一定不存在，但是如果 A 对象聚合了一个对象 B，则即使 A 不存在了，B 也可以单独存在。

**122）给我一个符合开闭原则的设计模式的例子？(答案)**

开闭原则要求你的代码对扩展开放，对修改关闭。这个意思就是说，如果你想增加一个新的功能，你可以很容易的在不改变已测试过的代码的前提下增加新的代码。有好几个设计模式是基于开闭原则的，如策略模式，如果你需要一个新的策略，只需要实现接口，增加配置，不需要改变核心逻辑。一个正在工作的例子是 Collections.sort() 方法，这就是基于策略模式，遵循开闭原则的，你不需为新的对象修改 sort() 方法，你需要做的仅仅是实现你自己的 Comparator 接口。

**123）抽象工厂模式和原型模式之间的区别？(答案)**

**124）什么时候使用享元模式？(答案)**

享元模式通过共享对象来避免创建太多的对象。为了使用享元模式，你需要确保你的对象是不可变的，这样你才能安全的共享。JDK 中 String 池、Integer 池以及 Long 池都是很好的使用了享元模式的例子。

### **Java 面试中其他各式各样的问题**

这部分包含 Java 中关于 XML 的面试题，JDBC 面试题，正则表达式面试题，Java 错误和异常及序列化面试题

**125）嵌套静态类与顶级类有什么区别？(答案)**

一个公共的顶级类的源文件名称与类名相同，而嵌套静态类没有这个要求。一个嵌套类位于顶级类内部，需要使用顶级类的名称来引用嵌套静态类，如 HashMap.Entry 是一个嵌套静态类，HashMap 是一个顶级类，Entry是一个嵌套静态类。

**126）你能写出一个正则表达式来判断一个字符串是否是一个数字吗？(解决方案)**

一个数字字符串，只能包含数字，如 0 到 9 以及 +、- 开头，通过这个信息，你可以下一个如下的正则表达式来判断给定的字符串是不是数字。

**127）Java 中，受检查异常 和 不受检查异常的区别？(答案)**

受检查异常编译器在编译期间检查。对于这种异常，方法强制处理或者通过 throws 子句声明。其中一种情况是 Exception 的子类但不是 RuntimeException 的子类。非受检查是 RuntimeException 的子类，在编译阶段不受编译器的检查。

![img](https://pic2.zhimg.com/80/v2-5dc2ec13f7165dca5551f19df54803a5_hd.jpg)

**128）Java 中，throw 和 throws 有什么区别？(答案)**

throw 用于抛出 java.lang.Throwable 类的一个实例化对象，意思是说你可以通过关键字 throw 抛出一个 Error 或者 一个Exception，如： throw new IllegalArgumentException(“size must be multiple of 2″) 而throws 的作用是作为方法声明和签名的一部分，方法被抛出相应的异常以便调用者能处理。Java 中，任何未处理的受检查异常强制在 throws 子句中声明。

**129）Java 中，Serializable 与 Externalizable 的区别？(答案)**

Serializable 接口是一个序列化 Java 类的接口，以便于它们可以在网络上传输或者可以将它们的状态保存在磁盘上，是 JVM 内嵌的默认序列化方式，成本高、脆弱而且不安全。Externalizable 允许你控制整个序列化过程，指定特定的二进制格式，增加安全机制。

**130）Java 中，DOM 和 SAX 解析器有什么不同？(答案)**

DOM 解析器将整个 XML 文档加载到内存来创建一棵 DOM 模型树，这样可以更快的查找节点和修改 XML 结构，而 SAX 解析器是一个基于事件的解析器，不会将整个 XML 文档加载到内存。由于这个原因，DOM 比 SAX 更快，也要求更多的内存，不适合于解析大 XML 文件。

**131）说出 JDK 1.7 中的三个新特性？(答案)**

虽然 JDK 1.7 不像 JDK 5 和 8 一样的大版本，但是，还是有很多新的特性，如 try-with-resource 语句，这样你在使用流或者资源的时候，就不需要手动关闭，Java 会自动关闭。Fork-Join 池某种程度上实现 Java 版的 Map-reduce。允许 Switch 中有 String 变量和文本。菱形操作符(<>)用于类型推断，不再需要在变量声明的右边申明泛型，因此可以写出可读写更强、更简洁的代码。另一个值得一提的特性是改善异常处理，如允许在同一个 catch 块中捕获多个异常。

**132）说出 5 个 JDK 1.8 引入的新特性？(答案)**

1、default关键字:引入了新的关键字default，通过使用default修饰方法，可以让我们在接口里面定义具体的方法实现

2、Lambda 表达式:函数式编程，允许把函数本身作为参数传入另一个函数，还允许返回一个函数

3、函数式接口：指仅仅只包含一个抽象方法的接口，提供了一个@FunctionalInterface注解来定义函数式接口

4.方法与构造函数引用：提供了另外一种调用方式::，当你需要使用方法引用时 ，目标引用放在分隔符::前 ，方法的名称放在后面 ，即`ClassName :: methodName` 。

5、Date API更新

**133）Java 中，Maven 和 ANT 有什么区别？(答案)**

虽然两者都是构建工具，都用于创建 Java 应用，但是 Maven 做的事情更多，在基于“约定优于配置”的概念下，提供标准的Java 项目结构，同时能为应用自动管理依赖（应用中所依赖的 JAR 文件），Maven 与 ANT 工具更多的不同之处请参见答案。
