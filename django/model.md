# model
模型准确且唯一地描述了数据，它包含了被存储的数据的字段和行为。

## 基本概念
1. 每个模型都是一个python类，这些类继承django.db.models.Model。
2. 模型类的每个属性都相当于一个数据库的字段。
3. Django根据类定义自动生成访问数据库的API。

### 默认行为
* 表名默认为“*应用名*_*模型类名*”
* 如果没有主键，则**id**字段会被自动添加

#### 自动设置主键
如果没有显式地设置主键，则Django会隐式地在模型类中添加字段：

    id = models.AutoField(primary_key=True)

### 启用模型
1. 在INSTALLED_APPS里面添加*应用名*
2. manage.py makemigrations
3. manage.py migrate

## 字段
模型中的每一个字段都是某个Field类的实例。

### 功能
Django利用这些字段类实现以下功能：
* 字段类型用以指定数据库数据类型
* 在渲染表单字段时默认使用的HTML视图
* 基本有效性验证功能，用于Django后台和自动生成表单

### 内置字段类型
\# [TODO](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#model-field-types)
### 自定义字段类型
\# [TODO](https://docs.djangoproject.com/zh-hans/3.1/howto/custom-model-fields/)

## 字段选项
每一种字段都需要指定一些特定的参数。  
一些可选的参数是通用的，可以用于任何字段类型。

### 常见参数
null、blank、choices、default、help_tex、primary_key、unique

## 关联关系
Django 提供了定义三种最常见的数据库关联关系的方法：多对一，多对多，一对一。

### 多对一关联
定义一个多对一的关联关系，使用 django.db.models.ForeignKey 类。  
ForeignKey 类需要两个位置参数：关联的模型类名 和 on_delete 选项。

建议设置 ForeignKey 字段名为要关联的模型名（小写字母）。

### 多对多关联
定义一个多对多的关联关系，使用 django.db.models.ManyToManyField 类。  
ManyToManyField 类需要添加一个位置参数，即要关联的模型类名。

建议设置 ManyToManyField 字段名为一个复数名词（关联的模型名，小写字母，复数形式），表示所要光联的模型对象的集合。

对于多对多关联关系的两个模型，可以在任何一个模型中添加 ManyToManyField 字段，但只能选择一个模型设置该字段，即不能同时在两模型中添加该字段。一般来讲，应该把 ManyToManyField 实例放到需要在表单中被编辑的对象中。

#### 在多对多关系中添加添加额外的属性字段
Django 允许你指定用于控制多对多关系的模型，可以在中间模型当中添加额外的字段。
在实例化 ManyToManyField 的时候使用 through 参数指定多对多关系使用哪个中间模型。

##### 中间模型
在设置中间模型的时候，显式地为多对多关系中涉及的中间模型指定外键。这种显式声明定义了这两个模型之间是如何关联的。

在中间模型当中有一些限制条件：
* 中间模型要么有且仅有一个指向源模型（或者目标模型）的外键，要么必须通过 ManyToManyField.through_fields 参数在多个外键当中手动选择一个外键；否则会会出现验证错误。
* 自己指向自己的多对多关系的中间模型当中，可以有两个指向同一个模型的外健，但这两个外健分表代表多对多关系（不同）的两端。如果外健的个数超过两个，必须指定 through_fields 参数，要不然会出现验证错误。

#### 创建关系
创建关系的方式：
1. 直接实例化中间模型来创建关系
2. 通过源/目标模型，使用add/create/set方法创建关系（要确保为所有的必需字段指定through_defaults）

#### 删除关系
* 如果自定义中间模型没有强制 (model1, model2) 对的唯一性，调用 remove() 方法会删除所有中间模型的实例。
* 调用 clear() 方法会删除所有多对多关系。

*上述的方法的调用者都是某个源/目标模型的实例。*

### 一对一关联
使用 OneToOneField 来定义一对一关系。  
OneToOneField 需要一个位置参数：与模型相关的类。

#### 作用
最有用的是作为某种方式“扩展”另一个模型的主键：
当一个对象以某种方式“**继承**”另一个对象时，这对该对象的主键非常有用。

可选的 parent_link 参数：当 True 并用于从另一个 concrete model 继承的模型中时，表示该字段应被用作回到父类的链接。

> oncrete model：具体的模型，一个非抽象模型。

## 跨文件模型
在定义模型的文件开头导入需要被关联的模型，接着就可以在其他有需要的模型类当中关联它。

## 字段命名限制
Django 对模型的字段名有一些限制：
1. 一个字段的名称不能是 Python 保留字，因为这会导致 Python 语法错误。
2. 一个字段名称不能包含连续的多个下划线，原因在于 Django 查询语法的工作方式。
3. 字段名不能以下划线结尾，原因同上。

> 这些限制是可以被解决的

## Meta选项
使用内部 Meta类 来可以给模型赋予元数据。

模型的元数据即“所有不是字段的东西”，比如排序选项（ ordering ），数据库表名（ db_table ），或是阅读友好的单复数名（ verbose_name 和 verbose_name_plural ）。这些都不是必须的，并且在模型当中添加 Meta类 也完全是可选的。

## 管理器
它是 Django 模型和数据库查询操作之间的接口，并且它被用作从数据库当中 获取实例，如果没有指定自定义的 Manager 默认名称是 objects。Manager **只能通过模型类来访问，不能通过模型实例来访问**。

## 模型方法
在模型中添加自定义方法会给你的对象提供自定义的“行级”操作能力。  
与之对应的是类 Manager 的方法意在提供“表级”的操作，模型方法应该在某个对象实例上生效。

### 作用
* 将相关逻辑代码放在模型中
* 重写之前定义的模型方法

## 模型继承
模型继承在 Django 中与普通类继承在 Python 中的工作方式几乎完全相同，其基类应该继承自 django.db.models.Model。

Django 有三种可用的集成风格：抽象基类、多表继承、代理模式。

### 抽象基类
#### 作用
当需要将公共信息放入很多模型时，可以使用父类用于子类公共信息的载体，避免在每个子类中把这些代码都敲一遍。

#### 用法
在该模型的 Meta 类中填入 abstract=True。

#### 特性
不会生成数据表，也没有管理器，也不能被实例化和保存。  
当其用作其它模型类的基类时，它的字段会自动添加至子类。

从抽象基类继承来的字段可被其它字段或值重写，或用 None 删除。

#### Meta继承
当一个抽象基类被建立，Django 将所有你在基类中申明的 Meta 内部类以属性的形式提供。
```python
from django.db import models

class CommonInfo(models.Model):
    # ...
    class Meta:
        abstract = True
        ordering = ['name']

class Student(CommonInfo):
    # ...
    class Meta(CommonInfo.Meta):  # 以属性的形式，显式地继承父类的Meta
        db_table = 'student_info'
```
若子类未定义自己的 Meta 类，它会隐式地继承父类的 Meta。当然，子类也可显式地继承父类的 Meta。

##### 特性
* Django 在安装 Meta 属性前，对抽象基类的 Meta 做了一个调整——设置 abstract=False。这意味着**抽象基类的子类不会自动地变成抽象类**。为了继承一个抽象基类创建另一个抽象基类，你需要在子类上显式地设置 abstract=True。
* **抽象基类的某些 Meta 属性对子类是没用的**。比如，包含 db_table 意味着所有的子类（你并未在子类中指定它们的 Meta）会使用同一张数据表，这肯定不是你想要的。
* 由于Python继承的工作方式，如果子类从多个抽象基类继承，则**默认情况下仅继承第一个列出的类的 Meta 选项**。为了从多个抽象类中继承 Meta 选项，必须显式地声明 Meta 继承。

#### related_name 和 related_query_name
在抽象基类中使用related_name 或 related_query_name一般会引发问题，因为基类中的字段都被子类继承，且保持了同样的值。  
\# [TODO](https://docs.djangoproject.com/zh-hans/3.1/topics/db/models/#be-careful-with-related-name-and-related-query-name)

### 多表继承
Django 支持的第二种模型继承方式是层次结构中的每个模型都是一个单独的模型。每个模型都指向分离的数据表，且可被独立查询和创建。继承关系介绍了子类和父类之间的连接（通过一个自动创建的 OneToOneField ）。

#### 用法
```python
class ChildModel(ParentModel):
    pass
```

#### 特性
父类所有字段均在子类中可用，虽然数据分别存在不同的表中。  
若有一个对象，即使父类的实例，也是子类的实例，则可以通过小写的模型名将对象转换成父/子类实例。

子类中自动创建的连接至父类的 OneToOneField 看起来像这样:
```python
child_ptr = models.OneToOneField(
    ParentModel, on_delete=models.CASCADE,
    parent_link=True,
    primary_key=True,
)
```
可以在子类中重写该字段，通过申明 OneToOneField 并设置 parent_link=True。
##### 示例
自己创建 OneToOneField，并设置 parent_link=True，表明该属性用于连接回父类。像这样自行指定父类连接字段，可以修改连接回父类的属性名。

#### Meta继承
多表继承情况下，子类不会继承父类的 Meta。

子类模型无法访问父类的 Meta 类。不过，有限的几种情况下：若子类未指定 ordering 属性或 get_latest_by 属性，子类会从父类继承这些。  
如果父类有排序，而你并不期望子类有排序，你可以显式的禁止它:
```python
class ChildModel(ParentModel):
    # ...
    class Meta:
        # Remove parent's ordering effect
        ordering = []
```

#### 继承与反向关系
由于多表继承使用隐式的 OneToOneField 连接子类和父类，所以直接从父类访问子类是可能的。
如果在继承父类模型的子类中添加了 ForeignKey 或 ManyToManyField 关联，则必须指定 related_name 属性。

如果不指定related_name，则会有以下情况：
```python
# 代码
class Supplier(Place):
    customers = models.ManyToManyField(Place)

# 报错
Reverse query name for 'Supplier.customers' clashes with reverse query
name for 'Supplier.place_ptr'.

HINT: Add or change a related_name argument to the definition for
'Supplier.customers' or 'Supplier.place_ptr'.
```

> 如果不指定related_name，<u>隐式的 ***OneToOneField***</u> 和<u>显式的 ***ForeignKey 或 ManyToManyField***</u>指向同一个模型（上述例子中的Place模型），则Supplier和Place之间存在两个关联，两个关联都没有指定related_name，则以默认规则生成的两个的反向关联名称就会相同，因此产生冲突。

### 代理模式
#### 背景
使用 多表继承 时，每个子类模型都会创建一张新表。这一般是期望的行为，因为子类需要一个地方存储基类中不存在的额外数据字段。不过，有时候你只想修改模型的 Python 级行为——可能是修改默认管理器，或添加一个方法。

#### 作用
这是代理模型继承的目的：为原模型创建一个 代理。你可以创建，删除和更新代理模型的实例，所以的数据都会存储的像你使用原模型（未代理的）一样。不同点是你可以修改代理默认的模型排序和默认管理器，而不需要修改原模型。

#### 总结
通过为原模型创建一个代理，可以在修改原模型的前提下，更改原模型的行为。

#### 用法
```python
class MyPerson(Person):
    class Meta:
        proxy = True  # 通过将 Meta 类的 proxy 属性设置为 True。

    def do_something(self):  # 为 Person 模型添加一个方法
        # ...
        pass
```

#### 特性
MyPerson 类与父类 Person 操作同一张数据表，Person 的实例能通过 MyPerson 访问，反之亦然。

代理模型继承“Meta”属性和普通模型一样。

QuerySet 仍会返回请求的模型。  
当你用 Person 对象查询时，Django 永远不会返回 MyPerson 对象。

#### 基类约束
一个代理模型必须继承自一个非抽象模型类。你不能继承多个非抽象模型类，因为代理模型无法在不同数据表之间提供任何行间连接。一个代理模型可以继承任意数量的抽象模型类，假如他们没有定义任何的模型字段。一个代理模型也可以继承任意数量的代理模型，只需他们共享同一个非抽象父类。

#### 管理器
若你未在代理模型中指定模型管理器，它会从父类模型中继承。如果你在代理模型中指定了管理器，它会成为默认管理器，但父类中定义的管理器仍是可用的。

#### 代理继承和未托管的模型间的区别
\# TODO 不懂“未托管的模型”

## 多重继承
Django 模型能继承自多个父类模型。  
Python 的命名规则这里也有效。第一个出现的基类（比如 Meta ）就是会被使用的那个；举个例子，如果存在多个父类包含 Meta，只有第一个会被使用，其它的都会被忽略。

## 不允许隐藏字段名
在普通的 Python 类继承中，允许子类重写父类的任何属性。在 Django 中，针对模型字段在，这一般是不允许的。  
此规范不针对从抽象模型基类继承获得的字段。这些字段可被其它字段或值重写，也可以通过配置 field_name = None 删除。  
这些限制只针对那些是 Field 实例的属性，普通的 Python 属性可被随便重写。

若你在祖先模型中重写了任何模型字段，Django 会抛出一个 FieldError。

## 在一个包中管理模型
若有很多 models.py 文件，用独立的文件管理它们会很实用。

## 用法
创建一个 models 包。删除 models.py，创建一个 myapp/models 目录，包含一个 \_\_init\_\_.py 文件和存储模型的文件。在 \_\_init\_\_.py 文件中导入这些模块。
```python
from .organic import Person
from .synthetic import Robot
```