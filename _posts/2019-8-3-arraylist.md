---
layout: post
title: 深度分析ArrayList源码
categories: ArrayList
description: ArrayList分析。
keywords: Java
---

ArrayList是我们开发中最常用的数据存储容器之一，其底层是数组实现的，我们可以在集合中存储任意类型的数据,那麽它的源码是怎么实现的呢。

## 1.简介

ArrayList是我们开发中最常用的数据存储容器之一，其底层是数组实现的，我们可以在集合中存储任意类型的数据，ArrayList是线程不安全的，非常适用于对元素进行查找，效率非常高。

## 2.继承体系
![ArrayList继承图](https://mmbiz.qpic.cn/mmbiz_png/C91PV9BDK3yUT9hffoWkuTicrshhl1d9JkxvvXoUmvsSzErEeTCcxTnYkYAgWexziaCbpcclHf0ocGhxBJdxlXqA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ArrayList实现List、RandomAccess、Cloneable、Serializable等接口。

1.Arraylist实现List，提供了基础的添加、删除、遍历等操作。
2.ArrayList实现RandomAccess，提供随机访问的能力。
3.ArrayList实现Cloneable，可以被克隆。
4.ArrayList实现Serializable，可以被序列化。

## 3.线程安全性

对ArrayList的操作一般分为两个步骤，改变元素的位置（大小)和操作元素。这个过程在多线程的环境下是不能保证具有原子性的，因此ArrayList在多线程下是不安全的。

## 4.源码分析

```
    /**
     * Default initial capacity.
     * 
     *  默认初始化容量
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     * 
     * 如果自定义容量为0，则会默认用它来初始化ArrayList，或者用于空数组替换
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     * 
     * 如果没有自定义容量，则会使用它来初始化ArrayList，或者用于空数组比对。
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     * 
     * 这就是ArrayList底层用到的数组
     * 非私有，用于简化嵌套类访问
     * transient 在已经实现序列化的类中，不允许某变量序列化。
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     * 
     *  实际ArrayList的大小/集合中元素的个数
     */
    private int size;

```

 <b>拓展：什么是序列化？</b>
    序列化是指：将对象转换成以字节序列的形式来表示，以便于持久化和传输。
    实现方法：实现序列化接口。
    然后用的时候拿出来进行反序列化即可又变成java对象。
    

  <b>transient关键字解析</b>
    java中的transient关键字的作用，简单地说，就是让某些被修饰的变量不参加序列化。

有了`transient`关键字声明，`transient Object[] elementData`,事实上我们使用ArrayList在网络传输的时候很正常，并没有出现空值,为什么呢?

看源码：
```
 private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        // 防止序列化期间有修改
        int expectedModCount = modCount;
        // 写出非transient非static属性（会写出size属性）
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        // 写出元素个数
        s.writeInt(size);

        // Write out all elements in the proper order.
          
        // 依次写出元素
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }

    /**
     * Reconstitute the <tt>ArrayList</tt> instance from a stream (that is,
     * deserialize it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
             
        // 声明为空数组
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        
        // 读入非transient非static属性（会读取size属性）
        s.defaultReadObject();

        // Read in capacity
        // 读入元素个数，没什么用，只是因为写出的时候写了size属性，读的时候也要按顺序来读
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
               
        // 计算容量
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            
        // 检查是否需要扩容
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            
        // 依次读取元素到数组中
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```

查看`writeObject()`方法可知，先调用`s.defaultWriteObject()`方法，再把size写入到流中，再把元素一个一个的写入到流中。

一般地，只要实现了Serializable接口即可自动序列化，`writeObject()`和`readObject()`是为了自己控制序列化的方式，这两个方法必须声明为private，在java.io.ObjectStreamClass#getPrivateMethod()方法中通过反射获取到writeObject()这个方法。

在ArrayList的`writeObject()`方法中先调用了`s.defaultWriteObject()`方法，这个方法是写入非static非transient的属性，在ArrayList中也就是size属性。同样地，在`readObject()`方法中先调用了`s.defaultReadObject()`方法解析出了size属性。


    那为什么不直接使用elementData中序列化，而采用上述的方式来实现序列化呢？
    
`elementData`定义为`transient`的优势，自己根据size序列化真实的元素，而不是根据数组的长度序列化元素，`elementData`是一个缓存数组，它通常会预留一些容量，等容量不足时再扩充容量，那么有些空间可能就没有实际存储元素，采用上述的方式来实现序列化，就可以保证只序列化实际存储的那些元素，而不是整个数组，从而节省空间。

### ArrayList(int initialCapacity)构造方法
```

 /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //如果传入的初始容量大于0，就新建一个数组存储元素
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //如果传入的初始化容量等于0，使用空数组EMPTY_ELEMENTDATA
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //如果传入的初始化容量小于0，抛出异常。
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

### ArrayList（）构造方法分析
```
 /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {

        //如果没有传入初始化容量，则使用空数组 DEFAULTCAPACITY_EMPTY_ELEMENTDATA
        //使用这个数组是在添加第一个元素的时候就会扩容到默认大小10
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

### ArrayList(Collection c) 构造方法分析
```
 /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     * 
     * 把传入集合的元素集合初始化到ArrayList中
     */
    public ArrayList(Collection<? extends E> c) {
        //集合转数组
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            //检查c.toArray()返回值是不是Object[]类型，如果不是，重新拷贝成Object[].class类型
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            //如果c是空集合，则初始化为空数组EMPTY_ELEMENTDATA
            this.elementData  = EMPTY_ELEMENTDATA;
        }
    }
```

为什么c.toArray();返回的有可能不是Object[]类型呢？请看下面的代码：
```
public class ArrayTest {

    public static void main(String[] args) {
        Father[] fathers = new Son[]{};
        //打印的结果：class [Lcom.ls.test.Son;
        System.out.println(fathers.getClass());

        //打印的结果：class [Ljava.lang.String;
        List<String> strList = new MyList();

        System.out.println(strList.toArray().getClass());
    }
}

class Father{}

class Son extends Father{}

class MyList extends ArrayList<String>{
  
    @Override
    public Object[] toArray() {
        return new String[]{"1","2","3"};
    }
}
```

### add(E e)的方法
    在数组的末尾追加元素,平均时间复杂度为O(1)。
```


    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     * 
     *  添加元素方法，添加元素到末尾，平均时间复杂度为O(1)
     */
    public boolean add(E e) {
        //检查是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //把元素放到最后一位
        elementData[size++] = e;
        return true;
    }   
    
```
它首先调用了ensureCapacityInternal(size + 1)方法。注意参数大小是+1 ,这个是面试考点。
```
 private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
```
这个方法里面又嵌套调用了两个方法：计算容量+确保容量。

计算容量：如果elementData中是空，则返回默认容量10和大小+1的最大值，否则返回大小+1

```
 private static int calculateCapacity(Object[] elementData, int minCapacity) {
        //如果是空数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA，则初始化为默认大小10.
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```

计算完容量后，进行确保容量可用：(modCount不用理它，它用来计算修改次数)

如果`size+1 > elementData.length`证明数组已经放满，则增加容量，调用grow()
    
```
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //扩容
            grow(minCapacity);
    }
```

增加容量：默认1.5倍扩容。 
1.获取当前数组的长度 >= oldCapacity
2.oldCapacity >> 1,表示将oldCapacity右移一位（位运算），相当于除2.再加上1，相当于新容量扩容1.5倍。
3.如果新容量小于需要的容量，则以需要的容量为准。
4.如果新容量比最大值还要大，则将新容量赋值为最大值。
5.以新容量拷贝出来一个新的数组赋值给elementData

```
 /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     * 扩容方法
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {

        // overflow-conscious code
    
        int oldCapacity = elementData.length;
        //新容量为旧容量的1.5倍。
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果新容量发现比需要的容量还小，则以需要的为准。
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
            //如果新容量已将超过最大容量了，则使用最大容量。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:

        //以新容量拷贝出来一个新的数组。
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
** size+1 的问题** 
    好了,到这里说一下为什么要size+1?
  
size + 1代表的含义是：
    1.如果集合添加元素成功后，集合中的实际元素个数。
    2.为了确保扩容不会出现错误。
    
假如不size+1处理，如果默认大小是0，则0 + 0 >> 1还是0。
如果size是1，则1 + 1 >> 1还是1。有人问：不是默认容量大小是10吗? 事实上，jdk1.8版本以后，ArrayList中的扩容放在add()方法中。之前放在构造方法中。我用的是1.8版本，默认所以ArrayList arrayList = new ArrayList();size应该是0。所以,size+ 1对扩容来讲很必要。

![Markdown](http://i1.fuimg.com/695115/ccf37698e32acaa8.png)
事实上上面的代码是证明不了容量大小的，因为size在只会调用add()方法时才会自增。

### add（int index，E element）方法
添加元素到指定位置，平均时间复杂度为O(n)。

```
 /**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     * 
     *   添加元素到指定位置，平均时间复杂度为O(n)
     */
    public void add(int index, E element) {
        //检查是否越界
        rangeCheckForAdd(index);
        //检查是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //将index及其之后的元素往后挪一位，则index位置处就空出来啦。
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);

        //将元素插入到index的位置
        elementData[index] = element;
        //大小增1
        size++;
    }

```
在这里说一下 `System.arraycopy(elementData, index, elementData, index + 1,
                         size - index)方法`：
```
    /* @param      src      the source array.
     * @param      srcPos   starting position in the source array.
     * @param      dest     the destination array.
     * @param      destPos  starting position in the destination data.
     * @param      length   the number of array elements to be copied.
     * @exception  IndexOutOfBoundsException  if copying would cause
     *               access of data outside array bounds.
     * @exception  ArrayStoreException  if an element in the <code>src</code>
     *               array could not be stored into the <code>dest</code> array
     *               because of a type mismatch.
     * @exception  NullPointerException if either <code>src</code> or
     *               <code>dest</code> is <code>null</code>.
     */
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
```
1.Object src：原数组

2.int srcPos：从元数据的起始位置开始

3.Object dest：目标数组

4.int destPos：目标数组的开始起始位置

5.int length：要复制的数组的长度   

    示例：大小为6，调用我们add(2,element)方法，则会从指数= 2+1=3的位置开始，将数组元素替换为从索引起始位置为index=2，为长度6-2=4的数据。

![640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbudHYyNAARjB51oR4sFRGZ3MC3pae6zZZ9lctHhdlNzBywNz6LtYleticZPAdvN1fSpfxsNCUzryoOQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### addAll(Collection c)方法
求两个集合的并集。
```
public boolean addAll(Collection<? extends E> c) {
        //将集合c转为数组
        Object[] a = c.toArray();
        int numNew = a.length;
        //检查是否需要扩容
        ensureCapacityInternal(size + numNew);  // Increments modCount
        //将c中的元素全部拷贝到数组的最后。
        System.arraycopy(a, 0, elementData, size, numNew);
        //大小增加c的大小
        size += numNew;
        //如果c不为空就返回true,否则返回false。
        return numNew != 0;
    }

```
###  get（int index）方法
获取指定索引位置的元素，时间复杂度为O(1)。

```
  public E get(int index) {
        //检查元素是否越界
        rangeCheck(index);
        //返回数组index位置的元素
        return elementData(index);
    }
        
 private void rangeCheck(int index) {
            if (index < 0 || index >= this.size)
                throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
        }
        
   @SuppressWarnings("unchecked")
    E elementData(int index) {
        //返回对应索引的值
        return (E) elementData[index];
    }

```
### remove(int index)方法
删除指定索引位置的元素，时间复杂度为O(n)。

```
 public E remove(int index) {
            
        // 检查是否越界
        rangeCheck(index);

        modCount++;
         
        // 获取index位置的元素
        E oldValue = elementData(index);
           
        // 如果index不是最后一位，则将index之后的元素往前挪一位
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        // 将最后一个元素删除，帮助GC
        elementData[--size] = null; // clear to let GC do its work
 
        // 返回旧值
        return oldValue;
    }
```
1.检查索引是否越界；

2.获取指定索引位置的元素；

3.如果删除的不是最后一位，则其它元素往前移一位；

4.将最后一位置为null，方便GC回收；

5.返回删除的元素。

可以看到，ArrayList删除元素的时候并没有缩容

### remove(Object o)方法

删除指定元素值的元素，时间复杂度为O(n)

```
 public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
            
            // 遍历整个数组，找到元素第一次出现的位置，并将其快速删除
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            
            // 遍历整个数组，找到元素第一次出现的位置，并将其快速删除
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
    
 private void fastRemove(int index) {
        modCount++;
         
        // 如果index不是最后一位，则将index之后的元素往前挪一位
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);

        // 将最后一个元素删除，帮助GC
        elementData[--size] = null; // clear to let GC do its work
    }

```

1.找到第一个等于指定元素值的元素；

2.快速删除；

fastRemove(int index)相对于remove(int index)少了检查索引越界的操作，可见jdk将性能优化到极致。

### retainAll(Collection c)方法

求两个集合的交集。

```
 public boolean retainAll(Collection<?> c) {
        // 集合c不能为null
        Objects.requireNonNull(c);
         
        // 调用批量删除方法，这时complement传入true，表示删除不包含在c中的元素
        return batchRemove(c, true);
    }

 /**

    * 批量删除元素

    * complement为true表示删除c中不包含的元素

    * complement为false表示删除c中包含的元素

    */
    private boolean batchRemove(Collection<?> c, boolean complement) {

         
    // 使用读写两个指针同时遍历数组

    // 读指针每次自增1，写指针放入元素的时候才加1
    
    // 这样不需要额外的空间，只需要在原有的数组上操作就可以了
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
                 
        // 遍历整个数组，如果c中包含该元素，则把该元素放到写指针的位置（以complement为准）
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
              
        // 正常来说r最后是等于size的，除非c.contains()抛出了异常
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                 
        // 如果c.contains()抛出了异常，则把未读的元素都拷贝到写指针之后
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                           
        // 将写指针之后的元素置为空，帮助GC
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
         // 新大小等于写指针的位置（因为每写一次写指针就加1，所以新大小正好等于写指针的位置）
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```
1.遍历elementData数组；

2.如果元素在c中，则把这个元素添加到elementData数组的w位置并将w位置往后移一位；

3.遍历完之后，w之前的元素都是两者共有的，w之后（包含）的元素不是两者共有的；

4.将w之后（包含）的元素置为null，方便GC回收；

### removeAll(Collection c)
求两个集合的单方向差集，只保留当前集合中不在c中的元素，不保留在c中不在当前集体中的元素。

```
 public boolean removeAll(Collection<?> c) {
        
     // 集合c不能为空
        Objects.requireNonNull(c);
         
     // 同样调用批量删除方法，这时complement传入false，表示删除包含在c中的元素
        return batchRemove(c, false);
    }
```
与retainAll(Collection c)方法类似，只是这里保留的是不在c中的元素。

## 总结

1.ArrayList内部使用数组存储元素，当数组长度不够时进行扩容，每次加一半的空间，ArrayList不会进行缩容；

2.ArrayList支持随机访问，通过索引访问元素极快，时间复杂度为O(1)；

3.ArrayList添加元素到尾部极快，平均时间复杂度为O(1)；

4.ArrayList添加元素到中间比较慢，因为要搬移元素，平均时间复杂度为O(n)；

5.ArrayList从尾部删除元素极快，时间复杂度为O(1)；

6.ArrayList从中间删除元素比较慢，因为要搬移元素，平均时间复杂度为O(n)；

7.ArrayList支持求并集，调用addAll(Collection c)方法即可；

8.ArrayList支持求交集，调用retainAll(Collection c)方法即可；

9.ArrayList支持求单向差集，调用removeAll(Collection c)方法即可;

    优点：

    因为其底层是数组，所以修改和查询效率高。

    可自动扩容（1.5倍）。

    缺点：

    插入和删除效率不高。

    线程不安全
    
    
##  最后最后：手写阉割版 ArrayList

```
public class MyArrayList {

    // 非私有，以简化嵌套类访问
    // transient 在已经实现序列化的类中，不允许某变量序列化
    transient Object[] elementData;

    //默认容量
    private static final int DEFAULT_CAPACITY = 10;

    // 用于空实例的 空数组实例
    private static final Object[] EMPTY_ELEMENTDATA = {};

    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // 实际ArrayList集合大小
    private int size;
    /**
     * 构造方法
     */
    public MyArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                    initialCapacity);
        }
    }
    public MyArrayList(){
        this(DEFAULT_CAPACITY);
    }

    public void add(Object o){
        //1. 判断数据容量是否大于 elementData
        ensureExplicitCapacity(size+1);
        //2. 使用下标进行赋值
        elementData[size++] = o;
    }
    private void ensureExplicitCapacity(int minCapacity){
        if (size == elementData.length){
            // 需要扩容,扩容1.5倍(ArrayList默认扩容1.5倍)
            // 注意：如果oldCapacity值为1
            int oldCapacity = elementData.length;
            int newCapacity = oldCapacity + (oldCapacity >> 1);
            // 如果新容量 < 最小容量， 则将最小容量赋值给新容量
            // 如果 oldCapacity=1, 则 minCapacity=1+1=2   newCapacity=1+(1>>1)=1
            if (newCapacity - minCapacity < 0){
                newCapacity = minCapacity;
            }
            // 创建新数组
            Object[] objects = new Object[newCapacity];
            // 将数据复制给新数组
            System.arraycopy(elementData, 0, objects, 0, elementData.length);
            // 修改引用
            elementData = objects;
        }
    }
    public Object get(int index) {
        rangeCheck(index);
        return elementData[index];
    }
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException("下标越界");
    }

    /**
     * 通过下标删除
     * @param index
     * @return
     */
    public Object remove(int index) {
        rangeCheck(index);

//        modCount++;
        // 先查出元素
        Object oldValue = elementData[index];
        // 找出置换结束位置
        int numMoved = size - index - 1;
        if (numMoved > 0)
            // 从 index+1 开始 将值覆盖为 index-numMoved 的值
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
    public boolean remove(Object o) {
        for (int index = 0; index < size; index++){
            if (o.equals(elementData[index])) {
                remove(index);
                return true;
            }
        }
        return false;
    }
}
```