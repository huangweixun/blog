---
title: ArrayList源码解析
date: 2018-11-07 14:49:37
tags: [ArrayList,动态数组]
---

​	ArrayList内部维护了一个动态再分配的Object[]数组，每一个类对象都有一个capacity属性，表示它们所封装的Object[]数组的长度，当向ArrayList中添加元素时，该属性值会自动增加。如果想ArrayList中添加大量元素，可使用ensureCapacity方法一次性增加capacity，可以减少增加重分配的次数提高性能。

<!-- more -->

### add

~~~java
public void add(int index, E element) {
		//检查索引是否越界
        rangeCheckForAdd(index);
		//将集合的capacit增加minCapacity
        ensureCapacityInternal(size + 1); 
       
        //从指定源数组中复制一个数组，复制从指定的位置开始，到目标数组的指定位置结束。
        //arraycopy(被复制的数组, 从第几个元素开始复制, 要复制到的数组, 从第几个元素开始粘贴, 一共需要复制的元素个数)
        //即在数组elementData从index位置开始，复制到index+1位置,共复制size-index个元素
        System.arraycopy(elementData, index, elementData, index + 1,size - index);
        elementData[index] = element;
        size++;
    }
~~~



1. 检查索引是否越界。
2. 判断添加一个元素之后是否需要扩容，如果需要扩容 `ensureCapacityInternal(size + 1)`内部会调用`grow()`进行扩容，详解请参考**扩容**  。
3. 将index位置元素，复制到index+1位置，后面的元素类推，既从index位置开始全部后移一个位置。
4. 将需要添加的元素添加到index位置。

### get

~~~java
    //返回至指定索引的值
    public E get(int index) 
    {
        rangeCheck(index); //检查给定的索引值是否越界
        return elementData(index);
    }

    E elementData(int index) {
        return (E) elementData[index];
    }
~~~

由于内部是数组，所以在获取指定位置的时间复杂度为O(1)。

remove

~~~java
    public E remove(int index)
    {
		//检查索引是否越界
        rangeCheck(index);
        modCount++;
        E oldValue = elementData(index);
        int numMoved = size - index - 1;
        if (numMoved > 0)
			//arraycopy(被复制的数组, 从第几个元素开始复制, 要复制到的数组, 从第几个元素开始粘贴, 一共需				要复制的元素个数)
            System.arraycopy(elementData, index+1, elementData, index,numMoved);
        elementData[--size] = null; //将原数组最后一个位置置为null
        return oldValue;
    }
~~~

1. 将index+1位置元素，复制到index位置，后面的元素类推，既从index位置开始全部前移一个位置。
2. 原数组最后一个位置设置为null

### 扩容

ArrayList每次调用add方法的时候，会对容量进行检查：`ensureCapacityInternal()`  。

~~~java
   //得到最小扩容量
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取默认的容量和传入参数的较大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
~~~

` ensureExplicitCapacity()`这个方法在`ensureCapacityInternal()`一定会调用。

~~~java
  	//判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }
~~~

该方法会判断最小容量是否大于现有数组长度，来决定是否调用`grow()`。

所以我们的扩容的核心代码其实是在`grow()`

~~~java
    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
       // 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
       //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
~~~

其实最核心代码为`int newCapacity = oldCapacity + (oldCapacity >> 1)`，通过位运算，新的容量更新为旧容量的1.5倍。确定扩容大小之后，调用`elementData = Arrays.copyOf(elementData, newCapacity)`，Arrays.copyOf()方法的作用是新建一个数组并将原数据赋值，最后将该数组返回。

