####Java中的ThreadLocal
在Java中的<code>ThreadLocal</code>类允许你创建的变量只被同一个线程读和写。因此，即使两个线程执行同样的代码，代码中有一个<code>ThreadLocal</code>类型的变量，两个线程相互之间也不能“看到”对方的<code>ThreadLocal</code>类型的变量、

#####创建一个ThreadLocal

    private ThreadLocal myThreadLocal = new ThreadLocal();

#####操作一个ThreadLocal

    myThreadLocal.set("A thread local value");

    String threadLocalValue = (String)myThreadLocal.get();

#####泛型的ThreadLocal
	

    private ThreadLocal myThreadLocal = new ThreadLocal<String>();
	myThreadLocal.set("Hello ThreadLocal");
	String threadLocalValue = myThreadLocal.get();

#####初始化ThreadLocal值

    private ThreadLocal myThreadLocal = new ThreadLocal<String>(){
	    @Override
	    protected String initialValue(){
		    return "This is the initial value";
	    }
    }

初始值对所有线程可见。

#####