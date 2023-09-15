# Scala

## 注释
1. 单行注释: `//`
2. 多行注释: `/* */`
3. 文档注释: 
```
/**
  *
  */
```

## 变量和常量
基本语法：
1. var 变量名 [: 变量类型] = 初始值
2. val 常量名 [: 常量类型] = 初始值

使用规则：
1. 声明变量时，类型可以省略，编译器自动推导，即类型推导
2. 类型确定后，就不能修改
3. 变量声明时，必须要有初始值
4. 在声明/定义一个变量时，可以使用 var 或者 val 来修饰，var 修饰的变量可改变，val 修饰的变量不可改
5. var 修饰的对象引用可以改变，val 修饰的对象则不可改变，但对象的状态(值)却是可以改变的

## 标识符的命名规范
Scala 对各种变量、方法、函数等命名时使用的字符序列称为标识符。

命名规则：
1. 以字母或者下划线开头，后接字母、数字、下划线
2. 以操作符开头，且只包含操作符(+ - * / # !等)
3. 用反引号`....`包括的任意字符串，即使是 Scala 关键字(39个)也可以

## 字符串输出
1. 字符串，通过+号连接
2. printf 用法:字符串，通过%传值。
3. 字符串模板(插值字符串):通过$获取变量值

```scala
var name: String = "pig"
var age: Int = 18


//(1)字符串，通过+号连接
println(name + " " + age)


//(2)printf 用法字符串，通过%传值。
printf("name=%s age=%d\n", name, age)


//(3)字符串模版，通过$引用

//(3.1)多行字符串
//在 Scala 中，利用三个双引号包围多行字符串就可以实现。
//输入的内容，带有空格、\t 之类，导致每一行的开始位置不能整洁对齐。
//应用 scala 的 stripMargin 方法，在 scala 中 stripMargin 默认是“|”作为连接符，
//在多行换行的行头前面加一个“|”符号即可。
val s =
    """
    |select
    |   name,
    |   age
    |from user
    |where name="zhangsan"
    """.stripMargin
println(s)

//(3.2)多行字符串+$变量取值
//如果需要对变量进行运算，那么可以加${}
val s1 =
    s"""
    |select
    |   name,
    |   age
    |from user
    |where name="$name" and age=${age+2}
    """.stripMargin
println(s1)

//(3.2)普通字符串+$变量取值
val s2 = s"name=$name"
println(s2)
```

## 数据类型
1. Scala中一切数据都是**对象**，都是Any的子类。
2. Scala中数据类型分为两大类: 数值类型(AnyVal)、引用类型(AnyRef)，不管是值类型还是引用类型都是**对象**。
3. Scala数据类型支持自动转换(隐式转换)，低精度的值类型向高精度值类型。
4. Scala中的StringOps是对Java中的String增强。
5. Unit: 对应Java中的void，用于方法返回值的位置，表示方法没有返回值。Unit是一个数据类型，只有一个对象 就是()。
6. Null是一个类型，只有一个对象就是null。它是所有引用类型(AnyRef)的子类。
7. Nothing，是所有数据类型的子类，主要用在一个函数没有明确返回值时使用。

![Scala+20221123141145](https://raw.githubusercontent.com/loli0con/picgo/master/images/Scala%2B20221123141145.png%2B2022-11-23-14-11-48)