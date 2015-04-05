####Java同步代码块
#####synchronized关键字
在Java中使用<code>synchronized</code>关键字来标识同步代码块。在Java中，一个同步代码块是在一些对象上进行同步。所有的同步代码块在某一个时刻只能有一个线程执行代码块里的代码。所有企图访问同步代码块的其它线程都会阻塞直到代码块里的线程离开代码块。

<code>synchronized</code>关键字可以用来标识四种不同类型的块：
- **实例方法**
- **静态方法**
- **实例方法中的代码块**
- **静态方法中的代码块**

#####同步实例方法

    public synchronized void add value(int value){
	    this.count += value;
    }

#####同步静态方法

    public static synchronized void add(int value){
	    count += value;
    }

#####实例方法中的同步代码块

    public void add(int value){
	    synchronized(this){
		    this.count += value;
	    }
    }
#####静态方法中的同步代码块

    public static void add(int value){
	    synchronized(this){
		    count += value;
	    }
    }
