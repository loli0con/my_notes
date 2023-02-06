# 集合
![Collection+Collection](https://raw.githubusercontent.com/loli0con/picgo/master/images/Collection%2BCollection.png%2B2021-06-01-17-54-35)

## Set
* TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
* HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
* LinkedHashSet：具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

## List
* ArrayList：基于动态数组实现，支持随机访问。
* Vector：和 ArrayList 类似，但它是线程安全的。
* LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

## Queue
* LinkedList：可以用它来实现双向队列。
* PriorityQueue：基于堆结构实现，可以用它来实现优先队列。




## 推荐阅读
* [ArrayList源码+扩容机制分析](https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/ArrayList%E6%BA%90%E7%A0%81%2B%E6%89%A9%E5%AE%B9%E6%9C%BA%E5%88%B6%E5%88%86%E6%9E%90.md)
* [streamAPI-入门](https://www.liaoxuefeng.com/wiki/1252599548343744/1322402873081889)
* [streamAPI-进阶](https://colobu.com/2016/03/02/Java-Stream/)
* [Java 集合框架综述](https://segmentfault.com/a/1190000023520835)
