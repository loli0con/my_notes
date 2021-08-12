# 映射
![Map+Map](https://raw.githubusercontent.com/loli0con/picgo/master/images/Map%2BMap.png%2B2021-06-01-23-22-13)
## Map
Map用于保存具有映射关系的数据，因此Map集合里保存着两组值，一组保存key，另一组保存value。  
key和value都可以是任何引用类型的数据，key不允许重复，即同一个Map对象的任何两个key通过equals方法比较总是返回flase。  
key和value之间存在单向一对一关系，即通过指定的key，总能找到唯一的、确定的value。  
Map有时也被成为字典或关联数组。

Map提供了一个Entry内部类来封装key-value对，而计算Entry存储规则时则只考虑Entry封装的key。

### Map 和 Set 的联系
Java源码是先实现了HashMap、TreeMap等集合，然后通过包装一个所有value都为null的Map集合实现了Set集合类。

## HashMap 和 Hashtable
标题没打错，就是`Hashtable`，过时了不推荐。

### HashMap
[HashMap原理和内部存储结构](https://juejin.cn/post/6844903763715555336)

HashMap保存key的方式与HashSet保存集合元素的方式完全相同，所以HashMap对key的要求与[HashSet对集合元素的要求](Collection.md#hashcode-%E5%92%8C-equals)完全相同。

HashMap通过两个对象的equals()方法的返回值判断两个value是否相等。

## LinkedHashMap
LinkedHashMap使用双向链表来维护key-value对的次序，该链表负责维护Map的迭代顺序，迭代顺序与key-value对的插入顺序保存一致。因为使用链表来维护内部顺序，所以在迭代访问Map的全部元素时将有较好的性能。

## Properties
Properties类是Hashtable类的子类，用于处理属性文件。

## SortedMap接口 和 TreeMap实现类
Map接口派生出了一个SortedMap子接口，SortedMap接口有一个TreeMap实现类。

TreeMap就是一个红黑树数据结构，每个key-value对即作为红黑树的一个节点。  
TreeMap存储key-value对（节点）时，需要根据key对节点进行排序，保证所有的key-value处于有序状态，有两种排序方式：自然排序 和 定制排序。

TreeMap中判断两个key相等的标准是：两个key通过compareTo()方法返回0，即认为这两个key是相等的。

与TreeSet类似，注意要让两个key通过equals()方法比较返回true时，它们通过compareTo()方法比较也应该返回0。
