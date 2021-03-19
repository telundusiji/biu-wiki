# Java基础篇

### 1. String属于基础类型吗

不属于，String属于引用类型。Java的基本数据类型有8种：byte-1，short-2，int-4，long-8，float-4，double-8，boolean-1，char-2

### 2. ==和equal区别

首先 == 对于基本类型和引用类型作用效果是不同的

* 在基本类型中 == 是比较的两个值是否相同
* 在引用类型中是比较两个对象的引用是否相同

而equal属于Object类的一个成员方法，由于在Java中Object类是所有类的父类，所以在Java中所有引用类型中都有equal方法，而基本类型(值类型)没有equal方法(不过基本类型的包装类也属于引用类型，所以他们也有equal方法)。

Object类中equal时直接比较两个对象的引用是否相同，所以若在没有重新equal方法的类的对象上使用equal就是比较两个对象的引用是否相同，若在重写了equal方法的类的对象上使用equal，则就按照重写的逻辑进行比较。

### 3. final在Java中的作用

final在Java中可以修饰类，方法，变量。

* final修饰的类不能被继承
* final修饰的方法不能被重写
* final修饰的变量，在声明时必须初始化，初始化后其值不能再被更改

### 4. StringBuffer和StringBuilder区别

StringBuffer和StringBuilder两个类主要功能都是进行字符串拼接操作。两个类在api的使用上基本相同，最大的不同点就是StringBuffer是线程安全的，StringBuilder是非线程安全的。StringBuffer的线程安全实现方式是在其方法上加Synchronized保证同步，其性能上相比StringBuilder会低一些，因此在我们编码使用中，若不需要考虑线程安全问题时，使用StringBuilder更加高效，需要考虑线程安全时使用StingBuffer更加安全。

### 5. String s="s"与 String s=new String("s") 有什么区别

这两种字符串的声明方式的主要的区别就是：内存的分配方式不一样。在Java中String属于一个比较特殊的类，在Jvm中一般都设置有String实例的常量池，String s="s"的方式，java 虚拟机会将“s”这个字符串存储在常量池中，并将常量池中"s"的地址赋给str变量；而 String s=new String("i") 则会在堆内存中分配空间存储“s”，并将堆中这个地址赋值给s。

**拓展回答**

当我们用 == 去比较这两种方式声明的两个变量的时候，由于其引用地址不同就会得到false

### 6. String 类的常用方法都有那些？简单举例几个

String中常有的方法有

* charAt: 返回其内部char数组的指定索引处的字符
* trim: 去除字符串两端空白
* replace: 将字符串中指定片段的字符串进行替换
* replaceAll: 将字符串中匹配的字符串片段进行替换，支持正则表达式匹配
* replaceFirst: 将字符串中匹配的第一个字符串片段进行替换，支持正则表达式匹配
* length: 返回字符串长度
* split: 指定分割符，对字符串分割
* subString: 对字符串进行截取
* toLowerCase: 将字符串英文字符转小写
* toUpperCase: 将字符串英文字符转大写

### 7. 关于抽象类简单说一说。抽象类必须有抽象方法吗？普通类和抽象类有什么区别？接口和抽象类有什么区别？

抽象类是Java中类的一种定义方式，定义抽象类使用abstract关键字。抽象类与一般类的不同在于抽象类不能被直接实例化，并且我们可以在抽象类中添加抽象方法，普通类中不可以添加抽象方法，抽象类中的抽象方法不用写具体实现。抽象类不能使用final修饰，可以被继承，若抽象的类的子类是普通类则这个子类就需要实现抽象父类的抽象方法，当然并不是抽象类中就必须包含抽象方法，在抽象类中抽象方法是可选的。

在Java中接口和抽象类的区别主要有一下几点

* 抽象类与子类是继承关系使用 extends 关键字，接口与实现类是实现关系使用implements关键字
* 抽象类可以自定义构造方法，接口不允许自定义构造方法
* 一个普通类可以实现多个接口，但是只能继承一个抽象类
* 接口种的方法，访问修饰都是public修饰，抽象类中非抽象方法访问修饰无限制，抽象类中抽象方法不可使用private修饰

### 8. Java  IO流分几种

按功能分：输入流和输出流

按类型分：字节流和字符流

字节流使用InputStream和OutputStream两个接口定义输入和输出，字符流使用Reader和Writer两个接口定义输入和输出

字节流和字符流的主要区别是字节流以字节（byte 8 位）为单位输入输出数据，字符流以字符（char 16 位）为单位输入输出数据。

字节流和字符流之间转换可以通过InputStreamReader和OutputStreamWriter实现，InputStreamReader可以将字节流转字符流，OutputStreamWriter将字符流转字节流。

常用字节流：FileInputStream，FileOutputStream，ByteArrayInputStream，ByteArrayOutputStream等

常用字符流：FileReader，FileWriter，BufferedReader，BufferedWriter等

### 9. BIO，NIO 和 AIO有什么区别？

* BIO 是同步阻塞式IO，也称为传统IO，这种IO的特点是模式简单使用方便，但是由于处理过程是同步阻塞式的，数据的读取写入必须阻塞在一个线程内等待其完成，所以并发情况下处理效率很低，可以在适用于连接数目比较小且固定的架构中
* NIO 是同步非阻塞 IO，是传统 IO 的升级，客户端和服务器端通过 Channel（通道）通讯，使用一个线程去轮询所有IO操作的状态改变，在相应的状态改变后，再进行后续操作，实现了多路复用。可用在连接数目多且连接比较短（轻操作）的架构

- AIO 是异步非阻塞IO，异步 IO 的操作基于事件和回调机制，它不需要单独启用线程去轮询所有IO操作的状态，而是在相应的状态改变后，由系统来通知回调相应的线程来处理，可用于连接数目多且连接比较长（重操作）的架构

### 10. Java中集合有哪些，简单说一下

Java中常有的集合种类有5种分别是：Map，Set，List，Queue，Stack。

* Map是映射，也就是存储Key-Value类型数据，同一个Map中Key不可以重复。常用的实现有HashMap ConcurrentHashMap（拓展回答：还有HashTable在早期的Java版本中使用的比较多），在Map中k-v对的顺序在一般情况下不是按照放入的顺序进行排序，而是按照key的hash值，不过当你需要Map记录k-v对的放入顺序时，也可以使用LinkedHashMap，LinkedHashMap内部使用双向链表记录了元素放入的先后顺序
* Set是集合，集合中的元素不能重复，在JDK中提供的几个Set常用实现HashSet，TreeSet，其底层的数据结构都是基于Map的，是利用了Map的Key不可能重复的特性。在Set中放入的元素一般情况下也不记录放入顺序，所以需要记录顺序时可以使用LinkedHashSet
* List是列表，列表中的元素是可以重复的，且可以按照放入顺序进行读取，常用的List实现有底层数据结构基于数组的ArrayList，底层基于双向链表的LinkedList等
* Queue是队列，队列是一种先进先出的特殊数据结构，Jdk中提供的队列实现按是否阻塞分为：阻塞队列（BlockQueue）、非阻塞队列（Queue）。按照类型除了单端队列还提供了双端队列（队列两端都可以出队和入队），常有的队列实现有：ArrayBlockingQueue，LinkedBlockingQueue，ConcurrentLinkedQueue，ArrayDeque等
* Stack是栈，栈是一种先进后出的数据结构，JDK中提供了一个栈的简单实现就叫Stack，这个Stack类是继承Vector类的，并没有太多代码，仅保证了栈先进后出的性质

### 11. Collection 和 Collections 有什么区别

- java.util.Collection 是一个集合类的顶级接口。它提供了对集合对象进行基本操作的通用接口方法。Collection接口是为各种具体的集合提供了最大化的统一操作方式，List与Set接口就是直接继承Collection。
- Collections则是集合类的一个工具类，其中提供了一系列静态方法，用于对集合进行一些常有操作，比如元素排序、查找等操作。

### 12. List、Set、Map 之间的区别是什么？

* Map是映射，存储k-v类型数据，同一个Map中Key不可以重复，Set是集合，同一个Set中存储的元素不可以重复，List是列表，List中存储元素可以重复。
* List和Set接口都是继承Collection的，Map接口不继承Collection接口
* JDK中提供的几种Set的实现，其底层数据结构都是使用的Map。例如HastSet基于HashMap，TreeSet基于TreeMap
* List中存储元素记录元素放入顺序，Map和Set一般情况下不记录放入其中的元素的顺序，若需记录可以分别使用LinkedHashMap和LinkedHashSet实现

拓展回答：以上区别点说完，可以在对每个类型，有哪些具体实现，这些实现有什么特性进行讲述（参照第10题）

### 13. Arraylist 、LinkedList 、Vector区别?

相同：

* 首先这三个集合都是实现List接口的标准，所以他们都具备List的基本功能，比如添加元素，删除元素，指定索引查找元素等

不同：

* 这三种实现的底层数据结构有差异，ArrayList和Vector底层是基于数组实现的，LinkedList基层是基于双向链表实现的
* 在底层数据结构的扩容机制上存在差异。首先LinkedList底层基于链表，所有不需要扩容。ArrayList和Vector底层都是基于数组，ArrayList在声明情况不添加元素时，其底层数组的length为0，再向其添加第一个元素时扩容数组长度为10，Vector默认声明时初始化底层数组length为10，ArrayList的数组扩容默认是增加原长度的0.5倍，Vector的数组扩容默认是增加原长度1倍，两者在元素减少时都不会自动缩容，不过都可以使用trimToSize方法来减少底层数组长度。
* 三者应用场景(优劣)也有区别。ArrayList和Vector底层都是基于数组，所以两者相对LinkedList来说在随机访问(根据索引查找元素)上效率高于LinkedList，但是由于数组中元素的增减大部分情况下需要移动数组中其他元素，而LinkedList底层是链表增减元素的效率也就比ArrayList和Vector要高。这三个实现中Vector是线程安全的，它的操作方法都使用了Synchronized修饰，而ArrayList和LinkedList都是线程不安全的，所以在并发场景下需要考虑线程安全时Vector更加合适，不过要使用线程安全的List还有CopyOnWriteArrayList或者使用Collections.synchronizedList生成也是可以的。
* 其他区别，比如在序列化上ArrayList序列化时按照Size进行，对于未使用的容量的空的元素不进行序列化，而Vector则是按照容量的大小进行序列化，会将未使用空元素也进行序列化浪费空间。LinkedList还实现了Queue的接口，默认就支持队列的操作，而ArrayList和Vector并没有

### 14. HashMap、HashTable、ConcurrentHashMap区别？

首先HashMap和HashTable和ConcurrentHashMap都实现了Map的接口，都具备Map的基本特性

不同:

* 三者直接继承不同。HashMap和ConcurrentHashMap继承AbstractMap，HashTable继承自Dictionary
* 三者对null的存储要求不一样。HashMap允许null 的key和value，HashTable和ConcurrentHashMap不允许
* 三者的线程安全问题。HashMap是非线程安全的。ConcurrentHashMap和HashTable是线程安全的，HashTable实现线程安全使用的Synchronized修饰，相对于ConcurrentHashMap的段锁其效率低一些。
* 三者初始容量和扩容方式也存在差异。HashMap和ConcurrentHashMap的默认容量都是16，扩容大小都是2的指数倍，HashTable初始大小是11，扩容大小是old*2+1

拓展回答：

* 三者的HashCode计算方法
* HashTable和ConcurrentHashMap的线程安全设计方式。

### 15. HashSet如何检查元素是否重复？

添加元素到 HashSet  时会先计算元素的 hashcode 来判断元素放入哈希表的索引位置，然后与同位置的其他的对象的 hashcode 值作比较，如果没有相同的 hashcode ， HashSet 会认为对象没有重复出现。如果发现有相同 hashcode 的元素，再使用 equals() ⽅法来检查 hashcode 相等的两个元素是否相同。如果两者相同， HashSet 则不会将该元素再加入。

在Java中由于HashSet的实现底层是基于HashMap的，所以其实这也是HashMap的检查key重复的方式。

### 16. 并发和并行区别

* 并行是指两个或两个以上事件在同一时刻发生。并发是指两个或两个以上事件在同一时间间隔发生。
* 并行是在不同实体上的多个事件，并行度是描述在多个实体同一时刻可以处理的事件数，并发是在同一实体上的多个事件，并发量是描述单个实体在一段时间内可以处理的事件数。

### 17. 线程和进程区别

进程是程序运行和资源分配的基本单位，一个程序至少有一个进程，一个进程至少有一个线程。进程在执行过程中拥有独立的内存单元，而多个线程共享内存资源，而效率更高。线程是进程的一个实体，是cpu调度和分派的基本单位，是比程序更小的能独立运行的基本单位。同一进程中的多个线程之间可以并发执行。

### 18. Java中创建线程的几种方式

* 继承Thread，重写run方法

```java
publice class NewThread extends Thread{
    public void run(){
        
    }
}

public static void main(String[] args){
    new NewThread().start();
}
```

* 实现Runnable接口

```java
public static void main(String[] agrs){
    new Thread(()->{
        //实现Runnable接口
    }).start();
}
```

* 实现Callable接口，结合FutureTask创建新线程

```java
public static void main(String[] agrs){
    FutureTask<String> stringFutureTask = new FutureTask<>(()->{
        //实现Callable接口
    });
    new Thread(stringFutureTask).start();
}
```

* 使用线程池来创建和管理线程

```java
public static void main(String[] args){
    ExecutorService executorService = Executors.newFixedThreadPool(3);
    executorService.execute(()->{
        //实现Runnable
    });
    executorService.submit(()->{
        //实现Runnable或者Callable
    });
}
```

### 19. Runnable和Callable创建新线程有什么不同

* Runnable接口的run方法返回值是void，在线程中纯粹的去执行run()方法中的代码
* Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来获取异步执行的结果。

### 20. 简单说明线程的几种状态

[https://www.runoob.com/note/34745](https://www.runoob.com/note/34745)

线程的状态有5种：创建、就绪、运行、阻塞、死亡

* 创建状态。在生成线程对象，并没有调用该对象的start方法，这是线程处于创建状态。
* 就绪状态。当调用了线程对象的start方法之后，线程调度程序还没有调度该线程时，该线程就是就绪状态。在线程运行之后，从等待或者睡眠中被唤醒后，但是还未被线程调度程序调度时，也会处于就绪状态。
* 运行状态。线程调度程序调度处于就绪状态的线程执行其代码，此时线程就进入了运行状态，开始运行run函数当中的代码。
* 阻塞状态。线程正在运行的时候，被暂停，就是阻塞状态，通常是为了等待某个事件的发生(比如说等待某项资源就绪)而暂停。sleep(睡眠), wait(等待)等方法都可以导致线程阻塞。
* 死亡状态。如果一个线程的run方法执行结束或者调用stop方法后，该线程就会死亡。对于已经死亡的线程，无法再使用start方法令其进入就绪 　

### 21. 线程的run()和start()有什么区别

start()方法是用来启动一个线程的，run()方法是“线程体”是在一个线程里要具体执行的逻辑。

当我们创建一个线程时，该线程就是创建状态，当调用start()方法时，线程就会进入就绪状态，就绪状态的线程被线程调度程序调度时进入运行状态，线程运行状态时就是执行的该线程的run()方法中的逻辑，run()方法中的逻辑内容执行完毕后线程就结束了进入死亡状态

### 22. sleep()和wait()的区别

方法类型不一样

* sleep方法是Thread类的静态方法
* wait方法是Object类的成员方法

使用方式不一样

* wait一般配合notify或notifyAll使用，需要使用在Synchronized同步方法或者代码块中

* sleep单独使用，没有特殊要求

使用效果不一样

* 使用sleep时，调用sleep方法的线程进入阻塞状态，等到睡眠时间结束后，线程自动进入就绪状态，调用sleep方法的线程不会释放对象锁
* 使用wait时，调用对象的wait方法的线程进入阻塞状态，进入和该对象相关的等待池中，直到等待超时或者其他线程调用了相关对象的notify()和notifyAll()唤醒。

### 23. notify()和notifyAll()区别

当调用了对象的notify或notifyAll时，notify()是随机唤醒一个该对象的wait线程，notifyAll()唤醒所有该对象的wait线程。被唤醒的的线程便会进入该对象的锁池中，锁池中的线程会去竞争该对象锁。

### 24. Java线程池有哪几种

Java中线程池推荐的有4中方式。

* 固定长度线程池 Executors.newFixedThreadPool(int nThreads)

> 创建一个固定长度的线程池。每当提交一个任务就创建一个线程，直到达到线程池的最大数量，这时线程规模将不再变化，当线程发生未预期的错误而结束时，线程池会补充一个新的线程。

* 可缓存的线程池 Executos.newCachedThreadPool()

> 创建一个可缓存的线程池，当需求增加时，可以自动添加新线程，线程池的规模不存在任何限制，如果线程池的规模超过了处理需求，将自动回收空闲线程。

* 单线程线程池 Executors.newSingleThreadExecutor()

> 创建一个单线程的线程池(单线程其实严格讲不应该称为池)，它创建单个工作线程来执行任务，如果这个线程异常结束，会创建一个新的来替代它；它的特点是能确保依照任务在队列中的顺序来串行执行。

*  创建一个固定长度支持调度线程池 Executors.newScheduledThreadPool(int corePoolSize)

> 创建了一个固定长度的线程池，该线程的任务支持以延迟或定时的方式来执行，类似于Timer。

### 25. ThreadPoolExecutor的构造方法的7个参数的含义

ThreadPoolExecutor的构造方法的7个参数分别是：corePoolSize, maximumPoolSize, keepAliveTime, timeUnit, workQueue, threadFactory, rejectedExecutionHandler。

* corePoolSize：核心池的大小。在创建了线程池后，默认情况下，线程池中并没有任何线程，而是等待有任务到来才创建线程去执行任务。除非调用了prestartAllCoreThreads()或者prestartCoreThread()方法，2个方法是预创建线程的意思，即在没有任务到来之前就创建corePoolSize个线程或者一个线程。当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中。
* maximumPoolSize：线程池最大线程数，表示在线程池中最多能创建多少个线程
* keepAliveTime：表示线程没有任务执行时最多保持多久时间会终止。默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用，当线程池中的线程数大于corePoolSize时，如果一个线程空闲的时间达到keepAliveTime，则会终止，直到线程池中的线程数不超过corePoolSize。但是如果调用了allowCoreThreadTimeOut(true)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0
* timeUnit：keepAliveTime的时间单位
* workQueue：一个阻塞队列，用来存储等待执行的任务
* threadFactory：线程工厂，主要用来创建线程。参数类型是ThreadFactory接口，需要传入该类的实现。
* rejectedExecutionHandler：表示当拒绝处理任务时的策略，有以下四种取值

> ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。
>
> ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。
>
> ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
>
> ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

### 26. 线程池的 submit 和 execute 方法的区别

- 两个方法接收入参不一样，execute只能接收Runnable的实现类，submit有不同的重载既可以接收Runnable的实现类也可以接收Callable的实现类
- 方法的返回值不一样，submit有返回值，会返回一个Future，可以通过这个Future获取任务的执行结果和捕获异常信息，而execute没有返回值，若执行的任务发生异常则直接在执行线程中抛出

### 27. Java线程池的几种状态

Java线程池共有五种状态：Running、ShutDown、Stop、Tidying、Terminated

* Running：线程池初始化状态就是RUNNING状态，线程池在RUNNING状态时，能够接收新任务，以及对已添加的任务进行处理
* ShutDown：当调用线程池的shutdown()方法后，线程池进入SHUTDOWN状态，线程池处在SHUTDOWN状态时，不接收新任务，但能处理已添加的任务
* Stop：当调用线程池的shutdownNow()方法后，线程池进入STOP状态，线程池处在STOP状态时，不接收新任务，不处理已添加的任务，并且会中断正在处理的任务
* Tidying：当线程池在SHUTDOWN状态下，workQueue为空且线程池中执行的任务也为空时，就会由 SHUTDOWN 进入TIDYING。当线程池在STOP状态下，线程池中执行的任务为空时，就会由STOP 进入 TIDYING。当线程池变为TIDYING状态时，会执行hook方法terminated()。terminated()在ThreadPoolExecutor类中默认是空实现，用户可以重写terminated()来实现线程池在进入TIDYING后的相应处理
* Terminated：线程池处在TIDYING状态时，执行完terminated()之后，就会由 TIDYING 进入 TERMINATED。线程池处于TERMINATED时，代表线程池已经彻底终止。

### 28. 什么是死锁

死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，在无外力作用，它们都将无法进行下去，此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。死锁是操作系统层面的一个错误，是进程死锁的简称。

在我们Java中常说的死锁指定线程死锁，他与进程死锁类似，线程死锁是多个线程由于资源竞争而造成阻塞，由于线程被⽆限期地阻塞，因此程序不可能正常终⽌。举个例子就是：此时有一个线程A，按照先锁a再获得锁b的的顺序获得锁，而在此同时又有另外一个线程B，按照先锁b再锁a的顺序获得锁，在某一时刻线程A获得了锁a，需要请求锁b，而线程B获得了锁b，需要请求锁a，此时两个线程都持有对方需要的资源同时又都想获取对方持有的资源，这个时候两个线程就死锁了，会无限期等待下去。

### 29. 如何避免死锁

在操作系统中有定义产生死锁的四个必要条件：

* **互斥条件：存在临界资源，该资源任意⼀个时刻只由⼀个线程占⽤**一个资源每次只能被一个进程使用，在一段时间内某资源仅为一个进程所占有，此时若有其他进程请求该资源，则请求进程只能等待
* **请求与保持：⼀个进程因请求资源⽽阻塞时，对已获得的资源保持不放**进程已经持有了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求被阻塞，但对自己已获得的资源保持不放
* **不可抢占（不可剥夺）：进程已获得的资源，在未使用完之前，不可被其他进程抢占，只能在使用完后自己释放**
* **循环等待：若⼲进程之间形成⼀种头尾相接的循环等待资源关系。**即一组等待进程 {P0，P1，…，Pn}，P0 等待的资源为 P1 占有，P1 等待的资源为 P2 占有，……，Pn-1 等待的资源为 Pn 占有，Pn 等待的资源为 P0 占有，形成一个资源等待的环。

那么要避免死锁只要破坏这个四个条件中任意一个就可以避免死锁。

* 破坏互斥，条件允许则可以增减临界资源数量，一般情况下临界资源是不能增加的
* 破坏请求与保持，申请资源时一次性申请
* 破坏不可抢占，线程在申请不到资源时，主动释放当前持有资源
* 破坏循环等待，按序申请资源，对要申请的资源多个线程按照相同的顺序

### 30. 什么是线程安全？体现在那些方面

在拥有共享数据的多条线程并行执行的程序中，线程安全的代码会通过同步机制保证各个线程都可以正常且正确的执行，不会出现数据污染等意外情况。

线程安全主要体现在三个方面：

* 原子性：提供了互斥访问，同一时刻只能有一个线程来对它进行操作
* 可见性：一个线程对主内存的修改可以及时的被其他线程观察到
* 有序性：一个线程观察其他线程中的指令执行顺序，由于指令重排序的存在，该观察结果一般杂乱无序。