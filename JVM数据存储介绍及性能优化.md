####JVM数据存储介绍及性能优化
#####JVM内存模式介绍
Java 虚拟机内存模型是 Java 程序运行的基础。为了能使 Java 应用程序正常运行，JVM 虚拟机将其内存数据分为程序计数器、虚拟机栈、本地方法栈、Java 堆和方法区等部分。
程序计数器 (Program Counter Register)
程序计数器 (Program Counter Register) 是一块很小内存空间，由于 Java 是支持线程的语言，当线程数量超过 CPU 数量时，线程之间根据时间片轮询抢夺 CPU 资源。对于单核 CPU 而言，每一时刻只能有一个线程在运行，而其他线程必须被切换出去。为此，每一个线程都必须用一个独立的程序计数器，用于记录下一条要运行的指令。各个线程之间的计数器互不影响，独立工作，是一块线程独有的内存空间。如果当前线程正在执行一个 Java 方法，则程序计数器记录正在执行的 Java 字节码地址，如果当前线程正在执行一个 Native 方法，则程序计数器为空。
虚拟机栈
虚拟机栈用于存放函数调用堆栈信息。Java 虚拟机栈也是线程私有的内存空间，它和 Java 线程在同一时间创建，它保存方法的局部变量、部分结果，并参与方法的调用和返回。
Java 虚拟机规范允许 Java 栈的大小是动态的或者是固定的。在 Java 虚拟机规范中定义了两种异常与栈空间有关：StackOverflowError 和 OutOfMemoryError。如果线程在计算过程中，请求的栈深度大于最大可用的栈深度，则抛出 StackOverflowError；如果 Java 栈可以动态扩展，而在扩展栈的过程中没有足够的内存空间来支持栈的发展，则抛出 OutOfMemeoryError。可以使用-Xss 参数来设置栈的大小，栈的大小直接决定了函数调用的可达深度。
下面的例子展示了一个递归调用的应用。计数器 count 记录了递归的层次，这个没有出口的递归函数一定会导致栈溢出。程序则在栈溢出时，打印出栈的当前深度。

>递归调用显示栈的最大深度

    public class TestStack{
	    private int count = 0;
	    //没有出口的递归函数
	    public void recursion(){
		    count++;//每次调用深度加1
		    recursion();
	    }

		public void testStack(){
			try{
				recursion();
			}catch(Throwale e){
				System.out.println("deep of stack is "+count);//打印栈溢出的深度
				e.printStackTrace();
			}
		}

		public static void main(String[] args){
			TestStack ts = new TestStack();
			ts.testStack();
		}
		
    }

>上述代码运行结果

    java.lang.StackOverflowError
	at TestStack.recursion(TestStack.java:7)
	at TestStack.recursion(TestStack.java:7)
	at TestStack.recursion(TestStack.java:7)
	at TestStack.recursion(TestStack.java:7)
	at TestStack.recursion(TestStack.java:7)
	at TestStack.recursion(TestStack.java:7)
	at TestStack.recursion(TestStack.java:7)deep of stack is 9013

虚拟机栈在运行时使用一种叫做栈帧的数据结构保存上下文数据。在栈帧中，存放了方法的局部变量表、操作数栈、动态连接方法和返回地址等信息。每一个方法的调用都伴随着栈帧的入栈操作。相应地，方法的返回则表示栈帧的出栈操作。如果方法调用时，方法的参数和局部变量相对较多，那么栈帧中的局部变量表就会比较大，栈帧会膨胀以满足方法调用所需传递的信息。因此，单个方法调用所需的栈空间大小也会比较多。
函数嵌套调用的次数由栈的大小决定。栈越大，函数嵌套调用次数越多。对一个函数而言，它的参数越多，内部局部变量越多，它的栈帧就越大，其嵌套调用次数就会减少。
本地方法栈
本地方法栈和 Java 虚拟机栈的功能很相似，本地方法栈用于存放函数调用堆栈信息。Java 虚拟机栈用于管理 Java 函数的调用，而本地方法栈用于管理本地方法的调用。本地方法并不是用 Java 实现的，而是使用 C 实现的。在 SUN 的 HotSpot 虚拟机中，不区分本地方法栈和虚拟机栈。因此，和虚拟机栈一样，它也会抛出 StackOverflowError 和 OutofMemoryError。
Java 堆
堆用于存放 Java 程序运行时所需的对象等数据。几乎所有的对象和数组都是在堆中分配空间的。Java 堆分为新生代和老生代两个部分，新生代用于存放刚刚产生的对象和年轻的对象，如果对象一直没有被回收，生存得足够长，老年对象就被移入老年代。新生代又可进一步细分为 eden、survivor space0 和 survivor space1。eden 即对象的出生地，大部分对象刚刚建立时都会被存放在这里。survivor 空间是存放其中的对象至少经历了一次垃圾回收，并得以幸存下来的。如果在幸存区的对象到了指定年龄仍未被回收，则有机会进入老年代 (tenured)。下面例子演示了对象在内存中的分配方式。

>进行一次新生代GC

    public class TestHeapGC{
	    public static void main(String[] args){
		    byte[] b1 = new byte[2014*1024/2];
		    byte[] b2 = new byte[2014*1024*8];
		    b2 = null;
		    b2 = new byte[1024 * 1024 * 8];
		    System.gc();
	    }
    }
>执行上面代码的虚拟机配置

    -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=15 -Xms40M -Xmx40M -Xmn20M

>运行输出

    [GC [DefNew: 9031K->661K(18432K), 0.0022784 secs] 9031K->661K(38912K),
	   0.0023178 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 
	 Heap
	 def new generation total 18432K, used 9508K [0x34810000, 0x35c10000, 0x35c10000)
	 eden space 16384K, 54% used [0x34810000, 0x350b3e58, 0x35810000)
	 from space 2048K, 32% used [0x35a10000, 0x35ab5490, 0x35c10000)
	 to space 2048K, 0% used [0x35810000, 0x35810000, 0x35a10000)
	 tenured generation total 20480K, used 0K [0x35c10000, 0x37010000, 0x37010000)
	 the space 20480K, 0% used [0x35c10000, 0x35c10000, 0x35c10200, 0x37010000)
	 compacting perm gen total 12288K, used 374K [0x37010000, 0x37c10000, 0x3b010000)
	 the space 12288K, 3% used [0x37010000, 0x3706db10, 0x3706dc00, 0x37c10000)
	 ro space 10240K, 51% used [0x3b010000, 0x3b543000, 0x3b543000, 0x3ba10000)
	 rw space 12288K, 55% used [0x3ba10000, 0x3c0ae4f8, 0x3c0ae600, 0x3c610000)


上述输出显示 JVM 在进行多次内存分配的过程中，触发了一次新生代 GC。在这次 GC 中，原本分配在 eden 段的变量 b1 被移动到 from 空间段 (s0)。最后分配的 8MB 内存被分配在 eden 新生代。
方法区
方法区用于存放程序的类元数据信息。方法区与堆空间类似，它也是被 JVM 中所有的线程共享的。方法区主要保存的信息是类的元数据。方法区中最为重要的是类的类型信息、常量池、域信息、方法信息。类型信息包括类的完整名称、父类的完整名称、类型修饰符和类型的直接接口类表；常量池包括这个类方法、域等信息所引用的常量信息；域信息包括域名称、域类型和域修饰符；方法信息包括方法名称、返回类型、方法参数、方法修饰符、方法字节码、操作数栈和方法栈帧的局部变量区大小以及异常表。总之，方法区内保持的信息大部分来自于 class 文件，是 Java 应用程序运行必不可少的重要数据。
在 Hot Spot 虚拟机中，方法区也称为永久区，是一块独立于 Java 堆的内存空间。虽然叫做永久区，但是在永久区中的对象同样也可以被 GC 回收的。只是对于 GC 的表现也和 Java 堆空间略有不同。对永久区 GC 的回收，通常主要从两个方面分析：一是 GC 对永久区常量池的回收；二是永久区对类元数据的回收。Hot Spot 虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收。
清单 6 所示代码会生成大量 String 对象，并将其加入常量池中。String.intern() 方法的含义是如果常量池中已经存在当前 String，则返回池中的对象，如果常量池中不存在当前 String 对象，则先将 String 加入常量池，并返回池中的对象引用。因此，不停地将 String 对象加入常量池会导致永久区饱和。如果 GC 不能回收永久区的这些常量数据，那么就会抛出 OutofMemoryError 错误。

>GC收集永久区

    public class permGenGC{
	    public static void main(String[] args){
		    for(int i = 0; i < Integer.MAX_VALUE; i++){
			    String t = String.valueOf(i).intern();
		    }
	    }
    }
>执行上面代码的虚拟机配置

    -XX:PermSize=2M -XX:MaxPermSize=4M -XX:+PrintGCDetails
>输出

    [Full GC [Tenured: 0K->149K(10944K), 0.0177107 secs] 3990K->149K(15872K),
	   [Perm : 4096K->374K(4096K)], 0.0181540 secs] [Times: user=0.02 sys=0.02, real=0.03 secs] 
	[Full GC [Tenured: 149K->149K(10944K), 0.0165517 secs] 3994K->149K(15936K),
	[Perm : 4096K->374K(4096K)], 0.0169260 secs] [Times: user=0.01 sys=0.00, real=0.02 secs] 
	[Full GC [Tenured: 149K->149K(10944K), 0.0166528 secs] 3876K->149K(15936K),
	[Perm : 4096K->374K(4096K)], 0.0170333 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]

每当常量池饱和时,FULL GC 总能顺利回收常量池数据，确保程序稳定持续进行。

#####JVM参数调优实例

由于 Java 字节码是运行在 JVM 虚拟机上的，同样的字节码使用不同的 JVM 虚拟机参数运行，其性能表现可能各不一样。为了能使系统性能最优，就需要选择使用合适的 JVM 参数运行 Java 应用程序。
设置最大堆内存
JVM 内存结构分配对 Java 应用程序的性能有较大的影响。
Java 应用程序可以使用的最大堆可以用-Xmx 参数指定。最大堆指的是新生代和老生代的大小之和的最大值，它是 Java 应用程序的堆上限。清单 9 所示代码是在堆上分配空间直到内存溢出。-Xmx 参数的大小不同，将直接决定程序能够走过几个循环，本例配置为-Xmx5M，设置最大堆上限为 5MB。

>Java堆分配空间

    import java.util.Vector;
	
	public class maxHeapTest{
		public static void main(String[] args){
			Vector v = new Vector();
			for(int i = 0; i <= 10; i++){
				byte[] b = new byte[1024 * 1024];
				v.add(b);
				System.out.println(i + " M is allocated");
			}
	
			System.out.println("Max memory: "+Runtime.getRuntime().maxMemory());
		}
	}
>运行结果

    0M is allocated
	1M is allocated
	2M is allocated
	3M is allocated
	Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at maxHeapTest.main(maxHeapTest.java:8)

此时表明在完成 4MB 数据分配后系统空闲的堆内存大小已经不足 1MB 了。
设置 GC 新生代区大小
参数-Xmn 或者用于 Hot Spot 虚拟机中的参数-XX:NewSize(新生代初始大小)、-XX：MaxNewSize 用于设置新生代的大小。设置一个较大的新生代会减小老生代的大小，这个参数对系统性能以及 GC 行为有很大的影响。新生代的大小一般设置为整个堆空间的 1/4 到 1/3 左右。
以清单 9 的代码为例，若使用 JVM 参数-XX:+PrintGCDetails -Xmx11M -XX:NewSize=2M -XX:MaxNewSize=2M -verbose:gc 运行程序，将新生代的大小减小为 2MB，那么 MinorGC 次数将从 4 次增加到 9 次 (默认情况下是 3.5MB 左右)。

>运行输出

    [GC [DefNew: 1272K->150K(1856K), 0.0028101 secs] 1272K->1174K(11072K),
	   0.0028504 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	[GC [DefNew: 1174K->0K(1856K), 0.0018805 secs] 2198K->2198K(11072K),
	0.0019097 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	clearing....
	[GC [DefNew: 1076K->0K(1856K), 0.0004046 secs] 3274K->2198K(11072K),
	0.0004382 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	[GC [DefNew: 1024K->0K(1856K), 0.0011834 secs] 3222K->3222K(11072K),
	0.0013508 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	[GC [DefNew: 1024K->0K(1856K), 0.0012983 secs] 4246K->4246K(11072K),
	0.0013299 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
	clearing....
	[GC [DefNew: 1024K->0K(1856K), 0.0001441 secs] 5270K->4246K(11072K),
	0.0001686 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	[GC [DefNew: 1024K->0K(1856K), 0.0012028 secs] 5270K->5270K(11072K),
	0.0012328 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	[GC [DefNew: 1024K->0K(1856K), 0.0012553 secs] 6294K->6294K(11072K),
	0.0012845 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	clearing....
	[GC [DefNew: 1024K->0K(1856K), 0.0001524 secs] 7318K->6294K(11072K),
	0.0001780 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
	Heap
	 def new generation total 1856K, used 1057K [0x36410000, 0x36610000, 0x36610000)
	 eden space 1664K, 63% used [0x36410000, 0x365185a0, 0x365b0000)
	 from space 192K, 0% used [0x365e0000, 0x365e0088, 0x36610000)
	 to space 192K, 0% used [0x365b0000, 0x365b0000, 0x365e0000)
	 tenured generation total 9216K, used 6294K [0x36610000, 0x36f10000, 0x37010000)
	 the space 9216K, 68% used [0x36610000, 0x36c35868, 0x36c35a00, 0x36f10000)
	 compacting perm gen total 12288K, used 375K [0x37010000, 0x37c10000, 0x3b010000)
	 the space 12288K, 3% used [0x37010000, 0x3706dc88, 0x3706de00, 0x37c10000)
	 ro space 10240K, 51% used [0x3b010000, 0x3b543000, 0x3b543000, 0x3ba10000)
	 rw space 12288K, 55% used [0x3ba10000, 0x3c0ae4f8, 0x3c0ae600, 0x3c610000)

设置持久代大小
持久代 (方法区) 不属于堆的一部分。在 Hot Spot 虚拟机中，使用-XX:MaxPermSize 参数可以设置持久代的最大值，使用-XX:PermSize 可以设置持久代的初始大小。持久代的大小直接决定了系统可以支持多少个类定义和多少常量。对于使用 CGLIB 或者 Javassit 等动态字节码生成工具的应用程序而言，设置合理的持久代大小有助于维持系统稳定。系统所支持的最大类与 MaxPermSize 成正比。一般来说，MaxPermSize 设置为 64MB 已经可以满足绝大部分应用程序正常工作。如果依然出现永久区溢出，可以设置为 128MB。这是两个很常用的永久区取值。如果 128MB 依然不能满足应用程序需求，那么对于大部分应用程序来说，则应该考虑优化系统的设计，减少动态类的产生，或者利用 GC 回收部分驻扎在永久区的无用类信息，以使系统健康运行。
设置线程栈大小
线程栈是线程的一块私有空间。在 JVM 中可以使用-Xss 参数设置线程栈的大小。
在线程中进行局部变量分配，函数调用时都需要在栈中开辟空间。如果栈的空间分配太小，那么线程在运行时可能没有足够的空间分配局部变量或者达不到足够的函数调用深度，导致程序异常退出；如果栈空间过大，那么开设线程所需的内存成本就会上升，系统所能支持的线程总数就会下降。由于 Java 堆也是向操作系统申请内存空间的，因此，如果堆空间过大，就会导致操作系统可用于线程栈的内存减少，从而间接减少程序所能支持的线程数量。

>尝试开启尽可能多的线程

    public class TestXss{
	    public static class MyThread extends Thread{
		    @Override
		    public void run(){
			    try{
				    Thread.sleep(10000);
			    }catch(InterruptedException e){
				    e.printStackTrace();
			    }
		    }
	    }

		public static void main(String[] args){
			int count = 0;

			try{
				for(int i = 0; i < 10000; i++){
					new MyThread().start();
					count++;
				}
			}catch(OutOfMemoryError e){
				System.out.println(count);
				System.out.println(e.getMessage());
			}
		}
    }
>虚拟机配置

    -Xss1M

>运行输出

    1578
	unable to create new native thread
	
>虚拟机配置

    -Xss20M

>运行输出

    69
	unable to create new native thread

实验证明如果改变系统的最大堆空间设定，可以发现系统所能支持的线程数量也会相应改变。
Java 堆的分配以 200MB 递增，当栈大小为 1MB 时，最大线程数量以 200 递减。当系统物理内存被堆占据时，就不可以被栈使用。当系统由于内存空间不够而无法创建新的线程时会抛出 OOM 异常。这并不是由于堆内存不够而导致的 OOM，而是因为操作系统内存减去堆内存后剩余的系统内存不足而无法创建新的线程。在这种情况下可以尝试减少堆内存以换取更多的系统空间来解决这个问题。综上所述，如果系统确实需要大量线程并发执行，那么设置一个较小的堆和较小的栈有助于提高系统所能承受的最大线程数。
设置堆的比例分配
参数-XX：SurvivorRatio 是用来设置新生代中 eden 空间和 s0 空间的比例关系。s0 和 s1 空间又分别称为 from 空间和 to 空间。它们的大小是相同的，职能也是相同的，并在 Minor GC 后互换角色。

>演示不断插入字符时使用的GC输出

    import java.util.ArrayList;
		import java.util.List;
	
	public class StringDemo {
	 public static void main(String[] args){
		 List<String> handler = new ArrayList<String>();
		 for(int i=0;i<1000;i++){
		 HugeStr h = new HugeStr();
		 ImprovedHugeStr h1 = new ImprovedHugeStr();
		 handler.add(h.getSubString(1, 5));
		 handler.add(h1.getSubString(1, 5));
	 }
	 }
	 
	 static class HugeStr{
		 private String str = new String(new char[800000]);
		 public String getSubString(int begin,int end){
		 return str.substring(begin, end);
	 }
	 }
	 
	 static class ImprovedHugeStr{
		 private String str = new String(new char[10000000]);
		 public String getSubString(int begin,int end){
		 return new String(str.substring(begin, end));
	 }
	 }
	}

>设置新生代堆为10MB，并使eden区是s0的8

    -XX:+PrintGCDetails -XX:MaxNewSize=10M -XX:SurvivorRatio=8

>运行输出

    [Full GC [Tenured: 233756K->233743K(251904K), 0.0524229 secs] 233756K->233743K(261120K),
		[Perm : 377K->372K(12288K)], 0.0524703 secs] [Times: user=0.06 sys=0.00, real=0.06 secs] 
	def new generation total 9216K, used 170K [0x27010000, 0x27a10000, 0x27a10000)
	 eden space 8192K, 2% used [0x27010000, 0x2703a978, 0x27810000)
	 from space 1024K, 0% used [0x27910000, 0x27910000, 0x27a10000)
	 to space 1024K, 0% used [0x27810000, 0x27810000, 0x27910000)
	 tenured generation total 251904K, used 233743K [0x27a10000, 0x37010000, 0x37010000)
	 the space 251904K, 92% used [0x27a10000, 0x35e53d00, 0x35e53e00, 0x37010000)
	 compacting perm gen total 12288K, used 372K [0x37010000, 0x37c10000, 0x3b010000)
	 the space 12288K, 3% used [0x37010000, 0x3706d310, 0x3706d400, 0x37c10000)
	 ro space 10240K, 51% used [0x3b010000, 0x3b543000, 0x3b543000, 0x3ba10000)
	rw space 12288K, 55% used [0x3ba10000, 0x3c0ae4f8, 0x3c0ae600, 0x3c610000)

修改参数 SurvivorRatio 为 2 运行程序，相当于设置 eden 区是 s0 的 2 倍大小，由于 s1 与 s0 相同，故有 eden=[10MB/(1+1+2)]*2=5MB。


>运行输出

    [Full GC [Tenured: 233756K->233743K(251904K), 0.0546689 secs] 233756K->233743K(259584K),
	   [Perm : 377K->372K(12288K)],0.0547257 secs] [Times: user=0.05 sys=0.00, real=0.05 secs] 
	def new generation total 7680K, used 108K [0x27010000, 0x27a10000, 0x27a10000)
	 eden space 5120K, 2% used [0x27010000, 0x2702b3b0, 0x27510000)
	 from space 2560K, 0% used [0x27510000, 0x27510000, 0x27790000)
	 to space 2560K, 0% used [0x27790000, 0x27790000, 0x27a10000)
	 tenured generation total 251904K, used 233743K [0x27a10000, 0x37010000, 0x37010000)
	 the space 251904K, 92% used [0x27a10000, 0x35e53d00, 0x35e53e00, 0x37010000)
	 compacting perm gen total 12288K, used 372K [0x37010000, 0x37c10000, 0x3b010000)
	 the space 12288K, 3% used [0x37010000, 0x3706d310, 0x3706d400, 0x37c10000)
	 ro space 10240K, 51% used [0x3b010000, 0x3b543000, 0x3b543000, 0x3ba10000)
	rw space 12288K, 55% used [0x3ba10000, 0x3c0ae4f8, 0x3c0ae600, 0x3c610000)

######Java堆参数总结

Java 堆操作是主要的数据存储操作，总结的主要参数配置如下。
与 Java 应用程序堆内存相关的 JVM 参数有：
-Xms：设置 Java 应用程序启动时的初始堆大小；
-Xmx：设置 Java 应用程序能获得的最大堆大小；
-Xss：设置线程栈的大小；
-XX：MinHeapFreeRatio：设置堆空间最小空闲比例。当堆空间的空闲内存小于这个数值时，JVM 便会扩展堆空间；
-XX：MaxHeapFreeRatio：设置堆空间的最大空闲比例。当堆空间的空闲内存大于这个数值时，便会压缩堆空间，得到一个较小的堆；
-XX：NewSize：设置新生代的大小；
-XX：NewRatio：设置老年代与新生代的比例，它等于老年代大小除以新生代大小；
-XX：SurvivorRatio：新生代中 eden 区与 survivor 区的比例；
-XX：MaxPermSize：设置最大的持久区大小；
-XX：TargetSurvivorRatio： 设置 survivor 区的可使用率。当 survivor 区的空间使用率达到这个数值时，会将对象送入老年代。

从所有这些参数描述信息和代码示例可以看到，没有哪一条固定的规则可以供程序员参考。性能优化需要根据您应用的实际情况来有选择性地挑选参数及配制值，没有完全绝对的最优方案，最优方案是基于您对 JVM 数据存储方式及自己代码的了解程度来作出的最佳选择。
