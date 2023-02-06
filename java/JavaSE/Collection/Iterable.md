# Iterable
![Iterable+20230206153924](https://raw.githubusercontent.com/loli0con/picgo/master/images/Iterable%2B20230206153924.png%2B2023-02-06-15-39-25)

Collection 继承了 Iterable 接口，其中的 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。

## 遍历
有如下方式去遍历集合：
* Iterable.forEach方法
* Iterable.iterator方法，获取一个迭代器（*往后&能删*）
  * 使用迭代器的hasNext、next、remove等方法
  * 使用迭代器的forEachRemaining方法
* for/forEach循环
* Collection.stream方法，StreamAPI

[迭代方式的总结和比较](https://juejin.cn/post/6863108420552261646)