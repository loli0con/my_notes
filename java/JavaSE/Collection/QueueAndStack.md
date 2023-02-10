# 队列/栈

## Queue
Queue用于模拟**队列**这种数据结构——“**先进先出**（FIFO）”的容器，Queue只有两个操作：
1. 把元素添加到队列末尾；
2. 从队列头部取出元素。

在Java的标准库中，队列接口Queue定义了以下几个方法：
1. int size()：获取队列长度；
2. boolean add(E)/boolean offer(E)：添加元素到队尾；
3. E remove()/E poll()：获取队首元素并从队列中删除；
4. E element()/E peek()：获取队首元素但并不从队列中删除。

添加、删除和获取队列元素总是有两个方法，这是因为在添加或获取元素失败时，这两个方法的行为是不同的：
||throw Exception|返回false或null|
|---|---|---|
|添加元素到队尾|add(E e)|boolean offer(E e)|
|取队首元素并删除|E remove()|E poll()|
|取队首元素但不删除|E element()|E peek()|

注意：不要把null添加到队列中，否则poll()方法返回null时，很难确定是取到了null元素还是队列为空。

## Priority
**优先队列**，根据队列元素的“优先级”进行重新排序，返回的总是优先级最高的元素。

PriorityQueue实现了一个优先队列：从队首获取元素时，总是获取优先级最高的元素。要使用PriorityQueue，必须给每个元素定义“优先级”，要么让元素实现Comparable接口，要么在创建PriorityQueue时传入Comparator。

PriorityQueue不可插入null元素，toString()方法返回值不可直接利用。

## Deque
Deque接口是Queue接口的子接口，它代表一个**双端队列**（Double Ended Queue），它里面定义了一些双端队列的方法：
* 将元素添加到队尾或队首：addLast()/offerLast()/addFirst()/offerFirst()；
* 从队首／队尾获取元素并删除：removeFirst()/pollFirst()/removeLast()/pollLast()；
* 从队首／队尾获取元素但不删除：getFirst()/peekFirst()/getLast()/peekLast()；
* 总是调用xxxFirst()/xxxLast()以便与Queue的方法区分开；
* 避免把null添加到队列。

### ArrayDeque
ArrayDeque是Deque接口典型的实现类，它是基于数组实现的双端队列，创建Deque时可指定一个numElement参数，用于指定Object[]的长度，默认为16。

## 栈
栈（Stack）是一种后进先出（LIFO：Last In First Out）的数据结构。Stack只有入栈和出栈的操作：
* 把元素压栈：push(E)；
* 把栈顶的元素“弹出”：pop()；
* 取栈顶元素但不弹出：peek()。

使用Deque可以实现Stack的功能，注意只调用push()/pop()/peek()方法，避免调用Deque的其他方法。不要使用遗留类`Stack`。

## LinkedList
LinkedList是一个全能选手，它即是List，又是Queue，还是Deque。在使用的时候，总是用特定的接口来引用它，这是因为持有接口说明代码的抽象层次更高，而且接口本身定义的方法代表了特定的用途。