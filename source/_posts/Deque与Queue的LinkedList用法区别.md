---
title: Deque与Queue的LinkedList用法区别
date: 2023-06-11 16:56:53
tags:
---

# Deque与Queue的LinkedList用法区别

## 队列和双端队列的比较

声明一个队列可以使用以下代码：

 ```
    Queue queue = new LinkedList();//单向
    Deque queue = new LinkedList();//双端队列
 ```

### 队列

`Queue` 是 Java 中的一个接口，它定义了以先进先出（FIFO）的顺序处理元素的操作。

`LinkedList`实现了`Queue`接口，意味着可以通过`LinkedList`来实例化一个队列。

```
    queue.add(e) 或 offer(e) //将元素添加到队列的末尾
    remove() 或 poll() //从队列的头部移除并返回元素
    element() 或 peek() //返回队列头部的元素
    size()
    .isEmpty()
```

  ### 双端队列

 `Deque`是一种双端队列，是`Queue`接口的一个子接口。它扩展了`Queue`接口并添加了一些额外的方法。双端队列是一种同时支持在前和在后插入、删除元素的线性数据结构。`LinkedList`同样也实现了`Deque`接口。

 ```
    deque.addLast(value);
    deque.removeLast();
    deque.peekLast();
 ```
