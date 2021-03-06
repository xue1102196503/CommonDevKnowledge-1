### （二）java深入源码级面试题

* **哪些情况下的对象会被垃圾回收机制处理掉？**

	Java GC机制主要完成3件事：确定哪些内存需要回收，确定什么时候需要执行GC，如何执行GC。
	
	分代分配，分代回收。对象将根据存活的时间被分为：年轻代（Young Generation）、年老代（Old Generation）、永久代（Permanent Generation，也就是方法区）
	
	**年轻代Minor GC或叫Young GC:**采用停止-复制（Stop-and-copy）”清理法。分为Eden区和两个Survivor区。内存首先在Eden区内连续分配，当Eden满时执行一次GC，仍存活的移入同一个survivor区，重复上述过程直至当前survivor区满，GC后仍存活的直接移入另一个survivor区（总有一个survivor为空）
	
	**年老代Major GC也叫Full GC:**对象如果在年轻代存活了足够长的时间而没有被清理掉（即在几次 Young GC后存活了下来），则会被复制到年老代，年老代的空间一般比年轻代大，能存放更多的对象，在年老代上发生的GC次数也比年轻代少。</br>
	如果对象比较大（比如长字符串或大数组），Young空间不足，则大对象会直接分配到老年代上（大对象可能触发提前GC，应少用，更应避免使用短命的大对象）。</br>
	年老代中维护一个512 byte的块——”card table“，所有老年代对象引用新生代对象的记录都记录在这里。Young GC时，只要查这里即可，不用再去查全部老年代，因此性能大大提高。
	
	**方法区（永久代）：**永久代的回收有两种：常量池中的常量，无用的类信息，常量的回收很简单，没有引用了就可以被回收。对于无用的类进行回收，必须保证3点：
	
	1. 类的所有实例都已经被回收
	2. 加载类的ClassLoader已经被回收
	3. 类对象的Class对象没有被引用（即没有通过反射引用该类的地方）

	引用计数法(循环引用时失效)  标记-清除算法 标记-整理算法 copying算法 generation算法
	
* **讲一下常见编码方式？**

 ASCII 码  128位
 
 GBK 《汉字内码扩展规范》
 
 UTF-16 Unicode码 2字节定长编码
 
 UTF-8 1~6字节变长编码
 
* **utf-8编码中的中文占几个字节；int型几个字节？**

	常用中文字符占用3个字节（大约2万多字），但超大字符集中的更大多数汉字要占4个字节
	</br>英文一个字节
	
* **静态代理和动态代理的区别，什么场景使用？（Proxy）**

	静态：由程序员创建或工具生成代理类的源码，再编译代理类。所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。装饰器模式。
	</br>优点：业务类只需要关注业务逻辑本身，保证了业务类的重用性。这是代理的共有优点。 
</br>缺点： 
1）代理对象的一个接口只服务于一种类型的对象，如果要代理的方法很多，势必要为每一种方法都进行代理，静态代理在程序规模稍大时就无法胜任了。</br> 
2）如果接口增加一个方法，除了所有实现类需要实现这个方法外，所有代理类也需要实现此方法。增加了代码维护的复杂度。

	动态：

	在静态代理模式下，Proxy所做的事情，无非是调用在不同的request时，调用触发realSubject对应的方法；更抽象点看，Proxy所作的事情；在Java中 方法（Method）也是作为一个对象来看待了，动态代理工作的基本模式就是将自己的方法功能的实现交给 InvocationHandler角色，外界对Proxy角色中的每一个方法的调用，Proxy角色都会交给InvocationHandler来处理，而InvocationHandler则调用具体对象角色的方法。
	
	![](https://ws3.sinaimg.cn/large/006tKfTcgy1fp22kmq4pcj30ku06u74w.jpg) 
	
	具体步骤是： 	
	a. 实现InvocationHandler接口创建自己的调用处理器 </br>
	b. 给Proxy类提供ClassLoader和代理接口类型数组创建动态代理类 </br>
	c. 以调用处理器类型为参数，利用反射机制得到动态代理类的构造函数 </br>
	d. 以调用处理器对象为参数，利用动态代理类的构造函数创建动态代理类对象</br> 
	
	动态代理优点 ：
1.动态代理类的字节码在程序运行时由Java反射机制动态生成，无需程序员手工编写它的源代码。 
2.动态代理类不仅简化了编程工作，而且提高了软件系统的可扩展性，因为Java 反射机制可以生成任意类型的动态代理类。

```
public class MyInvocationHandler implements InvocationHandler {
    private String TAG = "MyInvocationHandler";
    private Object obj;

    public Object bind(Object obj){
        this.obj = obj;
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(),obj.getClass().getInterfaces(),this);
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Log.d(TAG,"start"+method.getName());
        Object result = method.invoke(obj,args);
        Log.d(TAG,"end"+method.getName());
        return result;
    }
}

	
	Human human = new HumanImp();
        ((TextView)findViewById(R.id.tv_content)).setText(((Human)new MyInvocationHandler().bind(human)).eat());
```


   缺点： 
	JDK的动态代理机制只能代理实现了接口的类，而不能实现接口的类就不能实现JDK的动态代理，cglib是针对类来实现代理的，他的原理是对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。	

作用：1，方法增强，让你可以在不修改源码的情况下，增强一些方法，比如添加日志等。
2.以用作远程调用，好多rpc框架就是用代理方式实现的。

[参考文章](http://blog.csdn.net/fengyuzhengfan/article/details/49586277)
	
* **Java的异常体系**

	![](https://ws3.sinaimg.cn/large/006tNc79gy1fp31ljunraj30na0dudlm.jpg)

   Error是程序无法处理的错误，比如OutOfMemoryError、ThreadDeath等。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。   
	Exception是程序本身可以处理的异常，这种异常分两大类运行时异常（不检查异常）和非运行时异常（检查异常，不处理，程序就不能编译通过）。程序中应当尽可能去处理这些异常。
	
	try、catch、finally:   
	 * try、catch、finally三个语句块均不能单独使用，三者可以组成 try...catch...finally、try...catch、 
    try...finally三种结构，catch语句可以有一个或多个，finally语句最多一个。 
    * try、catch、finally三个代码块中变量的作用域为代码块内部，分别独立而不能相互访问。 
    如果要在三个块中都可以访问，则需要将变量定义到这些块的外面。 
    * 多个catch块时候，只会匹配其中一个异常类并执行catch块代码，而不会再执行别的catch块， 
    并且匹配catch语句的顺序是由上到下。

	throw:方法体内部
	</br>throws:方法体外部


