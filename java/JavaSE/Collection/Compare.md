# Compare

## Comparable
Java提供了一个Comparable接口，该接口里定义了一个compareTo(Object obj)方法，该方法返回一个整数值，实现该接口的类必须实现该方法，实现了该接口的类的对象就可以比较大小。当一个对象调用该方法与另一个对象进行比较时，例如obj.compareTo(obj2)，如果该方法返回0，则表明这两个对象相等；如果该方法返回一个正整数，则表明obj1大于obj2；如果该方法返回一个负整数，则表明obj1小于obj2。

```Java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

## Comparator
Comparator是一个函数式接口，里面包含一个int compare(T o1,T o2)方法，该方法用于比较o1和o2的大小；如果方法返回正整数，则表明o1大于o2；如果该方法返回0，则表明o1等于o2；如果该方法返回负整数，则表明o1小于o2。

```Java
public interface Comparator<T> {
    int compare(T o1, T o2);
    ...
}
```