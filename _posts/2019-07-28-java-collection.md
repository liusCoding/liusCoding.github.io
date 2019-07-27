---
layout: post
title: java容器分析介绍
categories: Java
description: Collection
keywords: Java
---

## 1.为什么需要容器？
通过，程序总是在运行时才能确定要创建对象的数量，甚至是对象的类型。
为了解决这个问题，需要在任意时刻位置创建任意数量的对象。大多数语言都提供某种方法来解决这个问题，java使用容器来解决这个问题。
容器也称集合类，基本的类型是 List,Set,Queue，Map,但由于Java 类库中使用了 Collection 关键字作接口。
所以一般用容器来称呼这些集合类。java容器工具的jar包是java.util.*。

## 2.容器的整体结构图
Java 容器主要可以划分为5个部分，List,Set，Map,Iterator，Enumeration枚举类、Arrays和Collections。


### Collection的类层次结构图


**简洁图**
![java-collection-hierarchy.jpg](https://www.bysocket.com/wp-content/uploads/2015/03/java-collection-hierarchy.jpg)

**详细图**
![Set_thumb.jpg](http://www.bysocket.com/wp-content/uploads/2015/03/Set_thumb.jpg)


## 3.List，Set，Queue，Map介绍

### List介绍

**List结构图**
![listTaxonomy.png](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/Java8-util/listTaxonomy.png)
<br>
**List介绍**<br>
一个**有序** Collection ,元素可以重复,有序存放。确切的说，列表通常允许满足 object.equals(object2)。实现 List 接口有：ArrayList,LinkedList,Vector,Stack等。

### Set介绍

**Set 结构图**
![setTaxonomy.png](https://gitee.com/chenssy/blog-home/raw/master/image/series-images/Java8-util/setTaxonomy.png)

**Set** 介绍
一个**不允许重复元素**的 collection ,是一种无序的集合。Set 不可以 object.equals(object2)。实现Set的接口有：EnumSet，HashSet,TreeSet等。

### Queue介绍

**Queue 结构图**
![1182892-20171122100317930-842768608.png](https://images2017.cnblogs.com/blog/1182892/201711/1182892-20171122100317930-842768608.png)
<br/>
**介绍 Queue**
queue 是用来模拟队列这种数据结构，队列通常是指“**先进先出**”的容器。新元素插入到队列的尾部，访问元素操作会返回队列头部的元素。
通常队列不允许随机访问队列中的元素，这种结构就像我们生活中的排队一样。

### Map介绍
**Map 结构图**
![MapClassHierarchy-600x354_thumb.jpg](http://www.bysocket.com/wp-content/uploads/2015/03/MapClassHierarchy-600x354_thumb.jpg)
**Map 介绍**
Map 是一个键值对的集合。它的每一个元素都包含一对键对象和值对象，每个键最多映射到一个值。它没有继承Collection接口。
实现 Map 的有：HashMap，TreeMap，HashTable,Properties,EnumMap。




