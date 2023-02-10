# Iterable
![Iterable+20230206153924](https://raw.githubusercontent.com/loli0con/picgo/master/images/Iterable%2B20230206153924.png%2B2023-02-06-15-39-25)

## 迭代器
Iterator是一种抽象的数据访问模型，使用Iterator模式进行迭代的好处有：
* 对任何集合都采用同一种访问模型；
* 调用者对集合内部结构一无所知；
* 集合类返回的Iterator对象知道如何迭代。

Java提供了标准的迭代器模型，即集合类实现java.util.Iterable接口，返回java.util.Iterator实例。

## 遍历集合
Collection 继承了 Iterable 接口，其中的 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。Iterator 对象是集合对象自己在内部创建的，它自己知道如何高效遍历内部的数据集合。编译器能把标准的for...each循环自动转换为Iterator遍历。

```Java
// 集合
List<String> list = List.of("Apple", "Orange", "Pear");

// for...each语法
for (String s : list) {
    System.out.println(s);
}

// 编译器转换结果
for (Iterator<String> it = list.iterator(); it.hasNext(); ) {
     String s = it.next();
     System.out.println(s);
}
```

有如下方式去遍历集合：
* Iterable.forEach方法
* Iterable.iterator方法，获取一个迭代器（**在遍历的同时能删除元素**）
  * 使用迭代器的hasNext、next、remove等方法
  * 使用迭代器的forEachRemaining方法
* for/forEach循环
* Collection.stream方法，StreamAPI

[迭代方式的总结和比较](https://juejin.cn/post/6863108420552261646)