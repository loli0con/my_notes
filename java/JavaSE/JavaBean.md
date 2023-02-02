# JavaBean

## 定义
如果一个Java类的每个实例变量都被使用private修饰，并为每个实例变量都提供了public修饰的setter方法（写方法）和getter方法（读方法），那么这个类就是一个符合JavaBean规范的类。

## 推荐阅读
1. [Java bean 是个什么概念？](https://www.zhihu.com/question/19773379)
2. [JavaBean-菜鸟教程](https://www.runoob.com/jsp/jsp-javabean.html)
3. [JavaBean-廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1260474416351680)

## boolean字段
boolean字段比较特殊，它的读方法一般命名为isXyz()：
```Java
// 读方法:
public boolean isChild()
// 写方法:
public void setChild(boolean value)
```

## JavaBean的作用
JavaBean主要用来传递数据，即把一组数据组合成一个JavaBean便于传输。此外，JavaBean可以方便地被IDE工具分析，生成读写属性的代码，主要用在图形界面的可视化设计中。

## 枚举JavaBean属性
要枚举一个JavaBean的所有属性，可以直接使用Java核心库提供的Introspector：
```Java
public class Main {
    public static void main(String[] args) throws Exception {
        BeanInfo info = Introspector.getBeanInfo(Person.class);
        for (PropertyDescriptor pd : info.getPropertyDescriptors()) {
            System.out.println(pd.getName());
            System.out.println("  " + pd.getReadMethod());
            System.out.println("  " + pd.getWriteMethod());
        }
    }
}

class Person {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```