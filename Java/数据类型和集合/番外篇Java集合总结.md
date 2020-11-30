关于Vector，它的实现和ArrayList非常类似，就不再单开一个章节来讲了，现在我们来对Java集合做一个归纳总结。

## 一. List框架图

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/30/08:55:30-1677914-20190630110527653-156301420.png)

首先上面的框架图可以表明顺序的关联关系，但并不全面，如ArrayList在继承了AbstractList抽象类的同时还实现了List接口。

1. List是一个接口，继承了Collection，同时Collection继承了Iterable，表明List的实现类都是可用迭代遍历的；
2. AbstractList是一个抽象类，实现了List接口，同时继承了AbstractCollection，针对一些常用方法，如add()，set()，remove()，给了默认实现，当然在具体的实现类中基本都重写了，该类中没有get()，size()方法。
3. AbstractSequentialList是一个抽象类，继承了AbstractList抽象类，实现了很多双向链表中根据索引操作的方法。
4. ArrayList、Vector、LinkedList、Stack都是具体的实现类。

## 二. ArrayList、Vector对比分析

| 类型      | 线程安全 | 内部结构     |      | 扩容规则                                     | 执行效率 | 序列化 |
| --------- | -------- | ------------ | ---- | -------------------------------------------- | -------- | ------ |
| ArrayList | 否       | 数组Object[] | 10   | 数组足够最小长度*1.5                         | 高       | 是     |
| Vector    | 是       | 数组Object[] | 10   | 默认数组足够最小长度*2，可自定义每次扩容数量 | 低       | 是     |

Vertor扩容方法：

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //capacityIncrement参数可通过构造函数传递进来，若没传递该参数，则数组大小设置为elementData.length * 2
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //扩容有上限
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

## 三. ArrayList、LinkedList对比分析

| 类型       | 内部结构     | 插入效率(正常情况) | 删除效率(正常情况) | 顺序遍历效率 | 随机遍历效率 | 占用内存 | 序列化 |
| ---------- | ------------ | ------------------ | ------------------ | ------------ | ------------ | -------- | ------ |
| ArrayList  | 数组Object[] | 低                 | 低                 | 高           | 高           | 低       | 是     |
| LinkedList | 双向链表Node | 高                 | 高                 | 高           | 低           | 高       | 是     |

上述的对比都是基于大数据量的情况下，如果只是几个元素或几十个元素，它们之间并没有多大区别。

------

问：插入效率为何说正常情况下ArrayList低，LinkedList高呢？

答：我们清楚ArrayList之所以插入效率低，有两个原因会造成时间的消耗。

 第一，当底层数组空间不足时需要扩容，扩容后需进行数组拷贝

 第二，当不在数组末尾插入数据，那么就需要移动数组元素

 知道了其插入效率低的原因后，那么很明显，数据扩容及拷贝只有在数组空间不足时才发生，如果我们正确使用，就像《阿里巴巴Java开发手册》中提到我们在创建集合对象时，就传递参数预先设置好数组大小，那么插入效率是非常高的；而90%的情况下我们在添加元素时都调用的是add(E e)，直接在末尾添加元素，很少调用add(int index, E e)在数组中部添加元素，这样其实移动数组元素就很少发生，因此插入效率也很高。

------

问：删除效率为何说正常情况下ArrayList低，LinkedList高呢？

答：因为删除效率高、低不是绝对的。其实删除操作可以分为两部分。

 第一：找到要删除的元素，这个通过索引找，ArrayList的执行效率要远高于LinkedList的执行效率；通过equals找则需要遍历整个集合，ArrayList和LinkedList执行效率基本一致。

 第二：删除元素及后续操作，这个如果删除是最后一个元素，执行效率基本一致；如果是删除的中间元素，那么ArrayList需进行数组元素移动，而LinkedList只需搭建起该元素的上一个节点和下一个节点的关系即可，LinkedList执行效率高于ArrayList。

 因此，需根据实际情况才可判断实际的执行效率。

------

问：遍历效率这个问题怎么说？

答：ArrayList通过数组实现，天然可以通过数组下标读取数据，顺序遍历、随机遍历效率都非常高；LinkedList通过双向链表实现，顺序遍历时，可直接通过本节点.next()直接找到相关联的下一个节点，效率很高，而如果LinkedList随机遍历时，首先需判断（传递的索引值与集合长度/2）的大小，来确定接下来是应该从第一个节点开始找还是最后节点开始找，越是靠近集合中部、集合越大，随机遍历执行效率越低。

------



## 四. 总结

本文对List集合进行了总结，包括类结构图，ArrayList和Vector对比分析，ArrayList和LinkedList的对比分析，若有不对之处，请批评指正，望共同进步，谢谢！