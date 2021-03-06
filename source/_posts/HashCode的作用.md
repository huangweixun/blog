---
title: HashCode的作用
date: 2018-11-07 18:02:19
tags: hashcode
---



<!-- more -->

### Hash

​	首先什么是Hash呢？Hash是散列的意思，就是把任意长度的输入，通过散列算法转换成固定长度的输出，该输出就是散列值。

​	不同的输入经过散列算法计算之后，可能得到相同的输出，这个就叫做碰撞。

​	如果两个输入的Hash值不同，那么这两个输入一定不同。

### HashCode的特性

1. HashCode的存在主要是用于**查找的快捷性**，如Hashtable，HashMap等
2. 如果两个对象相同， equals方法一定返回true，并且这两个对象的HashCode一定相同；
3. 两个对象的HashCode相同，并不一定表示两个对象就相同，即equals()不一定为true，只能够说明这两个对象在一个散列存储结构中。
4. 如果对象的equals方法被重写，那么对象的HashCode也尽量重写。

### HashCode有什么作用

​	首先我们先抛出一个问题：假设内存中有0到9十个位置，并每个位置都存放值，这时我们要查找一个值的时候，必须对所有的位置进行遍历查询。

​	那么如果使用HashCode可以怎么做呢？我们使用模运算，即hashcode%length，将值存放在取余的位置，那么这个时候我们要查找一个值的话，就不用遍历查询，直接hashcode%length找到存放的位置即可。

​	如果计算出来的位置有了数据怎么办？这时候就是把该数据添加到已有数据的后面，从而形成一个链表结构。

​	如果hashCode相同了，这时候就需要定义`equals()`。首先通过模运算确定位置，再通过equals来最终确定要找的值。

### 为什么重写equals()的时候必须重写HashCode()

首先，官方规定的equals与hashcode的关系是这样的：

~~~
如果两个对象相同（即用equals比较返回true），那么它们的hashCode值一定要相同；
如果两个对象的hashCode相同，它们并不一定相同(即用equals比较返回false)   
~~~

​	那么文章开头也说了HashCode的特性，他主要用于**查找的快捷性**，而真正判断两个对象是否相同的依据还是equals()方法。

​	所以为了确保这一规定，两者必须同时重写，否则可能会出现equals()相同但是HashCode()不同的情况。

### HashCode的应用场景

1. 比较对象的时候，重写HashCode和equals。
2. HashMap等集合在添加、查询的时候，会根据hashcode%length来确定保存在数组哪个位置。

### 总结

​	HashCode的最终目的就是为了**查询的快捷性**

