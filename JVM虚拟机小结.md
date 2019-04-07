# 			JVM虚拟机

原文:http://www.cnblogs.com/ityouknow/p/5603287.html

## 一. java类的加载机制

### 1、**什么是类的加载**

类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的**方法区**内，然后在**堆区**创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。

[![wps5F9E.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160621125941772-1913742708.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160621125940990-365229277.png)

类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误（LinkageError错误）如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误.

```
加载.class文件的方式
– 从本地系统中直接加载
– 通过网络下载.class文件
– 从zip，jar等归档文件中加载.class文件
– 从专有数据库中提取.class文件
– 将Java源文件动态编译为.class文件
```

### 2、**类的生命周期**

[![wps257C.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160621125943209-1443333281.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160621125942490-979409578.png)

​	其中类加载的过程包括了**加载、验证、准备、解析、初始化**五个阶段。在这五个阶段中，加载、验证、准备和初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定（也成为动态绑定或晚期绑定）。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。

#### •**加载**：查找并加载类的二进制数据

   加载时类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：

​    1、通过一个类的全限定名来获取其定义的二进制**字节流**。

​    2、将这个字节流所代表的**静态存储结构转化**为方法区的**运行时数据结构**。

​    3、在Java堆中生成一个代表这个类的**java.lang.Class对象**，作为对方法区中这些数据的访问入口。

​    相对于类加载的其他阶段而言，加载阶段（准确地说，是加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载。

​    加载阶段完成后，虚拟机外部的 二进制字节流就按照虚拟机所需的格式存储在方法区之中，而且在Java堆中也创建一个java.lang.Class类的对象，这样便可以通过该对象访问方法区中的这些数据。

#### •**连接**

#####  – 验证：确保被加载的类的正确性

​	验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成4个阶段的检验动作：

​	**文件格式验证**：验证字节流是否符合Class文件格式的规范；例如：是否以0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。

​	**元数据验证**：对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了java.lang.Object之外。

​	**字节码验证**：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。

​	**符号引用验证**：确保解析动作能正确执行。

​	验证阶段是非常重要的，但**不是必须**的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用**-Xverifynone**参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。

 

#####  – 准备：为类的静态变量分配内存，并将其初始化为默认值

   准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都将在方法区中分配。对于该阶段有以下几点需要注意：

​    	1、这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。

​    	2、这里所设置的初始值通常情况下是数据类型默认的零值（如0、0L、null、false等），而不是被在Java代码中被显式地赋予的值。

   假设一个类变量的定义为：public static int value = 3；

   那么变量value在准备阶段过后的初始值为0，而不是3，因为这时候尚未开始执行任何Java方法，而把value赋值为3的putstatic指令是在程序编译后，存放于类构造器<clinit>（）方法之中的，所以把value赋值为3的动作将在初始化阶段才会执行。

```
· 这里还需要注意如下几点：
· 对基本数据类型来说，对于类变量（static）和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值，而对于局部变量来说，在使用前必须显式地为其赋值，否则编译时不通过。
· 对于同时被static和final修饰的常量，必须在声明的时候就为其显式地赋值，否则编译时不通过；而只被final修饰的常量则既可以在声明时显式地为其赋值，也可以在类初始化时显式地为其赋值，总之，在使用前必须为其显式地赋值，系统不会为其赋予默认零值。
· 对于引用数据类型reference来说，如数组引用、对象引用等，如果没有对其进行显式地赋值而直接使用，系统都会为其赋予默认的零值，即null。
· 如果在数组初始化时没有对数组中的各元素赋值，那么其中的元素将根据对应的数据类型而被赋予默认的零值。
```

​	 3、如果类字段的字段属性表中存在ConstantValue属性，即同时被final和static修饰，那么在准备阶段变量value就会被初始化为ConstValue属性所指定的值。

   假设上面的类变量value被定义为： public static final int value = 3；

   编译时Javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将value赋值为3。回忆上一篇博文中对象被动引用的第2个例子，便是这种情况。我们可以理解为static final常量在编译期就将其结果放入了调用它的类的常量池中

 

#####  – 解析：**把类中的符号引用转换为直接引用**

​	解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。**符号引用**就是一组符号来描述目标，可以是任何字面量。

​	**直接引用**就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

#### •**初始化**

​	初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。在Java中对类变量进行初始值设定有两种方式：

  	①声明类变量时指定初始值

  	②使用静态代码块为类变量指定初始值

 JVM初始化步骤

​	 1、假如这个类还没有被加载和连接，则程序先加载并连接该类

​	 2、假如该类的直接父类还没有被初始化，则先初始化其直接父类

​	 3、假如类中有初始化语句，则系统依次执行这些初始化语句

类初始化时机：只有当对类的**主动使用**的时候才会导致类的初始化，类的主动使用包括以下六种：

– 创建类的实例，也就是**new**的方式

– 访问某个类或接口的**静态变量**，或者对该静态变量赋值

– 调用类的**静态方法**

– **反射**（如Class.forName(“com.shengsiyuan.Test”)）

– 初始化某个类的子类，则其**父类**也会被初始化

– Java虚拟机启动时被标明为**启动类**的类（包含main()方法Java Test），直接使用java.exe命令来运行某个主类

 

#### **结束生命周期**

•在如下几种情况下，Java虚拟机将结束生命周期

– 执行了System.exit()方法

– 程序正常执行结束

– 程序在执行过程中遇到了异常或错误而异常终止

– 由于操作系统出现错误而导致Java虚拟机进程终止

 

### 3、**类加载器**

#### **1. 类加载器**

先来一个小例子

```java
package com.neo.classloader;
public class ClassLoaderTest {
     public static void main(String[] args) {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        System.out.println(loader);
        System.out.println(loader.getParent());
        System.out.println(loader.getParent().getParent());
    }
}
```

运行后，输出结果：

```
sun.misc.Launcher$AppClassLoader@64fef26a
sun.misc.Launcher$ExtClassLoader@1ddd40f3
null
```

从上面的结果可以看出，并没有获取到ExtClassLoader的父Loader，原因是Bootstrap Loader（引导类加载器）是用C语言实现的，找不到一个确定的返回父Loader的方式，于是就返回null。

这几种类加载器的层次关系如下图所示：

[![wpsC239.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160621125944459-1013316302.jpg)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160621125943787-249179344.jpg)

**注意：这里父类加载器并不是通过继承关系来实现的，而是采用组合实现的。**

站在Java虚拟机的角度来讲，只存在两种不同的类加载器：**启动类加载器**：它使用C++实现（这里仅限于Hotspot，也就是JDK1.5之后默认的虚拟机，有很多其他的虚拟机是用Java语言实现的），是虚拟机自身的一部分；所有其他的类加载器：这些类加载器都由Java语言实现，独立于虚拟机之外，并且全部继承自抽象类java.lang.ClassLoader，这些类加载器需要由启动类加载器加载到内存中之后才能去加载其他的类。

站在Java开发人员的角度来看，类加载器可以大致划分为以下三类：

​	**启动类加载器**：Bootstrap ClassLoader，负责加载存放在JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库（如rt.jar，所有的java.*开头的类均被Bootstrap ClassLoader加载）。启动类加载器是无法被Java程序直接引用的。

​	**扩展类加载器**：Extension ClassLoader，该加载器由sun.misc.Launcher$ExtClassLoader实现，它负责加载DK\jre\lib\ext目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库（如javax.*开头的类），开发者可以直接使用扩展类加载器。

​	**应用程序类加载器**：Application ClassLoader，该类加载器由sun.misc.Launcher$AppClassLoader来实现，它负责加载用户类路径（ClassPath）所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

应用程序都是由这三种类加载器互相配合进行加载的，如果有必要，我们还可以加入自定义的类加载器。因为JVM自带的ClassLoader只是懂得从本地文件系统加载标准的java class文件，因此如果编写了自己的ClassLoader，便可以做到如下几点：

​	1）在执行非置信代码之前，自动验证数字签名。

​	2）动态地创建符合用户特定需要的定制化构建类。

​	3）从特定的场所取得java class，例如数据库中和网络中。



#### **2. JVM类加载机制**

​	**•全盘负责**，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入

​	**•父类委托**，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类

​	**•缓存机制**，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效.



### 4、**类的加载**

类加载有三种方式：

​	1、命令行启动应用时候由JVM初始化加载

​	2、通过Class.forName()方法动态加载

​	3、通过ClassLoader.loadClass()方法动态加载

例子：

```java
package com.neo.classloader;
public class loaderTest { 
        public static void main(String[] args) throws ClassNotFoundException { 
                ClassLoader loader = HelloWorld.class.getClassLoader(); 
                System.out.println(loader); 
                //使用ClassLoader.loadClass()来加载类，不会执行初始化块 
                loader.loadClass("Test2"); 
                //使用Class.forName()来加载类，默认会执行初始化块 
//                Class.forName("Test2"); 
                //使用Class.forName()来加载类，并指定ClassLoader，初始化时不执行静态块 
//                Class.forName("Test2", false, loader); 
        } 
}
```

demo类

```java
public class Test2 { 
        static { 
                System.out.println("静态初始化块执行了！"); 
        } 
}
```

分别切换加载方式，会有不同的输出结果。



**Class.forName()和ClassLoader.loadClass()区别**

​	Class.forName()：将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块；

​	ClassLoader.loadClass()：只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。

**注**：

​	Class.forName(name, initialize, loader)带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 。



### 5、**双亲委派模型**

​	双亲委派模型的工作流程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

**双亲委派机制:**

1、当AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成。

2、当ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成。

3、如果BootStrapClassLoader加载失败（例如在$JAVA_HOME/jre/lib里未查找到该class），会使用ExtClassLoader来尝试加载；

4、若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

**ClassLoader源码分析：**

```java
public Class<?> loadClass(String name)throws ClassNotFoundException {
            return loadClass(name, false);
    }
    
    protected synchronized Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException {
            // 首先判断该类型是否已经被加载
            Class c = findLoadedClass(name);
            if (c == null) {
                //如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
                try {
                    if (parent != null) {
                         //如果存在父类加载器，就委派给父类加载器加载
                        c = parent.loadClass(name, false);
                    } else {
                    //如果不存在父类加载器，就检查是否是由启动类加载器加载的类，通过调用本地方法native Class findBootstrapClass(String name)
                        c = findBootstrapClass0(name);
                    }
                } catch (ClassNotFoundException e) {
                 // 如果父类加载器和启动类加载器都不能完成加载任务，才调用自身的加载功能
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
```

**双亲委派模型意义**：

-系统类防止内存中出现多份同样的字节码

-保证Java程序安全稳定运行



### 6、**自定义类加载器**

​    通常情况下，我们都是直接使用系统类加载器。但是，有的时候，我们也需要自定义类加载器。比如应用是通过网络来传输 Java 类的字节码，为保证安全性，这些字节码经过了加密处理，这时系统类加载器就无法对其进行加载，这样则需要自定义类加载器来实现。自定义类加载器一般都是继承自 ClassLoader 类，从上面对 loadClass 方法来分析来看，我们只需要重写 findClass 方法即可。下面我们通过一个示例来演示自定义类加载器的流程：

```java
import java.io.*;

public class MyClassLoader extends ClassLoader {

    private String root;

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] loadClassData(String className) {
        String fileName = root + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
        try {
            InputStream ins = new FileInputStream(fileName);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length = 0;
            while ((length = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, length);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    public String getRoot() {
        return root;
    }

    public void setRoot(String root) {
        this.root = root;
    }

    public static void main(String[] args)  {

        MyClassLoader classLoader = new MyClassLoader();
        classLoader.setRoot("E:\\temp");

        Class<?> testClass = null;
        try {
            testClass = classLoader.loadClass("com.neo.classloader.Test2");
            Object object = testClass.newInstance();
            System.out.println(object.getClass().getClassLoader());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```

自定义类加载器的核心在于对字节码文件的获取，如果是加密的字节码则需要在该类中对文件进行解密。由于这里只是演示，并未对class文件进行加密，因此没有解密的过程。这里有几点需要注意：

​	1、这里传递的文件名需要是类的全限定性名称，即com.paddx.test.classloading.Test格式的，因为 defineClass 方法是按这种格式进行处理的。

​	2、最好不要重写loadClass方法，因为这样容易破坏双亲委托模式。

​	3、这类Test 类本身可以被 AppClassLoader 类加载，因此我们不能把 com/paddx/test/classloading/Test.class 放在类路径下。否则，由于双亲委托机制的存在，会直接导致该类由 AppClassLoader 加载，而不会通过我们自定义类加载器来加载。



## 二. Java内存模型

### **Java的内存结构**：

[![JUtH_20121024_RuntimeDataAreas_6_MemoryModel](https://images2015.cnblogs.com/blog/331425/201606/331425-20160623115840235-1252768148.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160623115838891-809895495.png)

![img](https://images2015.cnblogs.com/blog/820406/201603/820406-20160326200119386-756216654.png)

JVM内存结构主要有三大块：**堆内存**、**方法区**和**栈**。堆内存是JVM中最大的一块由**年轻代**和**老年代**组成，而年轻代内存又被分成三部分，**Eden空间**、**From Survivor空间**、**To Survivor空间**,默认情况下年轻代按照8:1:1的比例来分配；

方法区存储类信息、常量、静态变量等数据，是线程共享的区域，为与Java堆区分，方法区还有一个别名Non-Heap(非堆)；栈又分为java虚拟机栈和本地方法栈主要用于方法的执行。



在通过一张图来了解如何通过参数来控制各区域的内存大小

[![jvm_m_l](https://images2015.cnblogs.com/blog/331425/201606/331425-20160623115841781-223449019.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160623115841031-564040608.png)

**控制参数**
-Xms设置堆的最小空间大小。

-Xmx设置堆的最大空间大小。

-XX:NewSize设置新生代最小空间大小。

-XX:MaxNewSize设置新生代最大空间大小。

-XX:PermSize设置永久代最小空间大小。

-XX:MaxPermSize设置永久代最大空间大小。

-Xss设置每个线程的堆栈大小。



没有直接设置老年代的参数，但是可以设置堆空间大小和新生代空间大小两个参数来间接控制。

  **老年代空间大小=堆空间大小-年轻代大空间大小**

 

从更高的一个维度再次来看JVM和系统调用之间的关系

**![002hLfJYgy71J9KxlH53b](https://images2015.cnblogs.com/blog/331425/201606/331425-20160623115846235-947282498.png)**

方法区和堆是所有线程共享的内存区域；而java栈、本地方法栈和程序员计数器是运行是线程私有的内存区域。

jdk1.8的运行时数据区

![这里写图片描述](https://img-blog.csdn.net/20180812235058303?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JydWNlMTI4/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



**下面我们详细介绍每个区域的作用**

#### 1. Java堆（Heap）

​    对于大多数应用来说，Java堆（Java Heap）是Java虚拟机所管理的内存中**最大**的一块。Java堆是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例，**几乎所有的对象实例都在这里分配内存。**

​     Java堆是垃圾收集器管理的主要区域，因此很多时候也被称做“**GC堆**”。如果从内存回收的角度看，由于现在收集器基本都是采用的分代收集算法，所以Java堆中还可以细分为：**新生代和老年代；再细致一点的有Eden空间、From Survivor空间、To Survivor空间等。**

​	根据Java虚拟机规范的规定，Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，就像我们的磁盘空间一样。在实现时，既可以实现成固定大小的，也可以是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的（通过-Xmx和-Xms控制）。

如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

​	绝大部分 Java 程序员应该都见过 "java.lang.OutOfMemoryError: PermGen space "这个异常。这里的 “PermGen space”其实指的就是方法区。不过方法区和“PermGen space”又有着本质的区别。前者是 JVM 的规范，而后者则是 JVM 规范的一种实现，并且只有 HotSpot 才有 “PermGen space”，而对于其他类型的虚拟机，如 JRockit（Oracle）、J9（IBM） 并没有“PermGen space”。由于方法区主要存储类的相关信息，所以对于动态生成类的情况比较容易出现永久代的内存溢出。最典型的场景就是，在 jsp 页面比较多的情况，容易出现永久代内存溢出。

​	在 JDK 1.8 中， HotSpot 已经没有 “PermGen space”这个区间了，取而代之是一个叫做 Metaspace（元空间） 的东西。下面我们就来看看 Metaspace 与 PermGen space 的区别。

**Metaspace（元空间）**

​	元空间（Metaspace）：存储已被虚拟机加载的类信息。随着JDK8的到来，JVM不再有方法区（PermGen），原方法区存储的信息被分成两部分：1、虚拟机加载的**类信息**，2、**运行时常量池**。分别被移动到了**元空间**和**堆**中。

　　其实，移除永久代的工作从JDK1.7就开始了。JDK1.7中，存储在永久代的部分数据就已经转移到了Java Heap或者是 Native Heap。但永久代仍存在于JDK1.7中，并没完全移除，譬如**符号引用**(Symbols)转移到了native heap；**字面量**(interned strings)转移到了java heap；类的**静态变量**(class statics)转移到了java heap。

​	元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用**本地内存**。因此，默认情况下，元空间的大小仅受本地内存限制，但可以通过以下参数来指定元空间的大小：

　　-XX:MetaspaceSize，初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。
　　-XX:MaxMetaspaceSize，最大空间，默认是没有限制的。

　　除了上面两个指定大小的选项以外，还有两个与 GC 相关的属性：
　　-XX:MinMetaspaceFreeRatio，在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集
　　-XX:MaxMetaspaceFreeRatio，在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集

​	通过上面分析，大家应该大致了解了 JVM 的内存划分，也清楚了 JDK 8 中永久代向元空间的转换。不过大家应该都有一个疑问，就是为什么要做这个转换？所以，最后给大家总结以下几点原因：

　　1、字符串存在永久代中，容易出现性能问题和内存溢出。

　　2、类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出，太大则容易导致老年代溢出。

　　3、永久代会为 GC 带来不必要的复杂度，并且回收效率偏低。

　　4、Oracle 可能会将HotSpot 与 JRockit 合二为一。

#### **2. 方法区（Method Area）1.8之后已被移除,用元数据区代替**	

  方法区（Method Area）与Java堆一样，是各个线程共享的内存区域，**它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据**。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap（非堆），目的应该是与Java堆区分开来。

​	对于习惯在HotSpot虚拟机上开发和部署程序的开发者来说，很多人愿意把方法区称为“永久代”（Permanent Generation），本质上两者并不等价，仅仅是因为HotSpot虚拟机的设计团队选择把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已。

​	Java虚拟机规范对这个区域的限制非常宽松，除了和Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择不实现垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入了方法区就如永久代的名字一样“永久”存在了。这个区域的内存回收目标主要是针对常量池的回收和对类型的卸载，一般来说这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收确实是有必要的。

根据Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

 

#### **3. 程序计数器（Program Counter Register）**

​	程序计数器（Program Counter Register）是一块较小的内存空间，它的作用可以看做是当前线程所执行的字节码的行号指示器。在虚拟机的概念模型里（仅是概念模型，各种虚拟机可能会通过一些更高效的方式去实现），字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。 
​	由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间的计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。 
​      如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Natvie方法，这个计数器值则为空（Undefined）。

**此内存区域是唯一一个在Java虚拟机规范中没有规定任何OutOfMemoryError情况的区域。**

#### **4. JVM栈（JVM Stacks）**

​	与程序计数器一样，Java虚拟机栈（Java Virtual Machine Stacks）也是线程私有的，它的生命周期与线程相同。虚拟机栈描述的是Java方法执行的内存模型：每个方法被执行的时候都会同时创建一个栈帧（Stack Frame）用于存储**局部变量表、操作栈、动态链接、方法出口**等信息。**每一个方法被调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程**。 

​	局部变量表存放了编译期可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，它不等同于对象本身，根据不同的虚拟机实现，它可能是一个指向对象起始地址的引用指针，也可能指向一个代表对象的句柄或者其他与此对象相关的位置）和returnAddress类型（指向了一条字节码指令的地址）。

​	其中64位长度的long和double类型的数据会占用2个局部变量空间（Slot），其余的数据类型只占用1个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

​	在Java虚拟机规范中，对这个区域规定了两种异常状况：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出**StackOverflowError**异常；如果虚拟机栈可以动态扩展（当前大部分的Java虚拟机都可动态扩展，只不过Java虚拟机规范中也允许固定长度的虚拟机栈），当扩展时无法申请到足够的内存时会抛出**OutOfMemoryError**异常。

#### **5. 本地方法栈（Native Method Stacks）**

​	本地方法栈（Native Method Stacks）与虚拟机栈所发挥的作用是非常相似的，其区别不过是虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而**本地方法栈则是为虚拟机使用到的Native方法服务**。虚拟机规范中对本地方法栈中的方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机（譬如Sun HotSpot虚拟机）直接就把本地方法栈和虚拟机栈合二为一。与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。



### **对象分配规则**

- 对象优先分配在Eden区，如果Eden区没有足够的空间时，虚拟机执行一次**Minor GC**。
- 大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在Eden区和两个Survivor区之间发生大量的内存拷贝（新生代采用复制算法收集内存）。
- 长期存活的对象进入老年代。虚拟机为每个对象定义了一个年龄计数器，如果对象经过了1次Minor GC那么对象会进入Survivor区，之后每经过一次Minor GC那么对象的年龄加1，知道达到阀值对象进入老年区。
- 动态判断对象的年龄。如果Survivor区中相同年龄的所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象可以直接进入老年代。
- 空间分配担保。每次进行Minor GC时，JVM会计算Survivor区移至老年区的对象的平均大小，如果这个值大于老年区的剩余值大小则进行一次**Full GC**，如果小于检查HandlePromotionFailure设置，如果true则只进行Monitor GC,如果false则进行Full GC。 

## 三. GC垃圾回收

### **1. 概述**

​	垃圾收集 Garbage Collection 通常被称为“GC”，它诞生于1960年 MIT 的 Lisp 语言，经过半个多世纪，目前已经十分成熟了。

​	jvm 中，程序计数器、虚拟机栈、本地方法栈都是随线程而生随线程而灭，栈帧随着方法的进入和退出做入栈和出栈操作，实现了自动的内存清理，因此，我们的内存垃圾回收**主要集中于 java 堆和方法区中**，在程序运行期间，这部分内存的分配和使用都是动态的.

 

### **2. 对象存活判断**

判断对象是否存活一般有两种方式：

**引用计数**：每个对象有一个引用计数属性，新增一个引用时计数加1，引用释放时计数减1，计数为0时可以回收。此方法简单，**无法解决对象相互循环引用**的问题。

**可达性分析**（Reachability Analysis）：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。不可达对象。

```
在Java语言中，GC Roots包括：

  虚拟机栈中引用的对象。

  方法区中类静态属性实体引用的对象。

  方法区中常量引用的对象。

  本地方法栈中JNI引用的对象。
```

### **3. 垃圾收集算法**

#### **标记 -清除算法**

 	 “**标记-清除**”（Mark-Sweep）算法，如它的名字一样，算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收掉所有被标记的对象。之所以说它是最基础的收集算法，是因为后续的收集算法都是基于这种思路并对其缺点进行改进而得到的。

​	它的主要缺点有两个：一个是效率问题，标记和清除过程的效率都不高；另外一个是空间问题，标记清除之后会产生大量不连续的内存碎片，**空间碎片太多**可能会导致，当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作。

[![wpsA73E.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174234266-1575111287.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174233406-219010809.png)

 

#### **复制算法**

​	“复制”（Copying）的收集算法，它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。

​	这样使得每次都是对其中的一块进行内存回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可，实现简单，运行高效。只是这种算法的代价是将内存缩小为原来的一半，持续复制长生存期的对象则导致效率降低。

[![wps9D31.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174237703-328169343.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174235125-36263316.png)

 

#### **标记-整理算法**

​	复制收集算法在对象存活率较高时就要执行较多的复制操作，效率将会变低。更关键的是，如果不想浪费50%的空间，就需要有额外的空间进行分配担保，以应对被使用的内存中所有对象都100%存活的极端情况，所以在老年代一般不能直接选用这种算法。

​	根据老年代的特点，有人提出了另外一种“标记-整理”（Mark-Compact）算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存

[![wps3952.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174239219-1187241876.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174238516-890282785.png)

 

#### **分代收集算法**

**GC分代的基本假设：绝大部分对象的生命周期都非常短暂，存活时间短。**

​	“分代收集”（Generational Collection）算法，把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用**复制算法**，只需要付出少量存活对象的复制成本就可以完成收集。而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用**“标记-清理”或“标记-整理”**算法来进行回收。

 

### **4. 垃圾收集器**

​     如果说收集算法是内存回收的方法论，垃圾收集器就是内存回收的具体实现

 

#### **Serial收集器**

​	串行收集器是最古老，最稳定以及效率高的收集器，可能会产生较长的停顿，只使用**一个线程**去回收。新生代、老年代使用**串行**回收；**新生代复制算法**、**老年代标记-压缩**；垃圾收集的过程中会Stop The World（服务暂停）

参数控制：**-XX:+UseSerialGC**   串行收集器

 

[![wpsA77.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174240391-864339858.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174239719-1127409322.png)

 

 

#### **ParNew收集器**

ParNew收集器其实就是Serial收集器的多线程版本。**新生代并行，老年代串行；**新生代**复制**算法、老年代**标记-压缩**

参数控制：**-XX:+UseParNewGC  **ParNew收集器

​		**-XX:ParallelGCThreads **限制线程数量

 

[![wps6A83.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174244797-1533472044.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174241188-1502278645.png)

#### **Parallel收集器**

​	Parallel Scavenge收集器类似ParNew收集器，Parallel收集器更关注系统的**吞吐量**。可以通过参数来打开**自适应调节策略，**虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或最大的吞吐量；也可以通过参数控制GC的时间不大于多少毫秒或者比例；新生代复制算法、老年代标记-压缩

​	参数控制：**-XX:+UseParallelGC  **使用Parallel收集器+ 老年代串行



#### **Parallel  Old收集器**

​	Parallel Old是Parallel Scavenge收集器的老年代版本，使用多线程和“标记－整理”算法。这个收集器是在JDK 1.6中才开始提供

​	参数控制： **-XX:+UseParallelOldGC** 	使用Parallel收集器+ 老年代并行

 

#### **CMS 收集器**

​	CMS（Concurrent Mark Sweep）收集器是一种以获取**最短回收停顿时间**为目标的收集器。目前很大一部分的Java应用都集中在互联网站或B/S系统的服务端上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户带来较好的体验。

​	从名字（包含“Mark Sweep”）上就可以看出CMS收集器是基于**“标记-清除”**算法实现的，它的运作过程相对于前面几种收集器来说要更复杂一些，整个过程分为4个步骤，包括： 

1. 初始标记（CMS initial mark）
2. 并发标记（CMS concurrent mark）
3. 重新标记（CMS remark）
4. 并发清除（CMS concurrent sweep）

 

​	其中初始标记、重新标记这两个步骤仍然需要“Stop The World”。初始标记仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，并发标记阶段就是进行GC Roots Tracing的过程，而重新标记阶段则是为了修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。 
​      由于整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，所以总体上来说，CMS收集器的内存回收过程是与用户线程一起并发地执行。**老年代收集器**（新生代使用ParNew）

  **优点: 并发收集、低停顿**

 **缺点：产生大量空间碎片、并发阶段会降低吞吐量**

   参数控制：**-XX:+UseConcMarkSweepGC  **使用CMS收集器

​		**-XX:+ UseCMSCompactAtFullCollection**  Full GC后，进行一次碎片整理；整理过程是独占的，会引起停顿时间变长

​		**-XX:+CMSFullGCsBeforeCompaction**  设置进行几次Full GC后，进行一次碎片整理

​            	-**XX:ParallelCMSThreads **设定CMS的线程数量（一般情况约等于可用CPU数量）

[![wpsCA6E.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174246485-1027326741.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174245500-1388590060.png)



#### **G1收集器**

​	G1是目前技术发展的最前沿成果之一，HotSpot开发团队赋予它的使命是未来可以替换掉JDK1.5中发布的CMS收集器。与CMS收集器相比G1收集器有以下特点：

1. **空间整合**，G1收集器采用**标记-整理**算法，不会产生内存空间碎片。分配大对象时不会因为无法找到连续空间而提前触发下一次GC。


2. **可预测停顿**，这是G1的另一大优势，降低停顿时间是G1和CMS的共同关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为N毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒，这几乎已经是实时Java（RTSJ）的垃圾收集器的特征了。

上面提到的垃圾收集器，收集的范围都是整个新生代或者老年代，而G1不再是这样。使用G1收集器时，Java堆的内存布局与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的**独立区域（Region）**，虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔阂了，它们都是一部分（可以不连续）Region的集合。

[![wps3B4C.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174247641-1572774731.jpg)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174247125-2073079982.jpg)

 

​	G1的新生代收集跟ParNew类似，当新生代占用达到一定比例的时候，开始出发收集。和CMS类似，G1收集器收集老年代对象会有短暂停顿。

**收集步骤**：

1、**标记阶段**，首先初始标记(Initial-Mark),这个阶段是停顿的(Stop the World Event)，并且会触发一次普通Mintor GC。对应GC log:GC pause (young) (inital-mark)

2、**Root Region Scanning**，程序运行过程中会回收survivor区(存活到老年代)，这一过程必须在young GC之前完成。

3、**Concurrent Marking**，在整个堆中进行并发标记(和应用程序并发执行)，此过程可能被young GC中断。在并发标记阶段，若发现区域对象中的所有对象都是垃圾，那个这个区域会被立即回收(图中打X)。同时，并发标记过程中，会计算每个区域的对象活性(区域中存活对象的比例)。

[![wps93E7.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174249203-774809988.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174248703-2128319077.png)

4、**Remark**, 再标记，会有短暂停顿(STW)。再标记阶段是用来收集 并发标记阶段 产生新的垃圾(并发阶段和应用程序一同运行)；G1中采用了比CMS更快的初始快照算法:snapshot-at-the-beginning (SATB)。

5、**Copy/Clean up**，多线程清除失活对象，会有STW。G1将回收区域的存活对象拷贝到新区域，清除Remember Sets，并发清空回收区域并把它返回到空闲区域链表中。

[![wps47EC.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174250656-1729618538.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174249938-1165331497.png)

6、复制/清除过程后。回收区域的活性对象已经被集中回收到深蓝色和深绿色区域。

[![wpsEAB1.tmp](https://images2015.cnblogs.com/blog/331425/201606/331425-20160624174252078-703896755.png)](http://images2015.cnblogs.com/blog/331425/201606/331425-20160624174251406-2060589648.png)



#### G1垃圾收集器

​	G1（Garbage-First）是在JDK 7u4版本之后发布的垃圾收集器，并在jdk9中成为默认垃圾收集器。通过“-XX:+UseG1GC”启动参数即可指定使用G1 GC。从整体来说，G1也是利用多CPU来缩短stop the world时间，并且是高效的并发垃圾收集器。但是G1不再像上文所述的垃圾收集器，需要分代配合不同的垃圾收集器，因为G1中的垃圾收集区域是“分区”（Region）的。G1的分代收集和以上垃圾收集器不同的就是除了有年轻代的ygc，全堆扫描的fullgc外，还有包含所有年轻代以及部分老年代Region的MixedGC。G1的优势还有可以通过调整参数，指定垃圾收集的最大允许pause time。下面会详细阐述下G1分区以及分代的概念，以及G1 GC的几种收集过程的分类。

**G1分区的概念**
​	在G1之前的垃圾收集器，将堆区主要划分了Eden区，Old区，Survivor区。其中对于Eden，Survivor对回收过程来说叫做“年轻代垃圾收集”。并且年轻代和老年代都分别是连续的内存空间。 
​	G1将堆分成了若干**Region**,以下和”分区”代表同一概念。Region的大小可以通过G1HeapRegionSize参数进行设置，其必须是2的幂，范围允许为1Mb到32Mb。 JVM的会基于堆内存的初始值和最大值的平均数计算分区的尺寸，平均的堆尺寸会分出约2000个Region。分区大小一旦设置，则启动之后不会再变化。如下图简单画了下G1分区模型。 

![G1垃圾收集器分区图](https://img-blog.csdn.net/20180528172927336?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpamluZ3lhbzgyMDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

Eden regions(年轻代-Eden区)
Survivor regions(年轻代-Survivor区)
Old regions（老年代）
Humongous regions（巨型对象区域）
Free resgions（未分配区域，也会叫做可用分区）-上图中空白的区域

关于分区有几个重要的概念：

- G1还是采用**分代回收**，但是不同的分代之间内存不一定是连续的，不同分代的Region的占用数也不一定是固定的（不建议通过相关选项显式设置年轻代大小。会覆盖暂停时间目标。）。年轻代的Eden,Survivor数量会随着每一次GC发生相应的改变。
- 分区是不固定属于哪个分代的，所以比如一次ygc过后，原来的Eden的分区就会变成空闲的可用分区，随后也可能被用作分配巨型对象，成为H区等。
- G1中的**巨型对象**是指，占用了Region容量的50%以上的一个对象。Humongous区，就专门用来存储巨型对象。如果一个H区装不下一个巨型对象，则会通过连续的若干H分区来存储。因为巨型对象的转移会影响GC效率，所以并发标记阶段发现巨型对象不再存活时，会将其直接回收。ygc也会在某些情况下对巨型对象进行回收。
- 通过上图可以看出，分区可以有效利用内存空间，因为收集整体是使用“**标记-整理**”，Region之间基于“复制”算法，GC后会将存活对象复制到可用分区（未分配的分区），所以不会产生空间碎片。
- G1类似CMS，也会在比如一次fullgc中基于堆尺寸的计算重新调整（增加）堆的空间。但是相较于执行fullgc，G1 GC会在无法分配对象或者巨型对象无法获得连续分区来分配空间时，**优先尝试扩展堆空间**来获得更多的可用分区。原则上就是G1会计算执行GC的时间，并且极力减少花在GC上的时间（包括ygc,mixgc）,如果可能，会通过不断扩展堆空间来满足对象分配、转移的需要。
- 因为G1提供了“**可预测的暂停时间**”，也是基于G1的启发式算法，所以G1会估算年轻代需要多少分区，以及还有多少分区要被回收。ygc触发的契机就是在Eden分区数量达到上限时。一次ygc会回收所有的Eden和survivor区。其中存活的对象会被转移到另一个新的survivor区或者old区，如果转移的目标分区满了，会再将可用区标记成S或者O区。

### 5. 常用的收集器组合

|      | **新生代GC策略**       | **年老代GC策略**    | **说明**                                   |
| ---- | ----------------- | -------------- | ---------------------------------------- |
| 组合1  | Serial            | Serial Old     | Serial和Serial Old都是单线程进行GC，特点就是GC时暂停所有应用线程。 |
| 组合2  | Serial            | CMS+Serial Old | CMS（Concurrent Mark Sweep）是并发GC，实现GC线程和应用线程并发工作，不需要暂停所有应用线程。另外，当CMS进行GC失败时，会自动使用Serial Old策略进行GC。 |
| 组合3  | ParNew            | CMS            | 使用-XX:+UseParNewGC选项来开启。ParNew是Serial的并行版本，可以指定GC线程数，默认GC线程数为CPU的数量。可以使用-XX:ParallelGCThreads选项指定GC的线程数。如果指定了选项-XX:+UseConcMarkSweepGC选项，则新生代默认使用ParNew GC策略。 |
| 组合4  | ParNew            | Serial Old     | 使用-XX:+UseParNewGC选项来开启。新生代使用ParNew GC策略，年老代默认使用Serial Old GC策略。 |
| 组合5  | Parallel Scavenge | Serial Old     | Parallel Scavenge策略主要是关注一个可控的吞吐量：应用程序运行时间 / (应用程序运行时间 + GC时间)，可见这会使得CPU的利用率尽可能的高，适用于后台持久运行的应用程序，而不适用于交互较多的应用程序。 |
| 组合6  | Parallel Scavenge | Parallel Old   | Parallel Old是Serial Old的并行版本             |
| 组合7  | G1GC              | G1GC           | -XX:+UnlockExperimentalVMOptions -XX:+UseG1GC        #开启-XX:MaxGCPauseMillis =50                  #暂停时间目标-XX:GCPauseIntervalMillis =200          #暂停间隔目标-XX:+G1YoungGenSize=512m            #年轻代大小-XX:SurvivorRatio=6                            #幸存区比例 |



## 四. Jvm 调优-命令篇

​	运用jvm自带的命令可以方便的在生产监控和打印堆栈的日志信息帮忙我们来定位问题！虽然jvm调优成熟的工具已经有很多：**jconsole**、大名鼎鼎的**VisualVM**，IBM的**Memory Analyzer**等等，但是在生产环境出现问题的时候，一方面工具的使用会有所限制，另一方面喜欢装X的我们，总喜欢在出现问题的时候在终端输入一些命令来解决。所有的工具几乎都是依赖于jdk的接口和底层的这些命令，研究这些命令的使用也让我们更能了解jvm构成和特性。

Sun JDK监控和故障处理命令有**jps jstat jmap jhat jstack jinfo**下面做一一介绍

#### 1. jps

​	JVM Process Status Tool,显示指定系统内所有的HotSpot**虚拟机进程**。

**命令格式**

```
jps [options] [hostid]

```

**option参数**

> - -l : 输出主类全名或jar路径
> - -q : 只输出LVMID
> - -m : 输出JVM启动时传递给main()的参数
> - -v : 输出JVM启动时显示指定的JVM参数

其中[option]、[hostid]参数也可以不写。

**示例**

```
$ jps -l -m
  28920 org.apache.catalina.startup.Bootstrap start
  11589 org.apache.catalina.startup.Bootstrap start
  25816 sun.tools.jps.Jps -l -m
```



#### 2. jstat

​	jstat(JVM statistics Monitoring)是用于监视虚拟机**运行时状态**信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

**命令格式**

```
jstat [option] LVMID [interval] [count]

```

**参数**

> - [option] : 操作参数
> - LVMID : 本地虚拟机进程ID
> - [interval] : 连续输出的时间间隔
> - [count] : 连续输出的次数

**option 参数总览**

| Option   | Displays…                                |
| -------- | ---------------------------------------- |
| class    | class loader的行为统计。Statistics on the behavior of the class loader. |
| compiler | HotSpt JIT编译器行为统计。Statistics of the behavior of the HotSpot Just-in-Time compiler. |
| gc       | 垃圾回收堆的行为统计。Statistics of the behavior of the garbage collected heap. |
| ......   | ......                                   |



#### 3. jmap

​	jmap(JVM Memory Map)命令用于**生成heap dump文件**，如果不使用这个命令，还阔以使用-XX:+HeapDumpOnOutOfMemoryError参数来让虚拟机出现OOM的时候·自动生成dump文件。 jmap不仅能生成dump文件，还阔以查询finalize执行队列、Java堆和永久代的详细信息，如当前使用率、当前使用的是哪种收集器等。

**命令格式**

```
jmap [option] LVMID
```

**option参数**

> - dump : 生成堆转储快照
> - finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
> - heap : 显示Java堆详细信息
> - histo : 显示堆中对象的统计信息
> - permstat : to print permanent generation statistics
> - F : 当-dump没有响应时，强制生成dump快照



#### 4. jhat

jhat(JVM Heap Analysis Tool)命令是与jmap搭配使用，用来**分析jmap生成的dump**，jhat内置了一个微型的HTTP/HTML服务器，生成dump的分析结果后，可以在浏览器中查看。在此要注意，一般不会直接在服务器上进行分析，因为jhat是一个耗时并且耗费硬件资源的过程，一般把服务器生成的dump文件复制到本地或其他机器上进行分析。

**命令格式**

```
jhat [dumpfile]

```

**参数**

> - -stack false|true 关闭对象分配调用栈跟踪(tracking object allocation call stack)。 如果分配位置信息在堆转储中不可用. 则必须将此标志设置为 false. 默认值为 true.>
> - -refs false|true 关闭对象引用跟踪(tracking of references to objects)。 默认值为 true. 默认情况下, 返回的指针是指向其他特定对象的对象,如反向链接或输入引用(referrers or incoming references), 会统计/计算堆中的所有对象。>
> - -port port-number 设置 jhat HTTP server 的端口号. 默认值 7000.>
> - -exclude exclude-file 指定对象查询时需要排除的数据成员列表文件(a file that lists data members that should be excluded from the reachable objects query)。 例如, 如果文件列列出了 java.lang.String.value , 那么当从某个特定对象 Object o 计算可达的对象列表时, 引用路径涉及 java.lang.String.value 的都会被排除。>
> - -baseline exclude-file 指定一个基准堆转储(baseline heap dump)。 在两个 heap dumps 中有相同 object ID 的对象会被标记为不是新的(marked as not being new). 其他对象被标记为新的(new). 在比较两个不同的堆转储时很有用.>
> - -debug int 设置 debug 级别. 0 表示不输出调试信息。 值越大则表示输出更详细的 debug 信息.>
> - -version 启动后只显示版本信息就退出>
> - -J< flag > 因为 jhat 命令实际上会启动一个JVM来执行, 通过 -J 可以在启动JVM时传入一些启动参数. 例如, -J-Xmx512m 则指定运行 jhat 的Java虚拟机使用的最大堆内存为 512 MB. 如果需要使用多个JVM启动参数,则传入多个 -Jxxxxxx.



#### 5. jstack

​	jstack用于生成java虚拟机当前时刻的**线程快照**。线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

**命令格式**

```
jstack [option] LVMID
```

**option参数**

> - -F : 当正常输出请求不被响应时，强制输出线程堆栈
> - -l : 除堆栈外，显示关于锁的附加信息
> - -m : 如果调用到本地方法的话，可以显示C/C++的堆栈



#### 6. jinfo

jinfo(JVM Configuration info)这个命令作用是实时**查看和调整虚拟机运行参数**。 之前的jps -v口令只能查看到显示指定的参数，如果想要查看未被显示指定的参数的值就要使用jinfo口令

**命令格式**

```
jinfo [option] [args] LVMID
```

**option参数**

> - -flag : 输出指定args参数的值
> - -flags : 不需要args参数，输出所有JVM参数的值
> - -sysprops : 输出系统属性，等同于System.getProperties()