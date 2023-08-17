# SOLID原则

SOLID是以下是原则的缩写：
* Single Responsibility Principle 单一职责原则
* Open Closed Principle 开闭原则
* Liskov Substitution Principle 里氏替换原则
* Interface Segregation Principle 接口隔离原则
* Dependency Inversion Principle 依赖倒置原则



## 坏的设计

### 表现
* 僵硬性
  * 难以更改代码
  * 从管理角度，拒绝任何的变化成为一种制度
* 易碎性
  * 使是小小的改动也会导致级联性的后果的
  * 代码在意想不到的地方终止
* 固定性
  * 代码纠缠在一起根本不可能重用（高度的耦合）
* 黏滞性
  * 宁愿重新编写也不愿意修改

### 原因
* 需求不停地变化
* 设计的问题:“依赖管理”失衡

软件系统在其生命周期都在发生变化，无论是好的设计还是坏的设计，都面临着这个问题，但好的设计是稳定的。



## 开闭原则 - Open Closed Principle
![SOLID原则+20220314144517](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314144517.png%2B2022-03-14-14-45-17)

### 定义
软件系统应当允许功能扩展(即开放性)，但是不允许修改原有的代码(即关闭性)。

* Open for Extension – 可以扩展行为以满足新的需求
* CLosed for Modification – 但不允许修改模块的源代码

### 做法
接口和抽象类可以做到

![SOLID原则+20220314134715](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314134715.png%2B2022-03-14-13-47-21)

![SOLID原则+20220314134839](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314134839.png%2B2022-03-14-13-48-41)

### 启发
1. 定义所有的对象 - 数据为私有的
2. 不要使用全局变量 

修改公有的数据，经常冒着“打开”模块的风险，它们常常会引起涟漪效应，导致许多地方连锁修改。很难找全出错的地方并修改，一处修改会导致多处又出问题。



## 依赖倒置原则 - Dependency Inversion Principle
![SOLID原则+20220314144642](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314144642.png%2B2022-03-14-14-46-43)

### 定义
1. 高层模块不应当依赖低层模块，两者都依赖抽象
2. 抽象不能依赖细节，细节应当依赖抽象

### 过程化程序架构 和 面向对象系统架构
传统的面向过程的程序设计，以功能划分系统。高层模块是业务/应用规则，低层模块是对这些规则的实现，高层模块完全依赖调用低层模块提供的功能来完成自己的功能。因此，高层依赖低层。

![SOLID原则+20220314140958](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314140958.png%2B2022-03-14-14-10-00)

### 启发

#### 1
面向接口设计，而不是面向实现设计
![SOLID原则+20220314141412](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314141412.png%2B2022-03-14-14-14-17)

为什么面向接口设计？因为：
1. 抽象类/接口修改的概率偏低
2. 抽象概念容纳的范围广，易于扩展/修改
3. 不应当修改代表抽象的类/接口，符合OCP原则

#### 2
避免传递性依赖
![SOLID原则+20220314141548](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314141548.png%2B2022-03-14-14-15-54)

#### 3
每当有疑虑时，增加一个间接层

##### 例子
![SOLID原则+20220314143626](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314143626.png%2B2022-03-14-14-36-27)

![SOLID原则+20220314143650](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314143650.png%2B2022-03-14-14-36-53)

![SOLID原则+20220314143703](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314143703.png%2B2022-03-14-14-37-04)



## 单一职责原则 - Single Responsibility Principle
![SOLID原则+20220314144501](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314144501.png%2B2022-03-14-14-45-01)

### 定义
单一职责原则的描述是一个 class 应该只做一件事，一个 class 应该只有一个变化的原因。

更技术的描述该原则：应该只有一个软件定义的潜在改变（数据库逻辑、日志逻辑等）能够影响 class 的定义。

这意味着如果 class 是一个数据容器，比如 Book class 或者 Student class，考虑到这个实体有一些字段，应该只有我们更改了数据定义时才能够修改这些字段。



## 里氏替换原则
![SOLID原则+20220314144538](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314144538.png%2B2022-03-14-14-45-39)

### 定义
里氏替换原则描述的是子类应该能替换为它的基类。

意思是，给定 class B 是 class A 的子类，在预期传入 class A 的对象的任何方法传入 class B 的对象，方法都不应该有异常。

这是一个预期的行为，因为继承假定子类继承了父类的一切。子类可以扩展行为但不会收窄。

因此，当 class 违背这一原则时，会导致一些难于发现的讨厌的 bug。



## 接口隔离原则
![SOLID原则+20220314144604](https://raw.githubusercontent.com/loli0con/picgo/master/images/SOLID%E5%8E%9F%E5%88%99%2B20220314144604.png%2B2022-03-14-14-46-05)

### 定义
隔离意味着保持独立，接口隔离原则是关于接口的独立。

该原则描述了很多客户端特定的接口优于一个多用途接口。客户端不应该强制实现他们不需要的函数。