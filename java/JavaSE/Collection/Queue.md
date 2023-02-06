# Queue
Queue用于模拟**队列**这种数据结构——“**先进先出**（FIFO）”的容器。


## Priority
**优先队列**，根据队列元素的大小进行重新排序，**最小的元素先出队**。不可插入null元素，toString()方法返回值不可直接利用。

有两种排序方式（同TreeSet）：自然排序 和 定制排序。


## Deque
Deque接口是Queue接口的子接口，它代表一个**双端队列**，它里面定义了一些双端队列的方法。

Deque可以当成**栈**来使用。


### ArrayDeque
ArrayDeque是Deque接口典型的实现类，它是基于数组实现的双端队列，创建Deque时可指定一个numElement参数，用于指定Object[]的长度，默认为16。

### LinkedList
LinkedList实现了List接口（可以根据索引来随机访问集合中的元素），实现了Deque接口（可以当成双端队列来使用）。
