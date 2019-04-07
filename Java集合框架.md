

# 			Java集合框架

![](D:/java/java Collection/img/Java集合框架.png)

## 1 Collection基础

### 1 遍历Collection的几种方法

​	1) for-each语法

```java
Collection<Person> persons = new ArrayList<Person>();
for (Person person : persons) { 
    System.out.println(person.name);  
} 
```

​	2) 使用 Iterator 迭代器

```java
Collection<Person> persons = new ArrayList<Person>();
Iterator iterator = persons.iterator();
while (iterator.hasNext) { 
    System.out.println(iterator.next);  
} 
```

​	3) 使用 aggregate operations 聚合操作

```java
Collection<Person> persons = new ArrayList<Person>();
persons
    .stream()
    .forEach(new Consumer<Person>() {  
        @Override  
        public void accept(Person person) {  
            System.out.println(person.name);  
        }  
}); 
```

Aggregate Operations 聚合操作:

​	在 JDK 8 以后，推荐使用聚合操作对一个集合进行操作。聚合操作通常和 lambda 表达式结合使用，让代码看起来更简洁（因此可能更难理解）。下面举几个简单的栗子：

​	1.使用流来遍历一个 ShapesCollection，然后输出红色的元素：

```java
myShapesCollection.stream()
    .filter(e -> e.getColor() == Color.RED)
    .forEach(e -> System.out.println(e.getName()));
```

​	2.你还可以获取一个并行流（parallelStream），当集合元素很多时使用并发可以提高效率：

```java
myShapesCollection.parallelStream()
    .filter(e -> e.getColor() == Color.RED)
    .forEach(e -> System.out.println(e.getName()));      
```

​	3.聚合操作还有很多操作集合的方法，比如说你想把 Collection 中的元素都转成 String 对象，然后把它们 连起来：

```java
String joined = elements.stream()
    .map(Object::toString)
    .collect(Collectors.joining(", "));
```



**List 接口**
​	一个 List 是一个元素有序的、可以重复、可以为 null 的集合（有时候我们也叫它“序列”）。
​	Java 集合框架中最常使用的几种 List 实现类是 ArrayList，LinkedList 和 Vector。在各种 List 中，最好的做法是以 ArrayList 作为默认选择。 当插入、删除频繁时，使用 LinkedList，Vector 总是比 ArrayList 慢，所以要尽量避免使用它。

**为什么 List 中的元素 “有序”、“可以重复”呢?**

​	首先，List 的数据结构就是一个序列，存储内容时直接在内存中开辟一块连续的空间，然后将空间地址与索引对应。

​	其次,List 接口的实现类在实现插入元素时，都会根据索引进行排列。

​	由于 List 的元素在存储时互不干扰，没有什么依赖关系，自然**可以重复**（这点与 Set 有很大区别）。

**List 中除了继承 Collection 的一些方法，还提供以下操作：**
​	位置相关：List 和 数组一样，都是从 0 开始，我们可以根据元素在 list 中的位置进行操作，比如说 get, set, add, addAll, remove;
​	搜索：从 list 中查找某个对象的位置，比如 indexOf, lastIndexOf;
​	迭代：使用 Iterator 的拓展版迭代器 ListIterator 进行迭代操作;

​	范围性操作：使用 subList 方法对 list 进行任意范围的操作。subList 返回的仍是 List 原来的引用，只不过把开始位置 offset 和 size 改了下

和 Set，Map 一样，List 中如果想要根据两个对象的内容而不是地址比较是否相等时，需要重写 `equals()` 和 `hashCode()` 方法。 `remove()`, `contains()`, `indexOf()` 等等方法都需要依赖它们.



### 2 List 与 Array 区别？

List 在很多方面跟 Array 数组感觉很相似，尤其是 ArrayList，那 List 和数组究竟哪个更好呢？
​	相似之处： 
​		都可以表示一组同类型的对象
​		都使用下标进行索引
​	不同之处： 
​		数组可以存任何类型元素
​		List 不可以存基本数据类型，必须要包装
​		数组容量固定不可改变；List 容量可动态增长
​		数组效率高; List 由于要维护额外内容，效率相对低一些
​		容量固定时优先使用数组，容纳类型更多，更高效。

在容量不确定的情景下， List 更有优势，看下 ArrayList 和 LinkedList 如何实现容量动态增长：

ArrayList扩容: 默认初始大小为10,扩容为1.5倍



### 3 List 与 Array 之间的转换

在 List 中有两个转换成 数组 的方法：
​	Object[] toArray() 
​		返回一个包含 List 中所有元素的数组；
​	T[] toArray(T[] array) 

​		作用同上，不同的是当 参数 array 的长度比 List 的元素大时，会使用参数 array 保存 List 中的元素；否则会创建一个新的 数组存放 List 中的所有元素；



### 4 迭代器 Iterator, ListIterator

List 继承了 Collection 的 iterator() 方法，可以获取 Iterator，使用它可以进行向后遍历。
​	在此基础上，List 还可以通过 listIterator(), listIterator(int location) 方法（后者指定了游标的位置）获取更强大的迭代器 ListIterator。
​	使用 ListIterator 可以对 List 进行向前、向后双向遍历，同时还允许进行 add, set, remove 等操作。

​	List 的实现类中许多方法都使用了 ListIterator，比如 List.indexOf() 方法的一种实现：

```java
public int indexOf(E e) {
    for (ListIterator<E> it = listIterator(); it.hasNext(); )
        if (e == null ? it.next() == null : e.equals(it.next()))
            return it.previousIndex();
    // Element not found
    return -1;
}
```



### 5 List 的相关算法：

集合的工具类 Collections 中包含很多 List 的相关操作算法：

- sort ，归并排序
- shuffle ，随机打乱
- reverse ，反转元素顺序
- swap ，交换
- binarySearch ，二分查找
- ……



## 2 ArrayList

ArrayList 是 Java 集合框架中 [List接口](http://blog.csdn.net/u011240877/article/details/52802849) 的一个实现类。

可以说 ArrayList 是我们使用最多的 List 集合，它有以下特点：

1. 容量不固定，想放多少放多少（当然有最大阈值，但一般达不到）


2. 有序的（元素输出顺序与输入顺序一致）
3. 元素可以为 null
4. 效率高 

  ​	1. size(), isEmpty(), get(), set() iterator(), ListIterator() 方法的时间复杂度都是 O(1)

  ​	2. add() 添加操作的时间复杂度平均为 O(n)

  ​	3. 其他所有操作的时间复杂度几乎都是 O(n)

5. 占用空间更小: 对比 LinkedList，不用占用额外空间维护链表结构



### 1.底层数据结构，数组：

```java
transient Object[] elementData;
```

由于数组类型为 Object，所以允许添加 null 。 
transient 说明这个数组无法序列化。 
初始时为 DEFAULTCAPACITY_EMPTY_ELEMENTDATA 。

### 2.默认的空数组：

```java
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

private static final Object[] EMPTY_ELEMENTDATA = {};
```

### 3.数组初始容量为 10：

```java
private static final int DEFAULT_CAPACITY = 10;
```

### 4.数组中当前元素个数：

```java
private int size; //size <= capacity
```

### 5.数组最大容量：

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

一些虚拟器需要在数组前加个 头标签，所以减去 8 。 
当想要分配比 MAX_ARRAY_SIZE 大的个数就会报 `OutOfMemoryError`。

### 6.ArrayList 的关键方法

#### 1.构造函数

ArrayList 有三种构造函数：

1. 初始为空数组
2. 根据指定容量，创建个对象数组
3. 直接创建和指定集合一样内容的 ArrayList

```java
//初始为空数组
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

//根据指定容量，创建个对象数组
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

//直接创建和指定集合一样内容的 ArrayList
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray 有可能不返回一个 Object 数组
        if (elementData.getClass() != Object[].class)
            //使用 Arrays.copy 方法拷创建一个 Object 数组
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

#### 2.添加元素： (System.arraycopy 是底层方法,数组复制)

1. 在指定位置添加一个元素;

2. 添加一个集合;

3. 在指定位置，添加一个集合

   ​

```java
public boolean add(E e) {
    //对数组的容量进行调整
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

//在指定位置添加一个元素
public void add(int index, E element) {
    rangeCheckForAdd(index);

    //对数组的容量进行调整
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //整体后移一位，效率不太好啊
    System.arraycopy(elementData, index, elementData, index + 1,
                     size - index);
    elementData[index] = element;
    size++;
}


//添加一个集合
public boolean addAll(Collection<? extends E> c) {
    //把该集合转为对象数组
    Object[] a = c.toArray();
    int numNew = a.length;
    //增加容量
    ensureCapacityInternal(size + numNew);  // Increments modCount
    //挨个向后迁移
    System.arraycopy(a, 0, elementData, size, numNew);
    size += numNew;
    //新数组有元素，就返回 true
    return numNew != 0;
}

//在指定位置，添加一个集合
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    ensureCapacityInternal(size + numNew);  // Increments modCount

    int numMoved = size - index;
    //原来的数组挨个向后迁移
    if (numMoved > 0)
        System.arraycopy(elementData, index, elementData, index + numNew,
                         numMoved);
    //把新的集合数组 添加到指定位置
    System.arraycopy(a, 0, elementData, index, numNew);
    size += numNew;
    return numNew != 0;
}
```

#### 3.对数组的容量进行调整：

```java
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // 不是默认的数组，说明已经添加了元素
        ? 0
        // 默认的容量
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        //当前元素个数比默认容量大
        ensureExplicitCapacity(minCapacity);
    }
}

private void ensureCapacityInternal(int minCapacity) {
    //还没有添加元素
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        //最小容量取默认容量和 当前元素个数 最大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // 容量不够了，需要扩容
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

​	我们可以主动调用 ensureCapcity 来增加 ArrayList 对象的容量，这样就避免添加元素满了时扩容、挨个复制后移等消耗。

#### 4.扩容1.5 倍：

```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    // 1.5 倍 原来容量
    int newCapacity = oldCapacity + (oldCapacity >> 1);

    //如果所需容量超过原容量的1.5倍,则使用所需容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;

    //新的容量居然超出了 MAX_ARRAY_SIZE
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        //最大容量可以是 Integer.MAX_VALUE
        newCapacity = hugeCapacity(minCapacity);

    // minCapacity 一般跟元素个数 size 很接近，所以新建的数组容量为 newCapacity 更宽松些
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

#### 5.查询，修改等操作，直接根据角标对数组操作，都很快：

```java
E elementData(int index) {
    return (E) elementData[index];
}

//获取
public E get(int index) {
    rangeCheck(index);
    //直接根据数组角标返回元素，快的一比
    return elementData(index);
}

//修改
public E set(int index, E element) {
    rangeCheck(index);
    E oldValue = elementData(index);

    //直接对数组操作
    elementData[index] = element;
    //返回原来的值
    return oldValue;
}
```

#### 6.删除，还是有点慢：

1. 根据位置删除;
2. 删除某个元素;
3. 内部方法，“快速删除”，就是把重复的代码移到一个方法里;
4. 保留公共的;
5. 删除或者保留指定集合中的元素;
6. 清除全部;



```java
//根据位置删除
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    //挨个往前移一位
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //原数组中最后一个元素删掉
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}

//删除某个元素
public boolean remove(Object o) {
    if (o == null) {
        //挨个遍历找到目标
        for (int index = 0; index < size; index++)
            if (elementData[index] == null) {
                //快速删除
                fastRemove(index);
                return true;
            }
    } else {
        for (int index = 0; index < size; index++)
            if (o.equals(elementData[index])) {
                fastRemove(index);
                return true;
            }
    }
    return false;
}

//内部方法，“快速删除”，就是把重复的代码移到一个方法里
//没看出来比其他 remove 哪儿快了 - -
private void fastRemove(int index) {
    modCount++;
    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work
}

//保留公共的
public boolean retainAll(Collection<?> c) {
    Objects.requireNonNull(c);
    return batchRemove(c, true);
}

//删除或者保留指定集合中的元素
private boolean batchRemove(Collection<?> c, boolean complement) {
    final Object[] elementData = this.elementData;
    //使用两个变量，一个负责向后扫描，一个从 0 开始，等待覆盖操作
    int r = 0, w = 0;
    boolean modified = false;
    try {
        //遍历 ArrayList 集合
        for (; r < size; r++)
            //如果指定集合中是否有这个元素，根据 complement 判断是否往前覆盖删除
            if (c.contains(elementData[r]) == complement)
                elementData[w++] = elementData[r];
    } finally {
        //发生了异常，直接把 r 后面的复制到 w 后面
        if (r != size) {
            System.arraycopy(elementData, r,
                             elementData, w,
                             size - r);
            w += size - r;
        }
        if (w != size) {
            // 清除多余的元素，clear to let GC do its work
            for (int i = w; i < size; i++)
                elementData[i] = null;
            modCount += size - w;
            size = w;
            modified = true;
        }
    }
    return modified;
}

//清除全部
public void clear() {
    modCount++;
    //并没有直接使数组指向 null,而是逐个把元素置为空
    //下次使用时就不用重新 new 了
    for (int i = 0; i < size; i++)
        elementData[i] = null;

    size = 0;
}
```

#### 7.判断状态：

```java
public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

//遍历，第一次找到就返回
public int indexOf(Object o) {
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}

//倒着遍历
public int lastIndexOf(Object o) {
    if (o == null) {
        for (int i = size-1; i >= 0; i--)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = size-1; i >= 0; i--)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}
```

#### 8.转换成 数组：

```java
public Object[] toArray() {
    return Arrays.copyOf(elementData, size);
}

public <T> T[] toArray(T[] a) {
    //a数组容量不够,则重新创建一个数组并复制返回
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    //全部元素拷贝到 数组 a
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

看下 Arrays.copyOf() 方法：

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
    @SuppressWarnings("unchecked")
    T[] copy = ((Object)newType == (Object)Object[].class)
        ? (T[]) new Object[newLength]
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

如果 newType 是一个对象对组，就直接把 original 的元素拷贝到 对象数组中； 
否则新建一个 newType 类型的数组。



在 Java 集合深入理解：AbstractList 中我们介绍了 RandomAccess，里面提到，支持 RandomAccess 的对象，遍历时**使用 get 比 迭代器更快**。(可能是迭代器需要维护更多东西,比如上一次访问的索引,删除时为-1)

​	**ArrayList用for循环遍历比iterator迭代器遍历快，LinkedList用iterator迭代器遍历比for循环遍历快**

由于 ArrayList 继承自 RandomAccess， 而且它的迭代器都是基于 ArrayList 的方法和数组直接操作，所以遍历时 get 的效率要 >= 迭代器。

另外，由于 ArrayList 不是同步的，所以在并发访问时，如果在迭代的同时有其他线程修改了 ArrayList, fail-fast 的迭代器 Iterator/ListIterator 会报 ConcurrentModificationException 错。

因此我们在并发环境下需要外部给 ArrayList 加个同步锁，或者直接在初始化时用 Collections.synchronizedList 方法进行包装：

```java
List list = Collections.synchronizedList(new ArrayList(...));
```





## 3 Queue

​	Java 集合中的 Queue 继承自 Collection 接口 ，Deque, LinkedList, PriorityQueue, BlockingQueue 等类都实现了它。
​	Queue 用来存放 等待处理元素 的集合，这种场景一般用于缓冲、并发访问。

​	除了继承 Collection 接口的一些方法，Queue 还添加了额外的 添加、删除、查询操作。

添加、删除、查询这些个操作都提供了两种形式，其中一种在操作失败时直接抛出异常，而另一种则返回一个特殊的值：

![Queue](https://img-blog.csdn.net/20161019111111582)

### Queue 方法介绍：

### 1.add(E), offer(E) 在尾部添加:

```java
boolean add(E e);

boolean offer(E e);
```

他们的共同之处是**建议实现类**禁止添加 null 元素，否则会报空指针 NullPointerException；

不同之处在于 add() 方法在添加失败（比如队列已满）时会报 一些运行时错误 错；而 offer() 方法即使在添加失败时也不会奔溃，只会返回 false。

**注意**
​	Queue 是个接口，它提供的 add, offer 方法初衷是希望子类能够禁止添加元素为 null，这样可以避免在查询时返回 null 究竟是正确还是错误。
​	事实上大多数 Queue 的实现类的确响应了 Queue 接口的规定，比如 ArrayBlockingQueue，PriorityBlockingQueue 等等。

但还是有一些实现类没有这样要求，比如 **LinkedList**。

### 2.remove(), poll() 删除并返回头部：

```java
E remove();

E poll();
```

当队列为空时 remove() 方法会报 NoSuchElementException 错; 而 poll() 不会奔溃，只会返回 null。



### 3.element(), peek() 获取但不删除：

```java
E element();

E peek();
```

当队列为空时 element() 抛出异常；peek() 不会奔溃，只会返回 null。

### 其他

​	1.虽然 LinkedList 没有禁止添加 null，但是一般情况下 Queue 的实现类都不允许添加 null 元素，为啥呢？因为 poll(), peek() 方法在异常的时候会返回 null，你添加了 null　以后，当获取时不好分辨究竟是否正确返回。
​	2.Queue 一般都是 FIFO 的，但是也有例外，比如优先队列 priority queue（它的顺序是根据自然排序或者自定义 comparator 的）；再比如 LIFO 的队列（跟栈一样，后来进去的先出去）。

​	不论进入、出去的先后顺序是怎样的，使用 remove()，poll() 方法操作的都是 头部 的元素；而插入的位置则不一定是在队尾了，不同的 queue 会有不同的插入逻辑。



## 4 Deque  *Double ended queue (双端队列)* 

Deque 继承自 Queue,直接实现了它的有 **LinkedList, ArayDeque, ConcurrentLinkedDeque** 等。
Deque 支持容量受限的双端队列，也支持大小不固定的。一般双端队列大小不确定。
Deque 接口定义了一些从头部和尾部访问元素的方法。比如分别在头部、尾部进行插入、删除、获取元素。

和 Queue类似，每个操作都有两种方法，一种在异常情况下直接抛出异常奔溃，另一种则不会抛异常，而是返回特殊的值，比如 false, null …

![Deque](https://img-blog.csdn.net/20161019193301571)

插入（Insert）方法的第二种是针对固定大小的双端队列设计的。大多数情况下 插入都不会失败。

Deque 继承了 Queue 接口的方法。当 Deque 当做 队列使用时（FIFO），添加元素是添加到队尾，删除时删除的是头部元素。从 Queue 接口继承的方法对应容器的方法如图所示：

![这里写图片描述](https://img-blog.csdn.net/20161019194500774)

Deque 也能当栈用（后进先出）。这时入栈、出栈元素都是在 双端队列的头部 进行。Deque 中和栈对应的方法如图所示：

![这里写图片描述](https://img-blog.csdn.net/20161019232902599)



### Deque 的实现类

Deque 的实现类主要分为两种场景：
一般场景 
​	LinkedList 大小可变的**链表**双端队列，允许元素为 null
​	ArrayDeque 大下可变的**数组**双端队列，不允许 null
并发场景 

​	LinkedBlockingDeque 如果队列为空时，获取操作将会**阻塞**，直到有元素添加

### Deque 与 工作密取

​	在并发编程 中，双端队列 Deque 还用于 “**工作密取**” 模式。
什么是工作密取呢？
​	在 **生产者-消费者** 模式中，所有消费者都从一个工作队列中取元素，一般使用**阻塞队列**实现；
​	而在 工作密取 模式中，每个消费者有其单独的工作队列，如果它完成了自己双端队列中的全部工作，那么它就可以从**其他消费者的双端队列末尾秘密地获取工作**。

​	工作密取 模式 对比传统的 生产者-消费者 模式，更为**灵活**，因为多个线程不会因为在同一个工作队列中抢占内容发生竞争。在大多数时候，它们只是访问自己的双端队列。即使需要访问另一个队列时,也是从队列的尾部获取工作，**降低了队列上的竞争程度**。



## 5 LinkedList

LinkedList 继承自 [AbstractSequentialList 接口](http://blog.csdn.net/u011240877/article/details/52854681)，同时了还实现了 [Deque](http://blog.csdn.net/u011240877/article/details/52865173), [Queue](http://blog.csdn.net/u011240877/article/details/52860924) 接口。

LinkedList 的成员变量只有三个：

- 头节点 first
- 尾节点 last
- 容量 size

节点是一个双向节点： 

```java
private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

### LinkedList 的方法

1.关键的几个内部方法（头部添加删除，尾部添加删除，获取指定节点，指定节点的添加删除）

2.公开的添加方法：

3.删除方法：

清除全部元素其实只需要把首尾都置为 null, 这个链表就已经是空的，因为无法访问元素。 
但是为了避免浪费空间，需要把中间节点都置为 null：

4.公开的修改方法，只有一个 set :

​	



### 迭代器

​	关键方法介绍完了，接下来是内部实现的迭代器，需要注意的是 LinkedList 实现了一个倒序迭代器 DescendingIterator；还实现了一个 ListIterator ，名叫 ListItr迭代器

​	1.DescendingIterator 倒序迭代器

```java
//很简单，就是游标直接在 迭代器尾部，然后颠倒黑白，说是向后遍历，实际是向前遍历
private class DescendingIterator implements Iterator<E> {
        private final ListItr itr = new ListItr(size());
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
    }
```

 2.  ListItr 操作基本都是调用的内部关键方法，没什么特别的

     ### Spliterator

```
Spliterator 是 Java 8 引入的新接口，顾名思义，Spliterator 可以理解为 Iterator 的 Split 版本（但用途要丰富很多）。

使用 Iterator 的时候，我们可以顺序地遍历容器中的元素，使用 Spliterator 的时候，我们可以将元素分割成多份，分别交于不于的线程去遍历，以提高效率。

使用 Spliterator 每次可以处理某个元素集合中的一个元素 — 不是从 Spliterator 中获取元素，而是使用 tryAdvance() 或 forEachRemaining() 方法对元素应用操作。

但 Spliterator 还可以用于估计其中保存的元素数量，而且还可以像细胞分裂一样变为一分为二。这些新增加的能力让流并行处理代码可以很方便地将工作分布到多个可用线程上完成。
```

### LinkedList 特点

- 双向链表实现
- 元素时有序的，输出顺序与输入顺序一致
- **允许元素为 null**
- **所有指定位置的操作都是从头开始遍历进行的**
- 和 ArrayList 一样，**不是同步容器**

### 并发访问注意事项

linkedList 和 ArrayList 一样，不是同步容器。所以需要外部做同步操作，或者直接用 `Collections.synchronizedList`方法包一下，最好在创建时就报一下：

```java
List list = Collections.synchronizedList(new LinkedList(...));
```

​	LinkedList 的迭代器都是 fail-fast 的: 如果在并发环境下，其他线程使用迭代器以外的方法修改数据，会导致 ConcurrentModificationException.

ArrayList

​	基于数组，在数组中搜索和读取数据是很快的。因此 ArrayList 获取数据的时间复杂度是O(1);
​	但是添加、删除时该元素后面的所有元素都要移动，所以添加/删除数据效率不高；
​	另外其实还是有容量的，每次达到阈值需要扩容，这个操作比较影响效率。
LinkedList

​	基于双端链表，添加/删除元素只会影响周围的两个节点，开销很低；
​	只能顺序遍历，无法按照索引获得元素，因此查询效率不高；
​	没有固定容量，不需要扩容；

​	需要更多的内存， LinkedList 每个节点中需要多存储前后节点的信息，占用空间更多些。



## 6 Vector

​	Vector 和 ArrayList 一样，都是继承自 [AbstractList](http://blog.csdn.net/u011240877/article/details/52834074)。它是 **Stack 的父类**。英文的意思是 “矢量”。

### 1. Vector简介

1.底层也是个数组

```
protected Object[] elementData;
```

2.数组元素个数，为啥不就叫 size 呢？奇怪

```
protected int elementCount;
```

3.扩容时增长数量，允许用户自己设置。如果这个值是 0 或者 负数，扩容时会扩大 2 倍，而不是 1.5

```java
protected int capacityIncrement;
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

4.默认容量:10



### 2. Vector 的成员方法

#### 1.先来看 JDK 7 中 Vector 的 3 种扩容方式：

1. 根据指定的容量进行扩容;

2. 默认增长一倍的扩容;

3. 指定默认扩容数量的扩容;

   ```java
   //根据指定的容量进行扩容   
   private void grow(int newCapacity) {
       //创建个指定容量的新数组，这里假设指定的容量比当前数组元素个数多
       E[] newData = newElementArray(newCapacity);
       //把当前数组复制到新创建的数组
       System.arraycopy(elementData, 0, newData, 0, elementCount);
       //当前数组指向新数组
       elementData = newData;
   }

   //默认增长一倍的扩容
   private void growByOne() {
       int adding = 0;
       //扩容量 capacityIncrement 不大于 0，就增长一倍
       if (capacityIncrement <= 0) {
           if ((adding = elementData.length) == 0) {
               adding = 1;
           }
       } else {
           //否则按扩容量走
           adding = capacityIncrement;
       }

       //创建个新数组，大小为当前容量加上 adding
       E[] newData = newElementArray(elementData.length + adding);
       //复制，赋值
       System.arraycopy(elementData, 0, newData, 0, elementCount);
       elementData = newData;
   }

   //指定默认扩容数量的扩容
   private void growBy(int required) {
       int adding = 0;
       //扩容量 capacityIncrement 不大于 0
       if (capacityIncrement <= 0) {
           //如果当前数组内没有元素，就按指定的数量扩容
           if ((adding = elementData.length) == 0) {
               adding = required;
           }
           //增加扩容数量到 指定的以上
           while (adding < required) {
               adding += adding;
           }
       } else {
           //扩容量大于 0 ，还是按指定的扩容数量走啊
           adding = (required / capacityIncrement) * capacityIncrement;
           //不过也可能出现偏差，因为是 int 做除法，所以扩容值至少是 指定扩容量的一倍以上
           if (adding < required) {
               adding += capacityIncrement;
           }
       }
       //创建，复制，赋值一条龙
       E[] newData = newElementArray(elementData.length + adding);
       System.arraycopy(elementData, 0, newData, 0, elementCount);
       elementData = newData;
   }
   ```

   #### 再来看JDK 8 中的扩容机制，变成一种了：

   ```java
   //扩容，传入最小容量，跟 ArrayList.grow(int) 很相似，只是扩大量不同 
   private void grow(int minCapacity) {
       int oldCapacity = elementData.length;
       //如果增长量 capacityIncrement 不大于 0 ，就扩容 2 倍
       int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                        capacityIncrement : oldCapacity);
       if (newCapacity - minCapacity < 0)
           newCapacity = minCapacity;
       if (newCapacity - MAX_ARRAY_SIZE > 0)
           newCapacity = hugeCapacity(minCapacity);
       //
       elementData = Arrays.copyOf(elementData, newCapacity);
   }

   private static int hugeCapacity(int minCapacity) {
       if (minCapacity < 0) // overflow
           throw new OutOfMemoryError();
       return (minCapacity > MAX_ARRAY_SIZE) ?
           Integer.MAX_VALUE :
           MAX_ARRAY_SIZE;
   }
   ```

   ### 3. Vector 中的迭代器

   普通迭代器　Iterator:

   ListIterator:

   Vector 还支持 Enumeration　迭代：

   ​

   ### 4. modCount字段(属于AbstractList类)的作用:

   ​	该字段表示list结构上被修改的次数。结构上的修改指的是那些改变了list的长度大小或者使得遍历过程中产生不正确的结果的其它方式。

   ​	该字段被Iterator以及ListIterator的实现类所使用，如果该值被意外更改，Iterator或者ListIterator 将抛出ConcurrentModificationException异常，

   ​	这是jdk在面对迭代遍历的时候为了避免不确定性而采取的快速失败原则。

   ​	子类对此字段的使用是可选的，如果子类希望支持快速失败，只需要覆盖该字段相关的所有方法即可。单线程调用不能添加删除terator正在遍历的对象，否则将可能抛出ConcurrentModificationException异常，如果子类不希望支持快速失败，该字段可以直接忽略。

### 3 总结

#### Vector 特点

​	底层由一个可以增长的数组组成
​	Vector 通过 capacity (容量) 和 capacityIncrement (增长数量) 来尽量少的占用空间
​	扩容时默认扩大两倍
​	最好在插入大量元素前增加 vector 容量，那样可以减少重新申请内存的次数
​	通过 iterator 和 lastIterator 获得的迭代器是 **fail-fast** 的
​	通过 elements 获得的老版迭代器 **Enumeration 不是 fail-fast** 的
​	同步类，每个方法前都有同步锁 synchronized
​	在 JDK 2.0 以后，经过优化，Vector 也加入了 Java 集合框架大家族

#### Vector VS ArrayList

共同点：

​	都是基于数组
​	都支持随机访问
​	默认容量都是 10
​	都有扩容机制
区别：

​	Vector 出生的比较早，JDK 1.0 就出生了，ArrayList JDK 1.2 才出来
​	Vector 比 ArrayList 多一种迭代器 Enumeration
​	Vector 是线程安全的，ArrayList 不是
​	Vector 默认扩容 2 倍，ArrayList 是 1.5

如果没有线程安全的需求，一般推荐使用 ArrayList，而不是 Vector，因为每次都要获取锁，效率太低。



## 7 Stack 栈

**栈通常有三种操作：**

- push 入栈
- pop 栈顶元素出栈，并返回
- peek 获取栈顶元素，并不删除

Java 集合框架中的 Stack 继承自 [Vector](http://blog.csdn.net/u011240877/article/details/52900893)：

- 由于 Vector 有 4 个构造函数，加上 Stack 本身的一种，也就是说有 5 种创建 Stack 的方法
- 跟 Vector 一样，它是 **数组实现的栈**。

### 1 简介

1.构造函数

```
//构建一个空栈
public Stack() {
}
```

2.入栈

```java
//调用的 Vector.addElement()
public E push(E item) {
    addElement(item);

    return item;
}
```

```java
// Vector 的 addElement() 方法，就是在数组尾部添加元素：
public synchronized void addElement(E obj) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = obj;
}
```

3.获取顶端元素，但不删除

```
public synchronized E peek() {
    //调用 Vector.size() 返回元素个数
    int     len = size();

    if (len == 0)
        throw new EmptyStackException();
    //调用 Vector.elementAt 得到栈顶元素
    return elementAt(len - 1);
}
```

4.出栈

```java
public synchronized E pop() {
    E       obj;
    int     len = size();

    //调用 peek() 获取顶端元素，一会儿返回
    obj = peek();
    //调用 Vector.removeElementAt 删除顶端元素
    removeElementAt(len - 1);

    return obj;
}
```

5.查找元素是否在栈中

```java
public synchronized int search(Object o) {
    int i = lastIndexOf(o);
    //返回的是栈顶到该元素出现的位置的距离
    if (i >= 0) {
        return size() - i;
    }
    return -1;
}
```

6.是否为空

```
public boolean empty() {
    return size() == 0;
}
```

Vector.size():

```
public synchronized int size() {
    return elementCount;
}
```



### 2. 总结

Java 集合框架中的 Stack 具有以下特点：

- 继承自 Vector
- 有 5 种创建 Stack 的方法
- 采用数组实现
- **除了 push()，剩下的方法都是同步的**



## 8 Map

#### 	1 简介

​	Java 中的 Map 接口 是和 [Collection 接口](http://blog.csdn.net/u011240877/article/details/52773577) 同一等级的集合根接口，它 表示一个**键值对 (key-value) 的映射**。类似数学中 **函数 **的概念。

Map 的三个 collection 视图：

![这里写图片描述](https://img-blog.csdn.net/20161025193401464)

Map 接口提供了三种角度来分析 Map:

- KeySet
- Values
- Entry

Map 接口提供了三种角度来分析 Map:

**KeySet**
**Values**
**Entry**

1.KeySet
​	KeySet 是一个 Map 中键（key）的集合，以 **Set** 的形式保存，**不允许重复**，因此键存储的对象需要**重写 equals() 和 hashCode() 方法。**

​	在上图就是保存 AA, BB, CC, DD… 等键的集合。

​	可以通过 Map.keySet() 方法获得。

2.Values
​	Values 是一个 Map 中值 (value) 的集合，以 Collection 的形式保存，因此可以重复。

​	在上图就是保存 90,90,56,78… 等值的集合。

​	通过 Map.values() 方法获得。

3.Entry
​	Entry 是 Map 接口中的静态内部接口，表示**一个键值对的映射**，例如上图中 AA-90 这一组映射关系。

Entry 具有以下方法：

​	getKey() , 获取这组映射中的键 key
​	getValue() , 获取这组映射中的值 value
​	setValue() , 修改这组映射中的值
​	hashCode() , 返回这个 Entry 的哈希值
​	equals() , 对比 key-value 是否相等
通过 Map.entrySet() 方法获得的是一组 Entry 的集合，保存在 Set 中，所以 Map 中的 Entry 也不能重复。

```
public Set<Map.Entry<K,V>> entrySet();
```



#### 2 Map 的四种遍历方式

​	1.使用 keySet 遍历：

```java
 Set set = map.keySet();
    for (Object key : set) {
        System.out.println(map.get(key));
    }
```

​	2.使用 values 遍历：

```java
 Collection values = map.values();
    Iterator iterator = values.iterator();
    while (iterator.hasNext()){
        System.out.println("value " + iterator.next());
    }
```

​	3.使用 Entry 遍历

```java
Set entrySet = map.entrySet();
    for (Object o : entrySet) {
        Map.Entry entry = (Map.Entry) o;
        System.out.println(entry);      //key=value
        System.out.println(entry.getKey() + " / " + entry.getValue());
    }
```

​	jdk8的forEach(内部也是用for (Map.Entry<K, V> entry : entrySet()) )

```java
Map<Integer,Integer> map = new HashMap<>();
        map.forEach((k,v) ->{
            System.out.println(k);
            System.out.println(v);
        });
```

​	4.使用Iterator遍历

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>(); 
Iterator<Map.Entry<Integer, Integer>> entries = map.entrySet().iterator(); 
while (entries.hasNext()) { 
  Map.Entry<Integer, Integer> entry = entries.next(); 
  System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue()); 
}
```

#### 3 Map的实现类

![这里写图片描述](https://img-blog.csdn.net/20161025193344555)

Map 的实现类主要有 4 种：

​	Hashtable 
​		古老，线程安全
​	HashMap 
​		速度很快，但没有顺序
​	TreeMap 
​		有序的，效率比 HashMap 低
​	LinkedHashMap 
​		结合 HashMap 和 TreeMap 的有点，有序的同时效率也不错，仅比 HashMap 慢一点

其中后三个的区别很类似 Set 的实现类：

​	HashSet
​	TreeSet
​	LinkedHashSet

Map 的每个实现类都应该实现 2 个构造方法：

1. 无参构造方法，用于创建一个空的 map
2. 参数是 Map 的构造方法，用于创建一个包含参数内容的新 map



第二种构造方法允许我们复制一个 map。

虽然没有强制要求，但自定义 Map 实现类时最好都这样来。



#### 4 总结

Map 有以下特点：

​	1. 没有重复的 key
​	2. 每个 key 只能对应一个 value, 多个 key 可以对应一个 value
​	3. key,value 都可以是任何引用类型的数据，包括 null
​	4. Map 取代了古老的 Dictionary 抽象类

注意： 

​	另一方面，你应该尽量避免使用“可变”的类作为 Map 的键。如果你将一个对象作为键值并保存在 Map 中，之后又改变了其状态，那么 Map 就会产生混乱，你所保存的值可能丢失。





## 9 Hash

### 什么是 Hash

​	Hash（哈希），又称“散列”。

​	散列（hash）英文原意是“混杂”、“拼凑”、“重新表述”的意思。

​	在某种程度上，散列是与排序相反的一种操作，排序是将集合中的元素按照某种方式比如字典顺序排列在一起，而散列通过计算哈希值，打破元素之间原有的关系，使集合中的元素按照散列函数的分类进行排列。

​	在介绍一些集合时，我们总强调需要重写某个类的 equlas() 方法和 hashCode() 方法，确保唯一性。这里的 hashCode() 表示的是对当前对象的唯一标示。计算 hashCode 的过程就称作 哈希。

### 为什么要有 Hash

​	我们通常使用数组或者链表来存储元素，一旦存储的内容数量特别多，需要占用很大的空间，而且在**查找某个元素**是否存在的过程中，数组和链表都需要挨个循环比较，而通过 哈希 计算，可以大大**减少比较次数**。

​	哈希 其实是随机存储的一种优化，先进行分类，然后查找时按照这个对象的分类去找。

​	哈希通过一次计算大幅度缩小查找范围，自然比从全部数据里查找速度要快。

### 哈希函数

​	哈希的过程中需要使用哈希函数进行计算。

​	哈希函数是一种映射关系，根据数据的关键词 key ，通过一定的函数关系，计算出该元素存储位置的函数。

​	表示为：

address = H [key]



### 几种常见的哈希函数（散列函数）构造方法

#### 1. 直接定址法 

​	取关键字或关键字的某个线性函数值为散列地址。
​	即 H(key) = key 或 H(key) = a*key + b，其中a和b为常数。

#### 2. 除留余数法 

​	取关键字被某个不大于散列表长度 m 的数 p 求余，得到的作为散列地址。
​	即 H(key) = key % p, p < m。

#### 3. 数字分析法 

​	当关键字的位数大于地址的位数，对关键字的各位分布进行分析，选出分布均匀的任意几位作为散列地址。
​	仅适用于所有关键字都已知的情况下，根据实际应用确定要选取的部分，尽量避免发生冲突。

#### 4. 平方取中法 

​	先计算出关键字值的平方，然后取平方值中间几位作为散列地址。
​	随机分布的关键字，得到的散列地址也是随机分布的。

#### 5. 折叠法（叠加法） 

​	将关键字分为位数相同的几部分，然后取这几部分的叠加和（舍去进位）作为散列地址。
​	用于关键字位数较多，并且关键字中每一位上数字分布大致均匀。

6. #### 随机数法 

  ​选择一个随机函数，把关键字的随机函数值作为它的哈希值。
  ​通常当关键字的长度不等时用这种方法。



构造哈希函数的方法很多，实际工作中要根据不同的情况选择合适的方法，总的原则是尽可能少的产生冲突。

通常考虑的因素有**关键字的长度和分布情况、哈希值的范围**等。

如：当关键字是整数类型时就可以用除留余数法；如果关键字是小数类型，选择随机数法会比较好。



### 哈希冲突的解决

​	选用哈希函数计算哈希值时，可能不同的 key 会得到相同的结果，一个地址怎么存放多个数据呢？这就是冲突。

常用的主要有两种方法解决冲突：

#### 1.链接法（拉链法）

拉链法解决冲突的做法是： 
将所有关键字为同义词的结点链接在同一个单链表中。

若选定的散列表长度为 m，则可将散列表定义为一个由 m 个头指针组成的指针数组 T[0..m-1] 。

凡是散列地址为 i 的结点，均插入到以 T[i] 为头指针的单链表中。 
T 中各分量的初值均应为空指针。

在拉链法中，装填因子 α 可以大于 1，但一般均取 α ≤ 1。



#### 2.开放定址法

用开放定址法解决冲突的做法是：

```
	当冲突发生时，使用某种探测技术在散列表中形成一个探测序列。沿此序列逐个单元地查找，直到找到给定的关键字，或者碰到一个开放的地址（即该地址单元为空）为止（若要插入，在探查到开放的地址，则可将待插入的新结点存人该地址单元）。查找时探测到开放的地址则表明表中无待查的关键字，即查找失败。
```

简单的说：当冲突发生时，使用某种探查(亦称探测)技术在散列表中寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到。

按照形成探查序列的方法不同，可将开放定址法区分为线性探查法、二次探查法、双重散列法等。

##### a.线性探查法

hi=(h(key)+i) ％ m ，0 ≤ i ≤ m-1

基本思想是： 
探查时从地址 d 开始，首先探查 T[d]，然后依次探查 T[d+1]，…，直到 T[m-1]，此后又循环到 T[0]，T[1]，…，直到探查到 有空余地址 或者到 T[d-1]为止。

##### b.二次探查法

hi=(h(key)+i*i) ％ m，0 ≤ i ≤ m-1

基本思想是： 
探查时从地址 d 开始，首先探查 T[d]，然后依次探查 T[d+1^2]，T[d+2^2]，T[d+3^2],…，等，直到探查到 有空余地址 或者到 T[d-1]为止。

缺点是无法探查到整个散列空间。

##### c.双重散列法

hi=(h(key)+i*h1(key)) ％ m，0 ≤ i ≤ m-1

基本思想是： 
探查时从地址 d 开始，首先探查 T[d]，然后依次探查 T[d+h1(d)], T[d + 2*h1(d)]，…，等。

该方法使用了两个散列函数 h(key) 和 h1(key)，故也称为双散列函数探查法。

定义 h1(key) 的方法较多，但无论采用什么方法定义，都必须使 h1(key) 的值和 m 互素，才能使发生冲突的同义词地址均匀地分布在整个表中，否则可能造成同义词地址的循环计算。

该方法是开放定址法中最好的方法之一。



### 哈希的应用

​	哈希表
​	分布式缓存

#### 哈希表（散列表）

​	哈希表（hash table）是哈希函数最主要的应用。

​	哈希表是实现关联数组（associative array）的一种数据结构，广泛应用于实现数据的快速查找。

![这里写图片描述](https://img-blog.csdn.net/20161027004004320)

​	用哈希函数计算关键字的哈希值（hash value）,通过哈希值这个索引就可以找到关键字的存储位置，即桶（bucket）。哈希表不同于二叉树、栈、序列的数据结构一般情况下，在哈希表上的插入、查找、删除等操作的时间复杂度是 O(1)。

​	查找过程中，关键字的比较次数，取决于产生冲突的多少，产生的冲突少，查找效率就高，产生的冲突多，查找效率就低。因此，影响产生冲突多少的因素，也就是影响查找效率的因素。 
影响产生冲突多少有以下三个因素：

1. 哈希函数是否均匀；

2. 处理冲突的方法；

3. 哈希表的加载因子。

   哈希表的加载因子和容量决定了在什么时候桶数（存储位置）不够，需要重新哈希。

   加载因子太大的话桶太多，遍历时效率变低；太小的话频繁 rehash，导致性能降低。所以加载因子的大小需要结合时间和空间效率考虑。

   在 HashMap 中的加载因子为 0.75，即四分之三。



#### 分布式缓存

​	网络环境下的分布式缓存系统一般基于一致性哈希（Consistent hashing）。简单的说，一致性哈希将哈希值取值空间组织成一个虚拟的环，各个服务器与数据关键字K使用相同的哈希函数映射到这个环上，数据会存储在它顺时针“游走”遇到的第一个服务器。可以使每个服务器节点的负载相对均衡，很大程度上避免资源的浪费。

​	在动态分布式缓存系统中，哈希算法的设计是关键点。使用分布更合理的算法可以使得多个服务节点间的负载相对均衡，可以很大程度上避免资源的浪费以及部分服务器过载。 使用带虚拟节点的一致性哈希算法，可以有效地降低服务硬件环境变化带来的数据迁移代价和风险，从而使分布式缓存系统更加高效稳定。





## 10 AbstractMap

### 什么是 AbstractMap

​	AbstractMap 是 [Map 接口的](http://blog.csdn.net/u011240877/article/details/52929523)的实现类之一，也是 HashMap, TreeMap, ConcurrentHashMap 等类的父类。

​	AbstractMap 提供了 Map 的基本实现，使得我们以后要实现一个 Map 不用从头开始，只需要继承 AbstractMap, 然后按需求实现/重写对应方法即可。

​	AbstarctMap 中唯一的抽象方法：

```java
public abstract Set<Entry<K,V>> entrySet();
```

当我们要实现一个 **不可变**的 Map 时，只需要继承这个类，然后实现 `entrySet()` 方法，这个方法返回一个保存所有 key-value 映射的 set。 通常这个 Set 不支持 add(), remove() 方法，Set 对应的迭代器也不支持 remove() 方法。

如果想要实现一个 **可变的** Map,我们需要在上述操作外，重写 put() 方法，因为 默认不支持 put 操作：

```
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}
```

而且 entrySet() 返回的 Set 的迭代器，也得实现 remove() 方法，因为 AbstractMap 中的 删除相关操作都需要调用该迭代器的 remove() 方法。

正如其他集合推荐的那样，比如 AbstractCollection 接口 ，实现类最好提供两种构造方法：

一种是不含参数的，返回一个空 map
一种是以一个 map 为参数，返回一个和参数内容一样的 map

### AbstractMap 的成员变量

```
transient volatile Set<K>        keySet;
transient volatile Collection<V> values;
```

有两个成员变量：

​	keySet, 保存 map 中所有键的 Set
​	values, 保存 map 中所有值的集合
他们都是 transient, volatile, 分别表示不可序列化、并发环境下变量的修改能够保证线程可见性。

需要注意的是 **volatile 只能保证可见性，不能保证原子性，**需要保证操作是原子性操作，才能保证使用 volatile 关键字的程序在并发时能够正确执行。

### AbstractMap 的成员方法

AbstractMap 中实现了许多方法，实现类会根据自己不同的要求选择性的覆盖一些。

接下来根据看看 AbstractMap 中的方法。

#### 1.添加

```java
public V put(K key, V value) {
    throw new UnsupportedOperationException();
}

public void putAll(Map<? extends K, ? extends V> m) {
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
        put(e.getKey(), e.getValue());
}
```

可以看到默认是不支持添加操作的，实现类需要重写 put() 方法。

#### 2.删除

```java
public V remove(Object key) {
    //获取保存 Map.Entry 集合的迭代器
    Iterator<Entry<K,V>> i = entrySet().iterator();
    Entry<K,V> correctEntry = null;
    //遍历查找，当某个 Entry 的 key 和 指定 key 一致时结束
    if (key==null) {
        while (correctEntry==null && i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getKey()==null)
                correctEntry = e;
        }
    } else {
        while (correctEntry==null && i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                correctEntry = e;
        }
    }

    //找到了，返回要删除的值
    V oldValue = null;
    if (correctEntry !=null) {
        oldValue = correctEntry.getValue();
        //调用迭代器的 remove 方法
        i.remove();
    }
    return oldValue;
}

//调用 Set.clear() 方法清除
public void clear() {
    entrySet().clear();
}
```

#### 3.获取

```java
//时间复杂度为 O(n)
//许多实现类都重写了这个方法
public V get(Object key) {
    //使用 Set 迭代器进行遍历，根据 key 查找
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (key==null) {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (e.getKey()==null)
                return e.getValue();
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                return e.getValue();
        }
    }
    return null;
}
```

#### 4,查询状态

```java
//是否存在指定的 key
//时间复杂度为 O(n)
//许多实现类都重写了这个方法
public boolean containsKey(Object key) {
    //还是迭代器遍历，查找 key，跟 get() 很像啊
    Iterator<Map.Entry<K,V>> i = entrySet().iterator();
    if (key==null) {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            //getKey()
            if (e.getKey()==null)
                return true;
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (key.equals(e.getKey()))
                return true;
        }
    }
    return false;
}


//查询是否存在指定的值
public boolean containsValue(Object value) {
    Iterator<Entry<K,V>> i = entrySet().iterator();
    if (value==null) {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            //getValue()
            if (e.getValue()==null)
                return true;
        }
    } else {
        while (i.hasNext()) {
            Entry<K,V> e = i.next();
            if (value.equals(e.getValue()))
                return true;
        }
    }
    return false;
}


public int size() {
    //使用 Set.size() 获取元素个数
    return entrySet().size();
}

public boolean isEmpty() {
    return size() == 0;
}
```

#### 5 .用于比较的 equals(), hashCode()

```java
//内部用来测试 SimpleEntry, SimpleImmutableEntry 是否相等的方法
private static boolean eq(Object o1, Object o2) {
    return o1 == null ? o2 == null : o1.equals(o2);
}

//判断指定的对象是否和当前 Map 一致
//为什么参数不是泛型而是 对象呢
//据说是创建这个方法时还没有泛型 - -
public boolean equals(Object o) {
    //引用指向同一个对象
    if (o == this)
        return true;

    //必须是 Map 的实现类
    if (!(o instanceof Map))
        return false;
    //强转为 Map
    Map<?,?> m = (Map<?,?>) o;
    //元素个数必须一致
    if (m.size() != size())
        return false;

    try {
        //还是需要一个个遍历，对比
        Iterator<Entry<K,V>> i = entrySet().iterator();
        while (i.hasNext()) {
            //对比每个 Entry 的 key 和 value
            Entry<K,V> e = i.next();
            K key = e.getKey();
            V value = e.getValue();
            if (value == null) {
                //对比 key, value
                if (!(m.get(key)==null && m.containsKey(key)))
                    return false;
            } else {
                if (!value.equals(m.get(key)))
                    return false;
            }
        }
    } catch (ClassCastException unused) {
        return false;
    } catch (NullPointerException unused) {
        return false;
    }

    return true;
}

//整个 map 的 hashCode() 
public int hashCode() {
    int h = 0;
    //是所有 Entry 哈希值的和
    Iterator<Entry<K,V>> i = entrySet().iterator();
    while (i.hasNext())
        h += i.next().hashCode();
    return h;
}
```

#### 6.获取三个主要的视图

获取所有的键:

```java
public Set<K> keySet() {
    //如果成员变量 keySet 为 null,创建个空的 AbstractSet
    if (keySet == null) {
        keySet = new AbstractSet<K>() {
            public Iterator<K> iterator() {
                return new Iterator<K>() {
                    private Iterator<Entry<K,V>> i = entrySet().iterator();

                    public boolean hasNext() {
                        return i.hasNext();
                    }

                    public K next() {
                        return i.next().getKey();
                    }

                    public void remove() {
                        i.remove();
                    }
                };
            }

            public int size() {
                return AbstractMap.this.size();
            }

            public boolean isEmpty() {
                return AbstractMap.this.isEmpty();
            }

            public void clear() {
                AbstractMap.this.clear();
            }

            public boolean contains(Object k) {
                return AbstractMap.this.containsKey(k);
            }
        };
    }
    return keySet;
}
```

获取所有的值:

```java
public Collection<V> values() {
    if (values == null) {
        //没有就创建个空的 AbstractCollection 返回
        values = new AbstractCollection<V>() {
            public Iterator<V> iterator() {
                return new Iterator<V>() {
                    private Iterator<Entry<K,V>> i = entrySet().iterator();

                    public boolean hasNext() {
                        return i.hasNext();
                    }

                    public V next() {
                        return i.next().getValue();
                    }

                    public void remove() {
                        i.remove();
                    }
                };
            }

            public int size() {
                return AbstractMap.this.size();
            }

            public boolean isEmpty() {
                return AbstractMap.this.isEmpty();
            }

            public void clear() {
                AbstractMap.this.clear();
            }

            public boolean contains(Object v) {
                return AbstractMap.this.containsValue(v);
            }
        };
    }
    return values;
}
```

获取所有键值对，需要子类实现：

```java
public abstract Set<Entry<K,V>> entrySet();
```

### AbstractMap 中的内部类

正如 Map 接口 中有内部类 Map.Entry 一样， AbstractMap 也有两个内部类：

​	SimpleImmutableEntry, 表示一个不可变的键值对;
​	SimpleEntry, 表示可变的键值对;

SimpleImmutableEntry，不可变的键值对,实现了 Map.Entry < K,V> 接口：

```java
public static class SimpleImmutableEntry<K,V>
    implements Entry<K,V>, java.io.Serializable
{
    private static final long serialVersionUID = 7138329143949025153L;
    //key-value
    private final K key;
    private final V value;

    //构造函数，传入 key 和 value
    public SimpleImmutableEntry(K key, V value) {
        this.key   = key;
        this.value = value;
    }

    //构造函数2，传入一个 Entry，赋值给本地的 key 和 value
    public SimpleImmutableEntry(Entry<? extends K, ? extends V> entry) {
        this.key   = entry.getKey();
        this.value = entry.getValue();
    }

    //返回 键
    public K getKey() {
        return key;
    }

    //返回 值
    public V getValue() {
        return value;
    }

    //修改值，不可修改的 Entry 默认不支持这个操作
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    //比较指定 Entry 和本地是否相等
    //要求顺序，key-value 必须全相等
    //只要是 Map 的实现类即可，不同实现也可以相等
    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;
        return eq(key, e.getKey()) && eq(value, e.getValue());
    }

    //哈希值
    //是键的哈希与值的哈希的 异或
    public int hashCode() {
        return (key   == null ? 0 :   key.hashCode()) ^
               (value == null ? 0 : value.hashCode());
    }

    //返回一个 String
    public String toString() {
        return key + "=" + value;
    }
}
```

SimpleEntry, 可变的键值对:

```java
public static class SimpleEntry<K,V>
    implements Entry<K,V>, java.io.Serializable
{
    private static final long serialVersionUID = -8499721149061103585L;

    private final K key;
    private V value;

    public SimpleEntry(K key, V value) {
        this.key   = key;
        this.value = value;
    }
    public SimpleEntry(Entry<? extends K, ? extends V> entry) {
        this.key   = entry.getKey();
        this.value = entry.getValue();
    }
    public K getKey() {
        return key;
    }
    public V getValue() {
        return value;
    }
    //支持 修改值
    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }
    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;
        return eq(key, e.getKey()) && eq(value, e.getValue());
    }
    public int hashCode() {
        return (key   == null ? 0 :   key.hashCode()) ^
               (value == null ? 0 : value.hashCode());
    }
    public String toString() {
        return key + "=" + value;
    }
}
```



SimpleEntry 与 SimpleImmutableEntry 唯一的区别就是支持 setValue() 操作。

### 总结

​	和 AbstractCollection 接口，AbstractList 接口 作用相似， AbstractMap 是一个基础实现类，实现了 Map 的主要方法，默认不支持修改。

​	常用的几种 Map, 比如 HashMap, TreeMap, LinkedHashMap 都继承自它。



## 11 HashMap

### 什么是 HashMap

​	HashMap 是一个采用**哈希表**实现的键值对集合，继承自 AbstractMap，实现了 Map 接口 。 
​	HashMap 的特殊存储结构使得在获取指定元素前需要经过**哈希运算**，得到目标元素在哈希表中的位置，然后再进行少量比较即可得到元素，这使得 HashMap 的查找效率贼高。

​	当发生 哈希冲突（碰撞）的时候，HashMap 采用 拉链法 进行解决，因此 HashMap 的底层实现是**数组+链表**，如下图 所示：

![这里写图片描述](https://img-blog.csdn.net/20161027004004320)

### HashMap 的特点

结合平时使用，可以了解到 HashMap 大概具有以下特点：

​	底层实现是 **链表+数组**，JDK 8 后又加了 **红黑树**
​	实现了 Map 全部的方法
​	key 用 **Set** 存放，所以想做到 **key 不允许重复**，key 对应的类需要**重写 hashCode 和 equals** 方法
​	**允许空键和空值**（但空键只有一个，且放在第一位，下面会介绍）
​	元素是**无序**的，而且顺序会不定时改变
​	插入、获取的时间复杂度基本是 **O(1)**（前提是有适当的哈希函数，让元素分布在均匀的位置）
​	遍历整个 Map 需要的时间与 **桶(数组) 的长度**成正比（因此初始化时 HashMap 的容量不宜太大）
​	两个关键因子：**初始容量、加载因子**
​	

**Hashtable 不允许 null 并且同步**，其他的几乎和HashMap一样。

下面结合源码进行验证这些特点。

### HashMap 的 13 个成员变量

![这里写图片描述](https://img-blog.csdn.net/20161030092127363)



1.默认初始容量：16，必须是 2 的整数次方

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
```

2.默认加载因子的大小：0.75，可不是随便的，结合时间和空间效率考虑得到的

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

3.最大容量： 2^ 30 次方

```java
static final int MAXIMUM_CAPACITY = 1 << 30;
```

4.当前 HashMap 修改的次数，这个变量用来保证 *fail-fast* 机制

```java
transient int modCount;
```

5.阈值，下次需要扩容时的值，等于 容量*加载因子

```java
int threshold;
```

6.树形阈值：JDK 1.8 新增的，当使用 树 而不是列表来作为桶时使用。必须必 2 大

```java
static final int TREEIFY_THRESHOLD = 8;
```

7.非树形阈值：也是 1.8 新增的，扩容时分裂一个树形桶的阈值（？不是很懂 - -），要比 TREEIFY_THRESHOLD 小

```java
static final int UNTREEIFY_THRESHOLD = 6;
```

8.树形最小容量：桶可能是树的哈希表的最小容量。至少是 TREEIFY_THRESHOLD 的 4 倍，这样能避免扩容时的冲突

```java
static final int MIN_TREEIFY_CAPACITY = 64;
```

9.缓存的 键值对集合（另外两个视图：keySet 和 values 是在 [AbstractMap](http://blog.csdn.net/u011240877/article/details/52949046) 中声明的）

```java
transient Set<Map.Entry<K,V>> entrySet;
```

10.哈希表中的链表数组

```java
transient Node<K,V>[] table;
```

11.键值对的数量

```java
transient int size;
```

12.哈希表的加载因子

```java
final float loadFactor;
```



### HashMap 的初始容量和加载因子

由于 HashMap 扩容开销很大（需要创建新数组、重新哈希、分配等等），因此与扩容相关的两个因素：

​	容量：数组的数量
​	加载因子：决定了 HashMap 中的元素占有多少比例时扩容
成为了 HashMap 最重要的部分之一，它们决定了 HashMap 什么时候扩容。

HashMap 的默认加载因子为 0.75，这是在时间、空间两方面均衡考虑下的结果：

​	加载因子太大的话发生冲突的可能就会大，查找的效率反而变低
​	太小的话频繁 rehash，导致性能降低

当设置初始容量时，需要提前考虑 Map 中可能有多少对键值对，设计合理的加载因子，尽可能避免进行扩容。

如果存储的键值对很多，干脆设置个大点的容量，这样可以少扩容几次。



## HashMap 的关键方法

### 1. HashMap 的 4 个构造方法

```java
//创建一个空的哈希表，初始容量为 16，加载因子为 0.75
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

//创建一个空的哈希表，指定容量，使用默认的加载因子
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

//创建一个空的哈希表，指定容量和加载因子
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    //根据指定容量设置阈值
    this.threshold = tableSizeFor(initialCapacity);
}

//创建一个内容为参数 m 的内容的哈希表
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

​	其中第三种构造方法调用了 `tableSizeFor(int)` 来根据指定的容量设置阈值，这个方法经过若干次无符号右移、求异运算，得出最接近指定参数 cap 的 2 的 N 次方容量。假如你传入的是 5，返回的初始容量为 8 

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

​	第四种构造方法调用了 `putMapEntries()`，这个方法用于向哈希表中添加整个集合：

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        //数组还是空，初始化参数
        if (table == null) { 
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        //数组不为空，超过阈值就扩容
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            //先经过 hash() 计算位置，然后复制指定 map 的内容
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```



### 2.HashMap 中的链表节点

前面提到，HashMap 的底层数据结构之一（JDK 1.8 前单纯是）就是链表数组：

```
transient Node<K,V>[] table;
```

每一个链表节点如下：

```java
//实现了 Map.Entry 接口
static class Node<K,V> implements Map.Entry<K,V> {
    //哈希值，就是位置
    final int hash;
    //键
    final K key;
    //值
    V value;
    //指向下一个节点的指针
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            //Map.Entry 相等的条件：键相等、值相等、个数相等、顺序相等
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

### 3.HashMap 中的添加操作 **

下文用“桶”来指代要数组，每个桶都对应着一条链表：

```java
//添加指定的键值对到 Map 中，如果已经存在，就替换
public V put(K key, V value) {
    //先调用 hash() 方法计算位置
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //如果当前 哈希表内容为空，新建，n 指向最后一个桶的位置，tab 为哈希表另一个引用
    //resize() 后续介绍
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //如果要插入的位置没有元素，新建个节点并放进去
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        //如果要插入的桶已经有元素，替换
        // e 指向被替换的元素
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
        //p 指向要插入的桶第一个 元素的位置，如果 p 的哈希值、键、值和要添加的一样，就停止找，e 指向 p
            e = p;
        else if (p instanceof TreeNode)
        //如果不一样，而且当前采用的还是 JDK 8 以后的树形节点，调用 putTreeVal 插入
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
        //否则还是从传统的链表数组查找、替换

            //遍历这个桶所有的元素
            for (int binCount = 0; ; ++binCount) {
                //没有更多了，就把要添加的元素插到后面得了
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //当这个桶内链表个数大于等于 8，就要树形化
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //如果找到要替换的节点，就停止，此时 e 已经指向要被替换的节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //存在要替换的节点
        if (e != null) {
            V oldValue = e.value;
            //替换，返回
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //如果超出阈值，就得扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

根据代码可以总结插入逻辑如下:

1. 先调用 hash() 方法计算哈希值

2. 然后调用 putVal() 方法中根据哈希值进行相关操作
3. 如果当前 哈希表内容为空，新建一个哈希表
4. 如果要插入的桶中没有元素，新建个节点并放进去
5. 否则从桶中第一个元素开始查找哈希值对应位置 
   1. 如果桶中第一个元素的哈希值和要添加的一样，替换，结束查找
   2. 如果第一个元素不一样，而且当前采用的还是 JDK 8 以后的树形节点，调用 putTreeVal() 进行插入
   3. 否则还是从传统的链表数组中查找、替换，结束查找
   4. 当这个桶内链表个数大于等于 8，就要调用 treeifyBin() 方法进行树形化
6. 最后检查是否需要扩容

插入过程中涉及到几个其他关键的方法 ：

hash():	计算对应的位置
resize():	扩容
putTreeVal():	树形节点的插入
treeifyBin():	树形化容器

HashMap 在 JDK1.8 新增树形化相关的内容比较多，下一篇介绍，接下来先介绍传统的 HashMap 相关内容。



### 4.HashMap 中的哈希函数 hash() **

HashMap 中通过将传入键的 hashCode 进行无符号右移 16 位，然后进行按位异或，得到这个键的哈希值。

看看源码里的注释怎么说的：

```
Computes key.hashCode() and spreads (XORs) higher bits of hash to lower. Because the table uses power-of-two masking, sets of hashes that vary only in bits above the current mask will always collide. (Among known examples are sets of Float keys holding consecutive whole numbers in small tables.) So we apply a transform that spreads the impact of higher bits downward. There is a tradeoff between speed, utility, and quality of bit-spreading. Because many common sets of hashes are already reasonably distributed (so don’t benefit from spreading), and because we use trees to handle large sets of collisions in bins, we just XOR some shifted bits in the cheapest possible way to reduce systematic lossage, as well as to incorporate impact of the highest bits that would otherwise never be used in index calculations because of table bounds.
```

大概意思就是：

​	由于哈希表的容量都是 2 的 N 次方，在当前，元素的 hashCode() 在很多时候下低位是相同的，这将导致冲突（碰撞），因此 1.8 以后做了个移位操作：将元素的 hashCode() 和自己右移 16 位后的结果求异或。

由于 int 只有 32 位，无符号右移 16 位相当于把高位的一半移到低位：

![这里写图片描述](https://img-blog.csdn.net/20161030215139729)

举个栗子：

![这里写图片描述](https://img-blog.csdn.net/20161031000139666) 

这样可以避免只靠低位数据来计算哈希时导致的冲突，计算结果由高低位结合决定，可以避免哈希值分布不均匀。而且，采用位运算效率更高。

### 5.HashMap 中的初始化/扩容方法 resize() **

每次添加时会比较当前元素个数和阈值：

```java
//如果超出阈值，就得扩容
    if (++size > threshold)
        resize();
```

**扩容：**

```java
   final Node<K,V>[] resize() {
    //复制一份当前的数据
    Node<K,V>[] oldTab = table;
    //保存旧的元素个数、阈值
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
          // 旧的容量已经超过最大容量,则修改阈值为最大整数
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //新的容量为旧的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            //如果旧容量大于等于 16，新的阈值就是旧阈值的两倍
            newThr = oldThr << 1; // double threshold
    }
    //如果旧容量为 0 ，并且旧阈值>0，说明之前创建了哈希表但没有添加元素，初始化容量等于阈值
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        //旧容量、旧阈值都是0，说明还没创建哈希表，容量为默认容量，阈值为 容量*加载因子
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //如果新的阈值为 0 ，就得用 新容量*加载因子 重计算一次
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //更新阈值
    threshold = newThr;
    //创建新链表数组，容量是原来的两倍
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    //接下来就得遍历复制了
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                //旧的桶置为空
                oldTab[j] = null;
                //当前 桶只有一个元素，直接赋值给对应位置
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    //如果旧哈希表中这个位置的桶是树形结构，就要把新哈希表里当前桶也变成树形结构
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { //保留旧哈希表桶中链表的顺序
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    //do-while 循环赋值给新哈希表
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

扩容过程中几个关键的点：

1. 新初始化哈希表时，容量为**默认容量**，阈值为 容量*加载因子

2. 已有哈希表扩容时，容量、阈值均**翻倍**

3. 如果之前这个桶的节点类型是**树**，需要把新哈希表里当前桶也变成树形结构

4. 复制给新哈希表中需要重新索引（rehash），这里采用的计算方法是 

   1. e.hash & (newCap - 1)，等价于 e.hash % newCap ( newCap必须为2的整数次方)

      (实际上是hash&111,就是取hash的最后几位)

结合扩容源码可以发现扩容的确开销很大，需要迭代所有的元素，rehash、赋值，还得保留原来的数据结构。

所以在使用的时候，最好在初始化的时候就指定好 HashMap 的长度，尽量避免频繁 resize()。



### 6.HashMap 的获取方法 get() **

HashMap 另外一个经常使用的方法就是 get(key)，返回键对应的值:

如果 HashMap 中包含一个键值对 k-v 满足：

```
(key == null ? k == null : key.equals(k))
```

就返回值 v，否则返回 null;

```java
public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    //tab 指向哈希表，n 为哈希表的长度，first 为 (n - 1) & hash 位置处的桶中的头一个节点
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        //如果桶里第一个元素就相等，直接返回
        if (first.hash == hash &&
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        //否则就得慢慢遍历找
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                //如果是树形节点，就调用树形节点的 get 方法
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                //do-while 遍历链表的所有节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

查找 方法比较简单:

1. 先计算哈希值;
2. 然后再用 (n - 1) & hash 计算出桶的位置;
3. 在桶里的链表进行遍历查找。

时间复杂度一般跟链表长度有关，因此哈希算法越好，元素分布越均匀，get() 方法就越快，不然遍历一条长链表，太慢了。

不过在 JDK 1.8 以后 HashMap 新增了红黑树节点，优化这种极端情况下的性能问题。



### 总结

1. HashMap 有那么多优点，也有个缺点：不是同步的。

  ​当多线程并发访问一个 哈希表时，需要在外部进行同步操作，否则会引发数据不同步问题。

你可以选择加锁，也可以考虑用 Collections.synchronizedMap 包一层，变成个线程安全的 Map：

```java
Map m = Collections.synchronizedMap(new HashMap(...));
```

​	最好在初始化时就这么做。

2. HashMap 三个视图返回的迭代器都是 fail-fast 的：如果在迭代时使用非迭代器方法修改了 map 的内容、结构，迭代器就会报 ConcurrentModificationException 的错。

3. 当 HashMap 中有大量的元素都存放到同一个桶中时，这时候哈希表里只有一个桶，这个桶下有一条长长的链表，这个时候 HashMap 就相当于一个单链表，假如单链表有 n 个元素，遍历的时间复杂度就是 O(n)，完全失去了它的优势。

   针对这种情况，JDK 1.8 中引用了 红黑树（时间复杂度为 O(logn)） 优化这个问题。

   4.为什么哈希表的容量一定要是 2的整数次幂?

```
	首先，capacity 为 2的整数次幂的话，计算桶的位置 h&(length-1) 就相当于对 length 取模，提升了计算效率；
	其次，capacity 为 2 的整数次幂的话，为偶数，这样 capacity-1 为奇数，奇数的最后一位是 1，这样便保证了 h&(capacity-1) 的最后一位可能为 0，也可能为 1（这取决于h的值），即与后的结果可能为偶数，也可能为奇数，这样便可以保证散列的均匀性；
	而如果 capacity 为奇数的话，很明显 capacity-1 为偶数，它的最后一位是 0，这样 h&(capacity-1) 的最后一位肯定为 0，即只能为偶数，这样任何 hash 值都只会被散列到数组的偶数下标位置上，这便浪费了近一半的空间。
```

因此，哈希表容量取 2 的整数次幂，有以下 2 点好处：

​		1. 使用减法替代取模，提升计算效率；

​		2. 为了使不同 hash 值发生碰撞的概率更小，尽可能促使元素在哈希表中均匀地散列。

5.HashMap 允许 key, value 为 null，同时他们都保存在**第一个桶**中。

看代码，添加时先调用 hash()：

```java
public V put(K key, V value) {
    //先调用 hash() 方法计算位置
    return putVal(hash(key), key, value, false, true);
}
// 而在计算哈希值时，如果为 null 就直接返回 0 ，说明了这一点：
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

6.HashMap 中 equals() 和 hashCode() 有什么作用？

​	HashMap 的添加、获取时需要通过 key 的 hashCode() 进行 hash()，然后计算下标 ( n-1 & hash)，从而获得要找的同的位置。

​	当发生冲突（碰撞）时，利用 key.equals() 方法去链表或树中去查找对应的节点。

7.你知道 hash 的实现吗？为什么要这样实现？

​	在 JDK 1.8 的实现中，是通过 hashCode() 的高16位异或低16位实现的：

​		(h = k.hashCode()) ^ (h >>> 16)。

​	主要是从速度、功效、质量 来考虑的，这么做可以在桶的 n 比较小的时候，保证高低 bit 都参与到 hash 的计算中，同时位运算不会有太大的开销。



## 12. ConcurrentHashMap

​	其实可以看出JDK1.8版本的ConcurrentHashMap的数据结构已经接近HashMap，相对而言，ConcurrentHashMap只是增加了同步的操作来控制并发，从JDK1.7版本的**ReentrantLock+Segment+HashEntry**，到JDK1.8版本中**synchronized+CAS+HashEntry+红黑树**。

​	1.数据结构：取消了Segment分段锁的数据结构，取而代之的是数组+链表+红黑树的结构。

​	2.保证线程安全机制：JDK1.7采用segment的分段锁机制实现线程安全，其中segment继承自ReentrantLock。JDK1.8采用CAS+Synchronized保证线程安全。

​	3.锁的粒度：原来是对需要进行数据操作的**Segment**加锁，现调整为对每个**数组元素**加锁（Node）。

​	4.链表转化为红黑树:定位结点的hash算法简化会带来弊端,Hash冲突加剧,因此在链表节点数量大于8时，会将链表转化为红黑树进行存储。

​	5.查询时间复杂度：从原来的遍历链表O(n)，变成遍历红黑树O(logN)。