###Java中的锁
锁是类似于同步块的一种线程同步机制除了锁比Java中同步块更复杂外。锁是使用同步块创建的，因此，我们并不能完全摆脱<code>synchronized</code>关键字。

在<code>java.util.concurrent.locks</code>包中有几种锁实现，所以你可能自己实现锁。但是，你仍然需要知道如何使用它们以及它们背后的实现原理。

####一个简单的锁

让我们先看一个Java代码中的同步块：

    public class Counter{
	    private int count = 0;

		public int inc(){
			synchronized(this){
				return ++count;
			}
		}
    }
注意，在<code>inc</code>方法中的<code>synchronized(this)</code>代码块。这个代码块确保了在某一时刻只有一个线程可以执行<code>return ++count</code>。

<code>Counter</code>还可以像如下这样实现，使用一个<code>Lock</code>来代替同步块。

    public class Counter{
	    private Lock lock = new Lock();
	    private int count = 0;

		public int inc(){
			lock.lock();
			int newCount = ++count;
			lock.unlock();
			return newCount;
		}
    }

<code>lock()</code>锁住一个<code>Lock</code>实例，这样所有的线程在调用<code>lock()</code>方法都会阻塞知道<code>unlock</code>方法被执行。

下面是一个<code>Lock</code>的简单实现：

    public class Lock{
	    private boolean isLocked = false;

		public synchronized void lock(){
			while(isLocked){
				wait();
			}
			isLocked = true;
		}

		public synchronized void unlock(){	
			isLocked = false;
			notify();
		}
    }

注意，<code>while(isLocked)</code>循环也被称作“自旋锁”（spin lock）。当<code>isLocked</code>为<code>true</code>时，调用<code>lock()</code>的线程在<code>wait()</code>调用后阻塞等待。为防止该线程没有收到notify()调用也从wait()中返回（也称作虚假唤醒），这个线程会重新去检查isLocked条件以决定当前是否可以安全地继续执行还是需要重新保持等待，而不是认为线程被唤醒了就可以安全地继续执行了。如果<code>isLocked</code>为false，当前线程会退出<code>while(isLocked)</code>循环，并将<code>isLocked</code>设回true，让其它正在调用<code>lock()</code>方法的线程能够在Lock实例上加锁。

当线程执行完临界区的代码，调用<code>unlock()</code>方法。执行<code>unlock()</code>将<code>isLocked</code>设置回false，然后唤醒在<code>lock()</code>方法中<code>wait()</code>调用上阻塞等待的一个线程。

####锁重入

在Java中同步块是可以重入的。意思是，如果一个Java线程进入了一个同步代码块，进而持有同步代码块中的管程对象，这个线程也可以进入被同一个管程同步代码块。这儿有一个例子：

    public class Reentrant{
	    public synzhronized outer(){
		    inner();
	    }

		public synchronized inner(){
			//do something
		}
    }
    
我们注意到，<code>outer</code>和<code>inner</code>方法都被声明为<code>synchronized</code>，这在Java中等同于一个<code>synchronized(this)</code>块。如果一个线程调用<code>outer()</code>方法，然后在<code>outer()</code>方法中调用<code>inner()</code>方法是可以的，因为这两个方法（块）都被同步在同一个管程对象"this"上。如果一个线程已经持有个一个管程对象上的锁，它可以访问所有同步在这个管程对象上的代码块。这被称作重入（reentrance）。

前面的锁实现都是非重入的。如果我们像下面这样重写<code>Reentrance</code>类，调用<code>outer()</code>方法的线程将会阻塞在<code>inner()</code>方法中的<code>lock.lock()</code>里面。

    public class Reentrance2{
	    Lock lock = new Lock();

		public outer(){	
			lock.lock();
			inner();
			lock.unlock();
		}

		public synchronized inner(){
			lock.lock();
			//do something
			lock.unlock();
		}
    }

一个线程调用<code>outer()</code>会首先锁住<code>Lock</code>实例，然后继续调用<code>inner()</code>。在<code>inner()</code>内部，这个线程将会再次尝试锁住<code>Lock</code>实例。这将会失败（这个线程会被阻塞），因为<code>Lock</code>实例已经在<code>outer()</code>方法中被锁住。

当我们查看<code>lock()</code>方法的实现时，会发现线程两次调用<code>lock()</code>方法的过程中第二次在没有调用<code>unlock()</code>方法的情况下会被阻塞的原因也很明显：

    public class Lock{
	    boolean isLocked = false;
	
		public synchronized void lock()throws InterruptedException{
			while(isLocked){
				wait();
			}

			isLocked = true;
		}

		...
    }

while循环里面的条件决定了一个线程是否被允许离开<code>lock()</code>方法。当前的判断条件是只有当isLocked为false时lock操作才被允许，而没有考虑是哪个线程锁住了它。

为了使<code>Lock</code>可重入，我们需要做一点小小的改动。

    public class Lock{
	    boolean isLocked = false;
	    Thread lockedBy = null;
	    int lockedCount = 0;

		public synzhronized void lock()throws InterruptedException{
			Thread callingThread = Thread.currentThread();
			while(isLocked && lockedBy != callingThread){
				wait();
			}
			isLocked = true;
			lockedCount++;
			lockedBy = callingThread;
		}

		public synchronized void unlock(){
			if(Thread.currentThread() == this.lockBy){
				lockedCount--;

				if(lockedCount == 0){
					isLocked = false;
					notify();
				}
			}
		}
    }

注意到现在的while循环（自旋锁）也考虑到了已锁住该Lock实例的线程。如果当前的锁对象没有被加锁(isLocked = false)，或者当前调用线程已经对该Lock实例加了锁，那么while循环就不会被执行，调用lock()的线程就可以退出该方法（译者注：“被允许退出该方法”在当前语义下就是指不会调用wait()而导致阻塞）。

除此之外，我们需要记录同一个线程重复对一个锁对象加锁的次数。否则，一次unblock()调用就会解除整个锁，即使当前锁已经被加锁过多次。在unlock()调用没有达到对应lock()调用的次数之前，我们不希望锁被解除。

现在这个Lock类就是可重入的了。

####锁的公平性

Java的synchronized块并不保证尝试进入它们的线程的顺序。因此，如果多个线程不断竞争访问相同的synchronized同步块，就存在一种风险，其中一个或多个线程永远也得不到访问权 —— 也就是说访问权总是分配给了其它线程。这种情况被称作线程饥饿。为了避免这种问题，锁需要实现公平性。本文所展现的锁在内部是用synchronized同步块实现的，因此它们也不保证公平性。饥饿和公平中有更多关于该内容的讨论。

####在finally语句中调用unlock()

如果用Lock来保护临界区，并且临界区有可能会抛出异常，那么在finally语句中调用unlock()就显得非常重要了。这样可以保证这个锁对象可以被解锁以便其它线程能继续对其加锁。以下是一个示例：

    lock.lock();
	try{
		//do critical section code, which may throw exception
	}finally{
		lock.unlock();
	}

这个简单的结构可以保证当临界区抛出异常时Lock对象可以被解锁。如果不是在finally语句中调用的unlock()，当临界区抛出异常时，Lock对象将永远停留在被锁住的状态，这会导致其它所有在该Lock对象上调用lock()的线程一直阻塞。



