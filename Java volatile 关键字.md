####Java volatile 关键字
在Java中，<code>valatile</code>关键字被用来标识一个Java变量 ”being stored in main memory“。更准确地讲，每次读取一个<code>volatile</code>变量都会从主存中去读，而不是从CPU缓存中；每次修改一个<code>volatile</code>变量都会将其写入到主存中，而不仅仅是CPU缓存中。

#####保证内存可见性
在一个多线程的程序中，多个线程操作一个非volatile修饰的变量，每个线程可能会把变量从主存复制到CPU缓存中去，处于性能原因，如果你的计算机装有多个CPU，每个线程可能运行在不同的CPU上。这也就意味着，每个线程可能将变量拷贝到不同CPU到的CPU缓存中去。如下图：

![volatile](http://tutorials.jenkov.com/images/java-concurrency/java-volatile-1.png)

对于非volatile变量，Java虚拟机将数据从主存读入到CPU缓存，或者从CPU缓存写回到主存中并没有任何保证。

看这样的一个例子，多个线程访问一个包含一个counter变量的共享对象，声明如下：

    public class SharedObject{
	    public int counter = 0;
    }
线程1读取共享counter变量（值为0）到它的CPU的缓存中，然后将counter增加到1，没有将变化后的值写回到主存中。线程2做了和线程1同样的行为。线程1和线程2并没有进行同步。counter变量实际的值应该为2，但是每个线程在它们的CPU缓存中读到的变量的值为1，在主存中counter变量的值还是0.尽管线程最终会将counter变量的值写回到主存中，但值将会是错误的。

如果将counter声明为<code>volatile</code>，JVM会保证每次都从主存中读取变量，每次修改过值后都会写回到主存中。声明如下：

    public class SharedObject{
	    public volatile int counter = 0;
    }
在某些场景下，将变量简单声明为<code>volatile</code>就可以完全确保多个线程访问这个变量时可以最后被修改的值。

在两个线程都读和写同一个变量时，简单的将变量声明为<code>volatile</code>是不够的。
在CPU1中，线程1可能将counter变量（值为0）读入到一个CPU寄存器中。与此同时（或者稍靠后），在CPU2中，线程2可能将counter变量（值为0）读入到一个CPU寄存器中。两个线程都从主存中读取变量。现在，都将变量的值加1，然后写回到主存中。它们都增加counter的寄存器版本为1，都将值1写回到主存中。经过两次自增后counter变量的值应该为2.

这个多线程的问题就是没有看到变量最后被修改后的值因为值还没有被写回到主存中，一个线程的更新对其它线程是不可见的。

#####volatile
实际上，<code>volatile</code>保证了两点：
- **内存可见性**
- **防止指令重排序**

#####volatile的适用场景
如果一个线程读和写一个<code>volatile</code>变量，其它线程只读取这个变量，然后读线程可以确保看到这个变量最后的被修改的值。如果这个变量不被修改为<code>volatile</code>，将没有这样的担保。

#####volatile的性能考虑
- **发生在主存上的读写操作的代价要高于访问CPU缓存**
- **volatile阻止了指令重排序这种性能优化技术**