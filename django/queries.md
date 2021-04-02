# 执行查询
一旦创建 数据模型 后，Django 自动给予你一套数据库抽象 API，允许你创建，检索，更新和删除对象。

在本指南中（以及相关参考资料中），我们将引用以下模型，这些模型包含了一个 Webblog 应用：
```python
class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    number_of_comments = models.IntegerField()
    number_of_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```

## 创建对象
要创建一个对象，用关键字参数初始化它，然后调用 save() 将其存入数据库。

```python
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

这在幕后执行了 INSERT SQL 语句。Django 在你显式调用 save() 才操作数据库。  
save() 方法没有返回值。

要一步创建并保存一个对象，使用 create() 方法。

```python
>>> from blog.models import Blog
>>> Blog.objects.create(name='Beatles Blog', tagline='All the latest Beatles news.')
```

## 将修改保存至对象
要将修改保存至数据库中已有的某个对象，使用 save()。

有一个已被存入数据库中的 Blog 实例 b5，本例将其改名，并在数据库中更新其记录:
```python
>>> b5.name = 'New name'
>>> b5.save()
```

这在幕后执行了 UPDATE SQL 语句。Django 在你显示调用 save() 后才操作数据库。

### save详解
`Model.save(force_insert=False, force_update=False, using=DEFAULT_DB_ALIAS, update_fields=None)`
#### 重写
我们往往会重写save方法，添加自己的业务逻辑，然后在其中调用原来的save方法，保证Django基本工作机制正常。
```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def save(self, *args, **kwargs):
        do_something()   # 保存前做点私活
        super().save(*args, **kwargs)  # 一定不要忘记这行代码
        do_something_else()  # 保存后又加塞点东西
```

#### “共用”的save
模型实例的保存过程有一些值得探究的细节。

执行save方法后，Django才会真正为对象设置自增主键的值：
```python
>>> b2 = Blog(name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b2.id     # 返回None，因为此时b2还没有写入数据库，没有id值
>>> b2.save()
>>> b2.id     # 这回有了
```

当然，你也可以自己指定主键的值，不需要等待数据库为主键分配值，如下所示：
```python
>>> b3 = Blog(id=3, name='Cheddar Talk', tagline='Thoughts on cheese.')
>>> b3.id     # 返回 3.
>>> b3.save()
>>> b3.id     # 返回 3.
```

很显然，你必须确保分配的主键值是没有被使用过的，否则肯定出问题，因为在这种情况下，Django认为你是在更新一条已有的数据对象，而不是新建对象，比如下面的操作：
```python
b4 = Blog(id=3, name='Not Cheddar', tagline='Anything but cheese.')
b4.save() 
# 实际上是更新了上面的b3，而不是新建，此时b4==b3
```

上面现象的本质是**Django对SQL的INSERT和UPDATE语句进行了抽象合并，共用一个save方法**。

#### 强制保存
有些罕见情况下，可能你必须强制进行INSERT或者UPDATE操作，而不是让Django自动决定。这时候可以使用save方法的force_insert和force_update参数，将其中之一设置为True，强制指定保存模式。

#### 自增
有一种常见的需求是根据现有字段的值，更新成为新的值，比如点赞数+1的操作，通常我们可能写成如下的代码：
```python
>>> entry = Entry.objects.get(name='刘江的博客')
>>> entry.number_of_pingbacks += 1
>>> entry.save()
```

看起来没有什么问题，但实际上这里有个漏洞。首先会访问一次数据库，将number_of_pingbacks的值取出来，然后在Python的内存中进行加一操作，最后将新的值写回到数据库。两次读写倒还算好，关键是可能存在数据冲突，比如在同一时间有很多人点赞，肯定会出现错误。那如何解决这个问题呢？最简单的方法是使用Django的F表达式：

```python
>>> from django.db.models import F

>>> entry = Entry.objects.get(name='刘江的博客')
>>> entry.number_of_pingbacks = F('number_of_pingbacks') + 1
>>> entry.save()
```

为什么F表达式就可以避免上面的问题呢？因为Django设计的这个F表达式在获取关联字段值的时候不用先去数据库中取值然后在Python内存里计算，而是直接在数据库中取值和计算，直接更新数据库，不需要在Python中操作，自然就不存在数据竞争和冲突问题了。

#### 保存字段
save方法的最后一个参数是update_fields，它用于指定你要对模型的哪些字段进行更新，这对于性能可能有细微地提升，比如：
```python
product.name = 'Name changed again'
product.save(update_fields=['name'])
```
一些update_fields参数的说明：
* 接收任何的可迭代对象，每个元素都是字符串
* 参数值为空的迭代对象时，相当于跳过save方法
* 参数值为None时，默认更新所有字段
* 将强制为UPDATE方式

#### save执行流程
那么当你调用save()方法的时候，Django内部是怎么的执行顺序呢？

1. 触发pre-save信号，让任何监听此信号者执行动作。
2. 预处理数据。触发每个字段的pre-save()方法，用于实施自动地数据修改动作，比如时间字段处理auto_now_add或者auto_now参数。
3. 准备数据库数据。 要求每个字段提供的当前值是能够写入到数据库中的类型。类似整数、字符串等大多数类型不需要处理，只有一些复杂的类型需要做转换，比如时间。
4. 将数据插入到数据库内。
5. 触发post_save信号。

#### 注意
对于批量创建和批量更新操作，save()方法不会调用，甚至连pre_delete或者post_delete信号都不会触发，此时自定义的代码都是无效的。

### 保存 ForeignKey 和 ManyToManyField 字段
更新 ForeignKey 字段的方式与保存普通字段的方式相同——只需将正确类型的实例分配给相关字段。本例为 Entry 类的实例 entry 更新了 blog 属性，假设 Entry 和 Blog 的实例均已保存在数据库中（因此能在下面检索它们）：
```python
>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name="Cheddar Talk")
>>> entry.blog = cheese_blog
>>> entry.save()
```

更新 ManyToManyField 字段有点不同——在字段上使用 add() 方法为关联关系添加一条记录。本例将 Author 实例 joe 添加至 entry 对象:
```python
>>> from blog.models import Author
>>> joe = Author.objects.create(name="Joe")
>>> entry.authors.add(joe)
```

要一次添加多行记录至 ManyToManyField 字段，在一次调用 add() 时传入多个参数，像这样:
```python
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

## 检索对象
要从数据库检索对象，要通过模型类的 Manager 构建一个 QuerySet。

一个 QuerySet 代表来自数据库中对象的一个集合。它可以有 0 个，1 个或者多个 filters. Filters，可以根据给定参数缩小查询结果量。  
在 SQL 的层面上， QuerySet 对应 SELECT 语句，而*filters*对应类似 WHERE 或 LIMIT 的限制子句。

你能通过模型的 Manager 获取 QuerySet。每个模型至少有一个 Manager，默认名称是 objects。像这样直接通过模型类使用它:
```python
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name='Foo', tagline='Bar')
>>> b.objects
Traceback:
    ...
AttributeError: "Manager isn't accessible via Blog instances."
```

> Managers 只能通过模型类访问，而不是通过模型实例，目的是强制分离 “表级” 操作和 “行级” 操作。

### 1. 检索全部对象
从数据库中检索对象最简单的方式就是检索全部。为此，在 Manager 上调用 all() 方法:
```python
>>> all_entries = Entry.objects.all()
```
方法 all() 返回了一个包含数据库中所有对象的 QuerySet 对象。

### 2. 通过过滤器检索指定对象
all() 返回的 QuerySet 包含了数据表中所有的对象。虽然，大多数情况下，你只需要完整对象集合的一个子集。

要创建一个这样的子集，你需要通过添加过滤条件精炼原始 QuerySet。两种最常见的精炼 QuerySet 的方式是：
* filter(\*\*kwargs): 返回一个新的 QuerySet，包含的对象**满足**给定查询参数。
* exclude(\*\*kwargs): 返回一个新的 QuerySet，包含的对象**不满足**给定查询参数。

> 查询参数（\*\*kwargs）应该符合 *Field lookups* 的要求。

例如，要包含获取 2006 年的博客条目（entries blog）的 QuerySet，像这样使用 filter():
`Entry.objects.filter(pub_date__year=2006)`

通过默认管理器类也一样:
`Entry.objects.all().filter(pub_date__year=2006)`

#### 链式过滤器
精炼 QuerySet 的结果本身还是一个 QuerySet，所以能串联精炼过程。例子:
```python
>>> Entry.objects.filter(
...     headline__startswith='What'
... ).exclude(
...     pub_date__gte=datetime.date.today()
... ).filter(
...     pub_date__gte=datetime.date(2005, 1, 30)
... )
```
这个先获取包含数据库所有条目（entry）的 QuerySet，然后排除一些，再进入另一个过滤器。最终的 QuerySet 包含标题以 "What" 开头的，发布日期介于 2005 年 1 月 30 日与今天之间的所有条目。

#### 每个 QuerySet 都是唯一的
每次精炼一个 QuerySet，你就会获得一个全新的 QuerySet，后者与前者毫无关联。每次精炼都会创建一个单独的、不同的 QuerySet，能被存储，使用和复用。

举例：
```python
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```
这三个 QuerySets 是独立的。第一个是基础 QuerySet，包含了所有标题以 "What" 开头的条目。第二个是第一个的子集，带有额外条件，排除了 pub_date 是今天和今天之后的所有记录。第三个是第一个的子集，带有额外条件，只筛选 pub_date 是今天或未来的所有记录。最初的 QuerySet (q1) 不受筛选操作影响。

#### QuerySet 是惰性的
QuerySet 是惰性的 —— 创建 QuerySet 并不会引发任何数据库活动。你可以将一整天的过滤器都堆积在一起，Django 只会在 QuerySet 被 **计算** 时执行查询操作。来瞄一眼这个例子:
```python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```
虽然这看起来像是三次数据库操作，实际上只在最后一行 (print(q)) 做了一次。一般来说， QuerySet 的结果直到你 “要使用” 时才会从数据库中拿出。当你要用时，才通过数据库 计算 出 QuerySet。

#### 用 get() 检索单个对象
filter() 总是返回一个 QuerySet，即便只有一个对象满足查询条件 —— 这种情况下， QuerySet 只包含了一个元素。

若你知道只会有一个对象满足查询条件，你可以在 Manager 上使用 get() 方法，它会直接返回这个对象:
```python
>>> one_entry = Entry.objects.get(pk=1)
```
你可以对 get() 使用与 filter() 类似的所有查询表达式 —— 同样的，参考 Field lookups。

注意， 使用切片 [0] 时的 get() 和 filter() 有点不同。如果没有满足查询条件的结果， get() 会抛出一个 DoesNotExist 异常。该异常是执行查询的模型类的一个属性 —— 所有，上述代码中，若没有哪个 Entry 对象的主键是 1，Django 会抛出 Entry.DoesNotExist。

类似了，Django 会在有不止一个记录满足 get() 查询条件时发出警告。这时，Django 会抛出 MultipleObjectsReturned，这同样也是模型类的一个属性。

#### 其它 QuerySet 方法
大多数情况下，你会在需要从数据库中检索对象时使用 all()， get()， filter() 和 exclude()。然而，这样远远不够；完整的各种 QuerySet 方法请参阅 [QuerySet API 参考](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/)。

#### 限制 QuerySet 条目数
利用 Python 的数组切片语法将 QuerySet 切成指定长度。这等价于 SQL 的 LIMIT 和 OFFSET 子句。  
参考下面的例子：
```python
>>> Entry.objects.all()[:5]      # 返回前5个对象
>>> Entry.objects.all()[5:10]    # 返回第6个到第10个对象
```

不支持负索引 (例如 Entry.objects.all()[-1])

一般情况下， QuerySet 的切片返回一个新的 QuerySet —— 其并未执行查询。一个特殊情况是使用了的 Python 切片语法的 “步长”。例如，这将会实际的执行查询命令，为了获取从前 10 个对象中，每隔一个抽取的对象组成的列表:
`>>> Entry.objects.all()[:10:2]`

由于对 queryset 切片工作方式的模糊性，禁止对其进行进一步的排序或过滤。

要检索 单个 对象而不是一个列表时（例如 SELECT foo FROM bar LIMIT 1），请使用索引，而不是切片。例如，这会返回按标题字母排序后的第一个 Entry:
`>>> Entry.objects.order_by('headline')[0]`

这大致等价于:
`>>> Entry.objects.order_by('headline')[0:1].get()`

然而，注意一下，若没有对象满足给定条件，前者会抛出 IndexError，而后者会抛出 DoesNotExist。参考 get() 获取更多细节。

#### 字段查询
字段查询即你如何制定 SQL WHERE 子句。它们以关键字参数的形式传递给 QuerySet 方法 filter()， exclude() 和 get()。

基本的查询关键字参数遵照 field__lookuptype=value。（有个双下划线）。例如:
```python
>>> Entry.objects.filter(pub_date__lte='2006-01-01')
```

转换为 SQL 语句大致如下：
```python
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

**默认查找类型为exact**。

查询子句中指定的字段必须是模型的一个字段名。不过也有个例外，在 ForeignKey 中，你可以指定以 _id 为后缀的字段名。这种情况下，value 参数需要包含 foreign 模型的主键的原始值。例子：
```python
>>> Entry.objects.filter(blog_id=4)
```

如果你传递了一个非法的键值，查询函数会抛出TypeError异常。

数据库 API 支持两套查询类型；完整参考文档位于[字段查询参考](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/#field-lookups)。

#### 跨关系查询
Django 提供了一种强大而直观的方式来解决跨越关联的查询，它在后台自动执行包含JOIN的SQL语句。要跨越某个关联，只需使用关联的模型字段名称，并使用双下划线分隔，直至你想要的字段（可以链式跨越，无限跨度）。

本例检索出所有的 Entry 对象，其 Blog 的 name 为 'Beatles Blog' ：
```python
>>> Entry.objects.filter(blog__name='Beatles Blog')
```

跨域的深度随你所想。

反向操作也能行。尽管可以自定义，但默认情况下，你使用模型的小写名称在查找中引用“反向”关系。

本例检索的所有 Blog 对象均拥有少一个 标题 含有 'Lennon' 的条目:
```python
>>> Blog.objects.filter(entry__headline__contains='Lennon')
```

如果你在跨多个关系进行筛选，而某个中间模型的没有满足筛选条件的值，Django 会将它当做一个空的（所有值都是 NULL）但是有效的对象。这样就意味着不会抛出错误。例如，在这个过滤器中:
```python
Blog.objects.filter(entry__authors__name='Lennon')
```
假设有个关联的 Author 模型），若某项条目没有任何关联的 author，它会被视作没有关联的 name，而不是因为缺失 author 而抛出错误。通常，这是比较符合逻辑的处理方式。唯一可能让你困惑的是当你使用isnull的时候：
```python
Blog.objects.filter(entry__authors__name__isnull=True)
```
将会返回 Blog 对象，包含 author 的 name 为空的对象，以及那些 entry 的 author 为空的对象。若你不想要后面的对象，你可以这样写:
```python
Blog.objects.filter(entry__authors__isnull=False, entry__authors__name__isnull=True)
```

##### 跨越多值的关系查询
最基本的filter和exclude的关键字参数只有一个，这种情况很好理解。但是当关键字参数有多个，且是跨越外键或者多对多的情况下，那么就比较复杂，让人迷惑了。我们看下面的例子：
```python
Blog.objects.filter(entry__headline__contains='Lennon', entry__pub_date__year=2008)  # 式1
```
这是一个跨外键、两个过滤参数的查询。此时我们理解两个参数之间属于-与“and”的关系，也就是说，过滤出来的BLog对象对应的entry对象必须同时满足上面两个条件。这点很好理解。也就是说**上面要求entry同时满足两个条件**。

但是，当采用链式过滤就不一样了。要筛选所有条目标题包含 "Lennon" 和条目发布于 2008 年的博客（要求目标blog下的entry分别满足下面两个条件，不同时满足也行，当然同时满足也行），我们会这样写:
```python
Blog.objects.filter(entry__headline__contains='Lennon').filter(entry__pub_date__year=2008)  # 式2
```

更头疼的来了，看看exclude的情况！
```python
Blog.objects.exclude(entry__headline__contains='Lennon', entry__pub_date__year=2008)  # 式3
```

这会排除一些blog，这些blog的关联文章中同时有下面两种类型：
* headline中包含“Lennon”
* 在2008年发布的
而不管关联文章中是否有同时具备以上两个条件的文章。

那么要排除关联有同时满足上面两个条件的文章的blog，该怎么办呢？看下面：
```
Blog.objects.exclude(
entry=Entry.objects.filter(
    headline__contains='Lennon',
    pub_date__year=2008,
),
)
```
（有没有很坑爹的感觉？所以，建议在碰到跨关系的多值查询时，尽量使用Q查询）

#### F表达式
在之前的例子中，我们已经构建过的 filter 都是将模型字段值与常量做比较。但是，要怎么做才能将模型字段值与同一模型中的另一字段做比较呢？

Django 提供了 F 表达式 实现这种比较。 F() 的实例充当查询中的模型字段的引用。这些引用可在查询过滤器中用于在同一模型实例中比较两个不同的字段。 

例如，为了查找comments数目多于pingbacks数目的Entry，可以构造一个F()对象来引用pingback数目，并在查询中使用该F()对象：
```python
>>> from django.db.models import F
>>> Entry.objects.filter(number_of_comments__gt=F('number_of_pingbacks'))
```
Django 支持对 F() 对象进行加、减、乘、除、求余和次方，另一操作数既可以是常量，也可以是其它 F() 对象。  
例如查找comments数目比pingbacks两倍还要多的Entry，我们可以这么写：
```python
>>> Entry.objects.filter(number_of_comments__gt=F('number_of_pingbacks') * 2)
```
为了查询rating比pingback和comment数目总和要小的Entry，我们可以这么写：
```python
>>> Entry.objects.filter(rating__lt=F('number_of_comments') + F('number_of_pingbacks'))
```
你也能用双下划线在 F() 对象中通过关联关系查询。带有双下划线的 F() 对象将引入访问关联对象所需的任何连接。  
例如，查询author的名字与blog名字相同的Entry：
```python
>>> Entry.objects.filter(authors__name=F('blog__name'))
```
对于date和date/time字段，还可以加或减去一个timedelta对象。下面的例子将返回发布时间超过3天后被修改的所有Entry：
```python
>>> from datetime import timedelta
>>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
```
F()对象还支持.bitand()、.bitor()、.bitxor()、.bitrightshift()和.bitleftshift()等位操作，例如：
```python
>>> F('somefield').bitand(16)
```

#### 主键 (pk) 查询快捷方式
出于方便的目的，Django 提供了一种 pk 查询快捷方式， pk 表示主键 "primary key"。

示例 Blog 模型中，主键是 id 字段，所以这 3 个语句是等效的:
```python
>>> Blog.objects.get(id__exact=14) # 显式形式
>>> Blog.objects.get(id=14) # __exact 是隐式的
>>> Blog.objects.get(pk=14) # pk 隐式地表示 id__exact
```

pk 的使用并不仅限于 __exact 查询——任何的查询项都能接在 pk 后面，执行对模型主键的查询:
```python
# Get blogs entries with id 1, 4 and 7
>>> Blog.objects.filter(pk__in=[1,4,7])

# Get all blog entries with id > 14
>>> Blog.objects.filter(pk__gt=14)
```

pk 查找也支持跨连接。例如，以下 3 个语句是等效的:
```python
>>> Entry.objects.filter(blog__id__exact=3) # 显式形式
>>> Entry.objects.filter(blog__id=3)        # __exact 是隐式的
>>> Entry.objects.filter(blog__pk=3)        # __pk 隐式地表示 __id__exact
```

#### 百分号和下划线
在原生SQL语句中%符号有特殊的作用。Django帮你自动转义了百分符号和下划线，你可以和普通字符一样使用它们，如下所示：
```python
>>> Entry.objects.filter(headline__contains='%')
# 它和下面的一样
# SELECT ... WHERE headline LIKE '%\%%';
```

#### 缓存和 QuerySet
每个QuerySet都包含一个缓存，用于减少对数据库的实际操作。理解这个概念，有助于你提高查询效率。

对于新创建的QuerySet，它的缓存是空的。当QuerySet第一次被提交后，数据库执行实际的查询操作，Django会把查询的结果保存在QuerySet的缓存内，随后的对于该QuerySet的提交将重用这个缓存的数据。

要想高效的利用查询结果，降低数据库负载，你必须善于利用缓存。看下面的例子，这会造成2次实际的数据库操作，加倍数据库的有一些操作不会缓存QuerySet，例如切片和索引。这就导致这些操作没有缓存可用，每次都会执行实际的数据库查询操作。例如：负载，同时由于时间差的问题，可能在两次操作之间数据被删除或修改或添加，导致脏数据的问题：
```python
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
```

要避免此问题，保存 QuerySet 并复用它:
```python
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # 提交查询
>>> print([p.pub_date for p in queryset]) # 重用查询缓存
```

##### 何时不会被缓存
有一些操作不会缓存QuerySet，例如切片和索引。这就导致这些操作没有缓存可用，每次都会执行实际的数据库查询操作。例如：
```python
>>> queryset = Entry.objects.all()
>>> print(queryset[5]) # 查询数据库
>>> print(queryset[5]) # 再次查询数据库
```

但是，如果已经遍历过整个QuerySet，那么就相当于缓存过，后续的操作则会使用缓存，例如：
```python
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # 查询数据库
>>> print(queryset[5]) # 使用缓存
>>> print(queryset[5]) # 使用缓存
```

下面的这些操作都将遍历QuerySet并建立缓存：
```python
>>> [entry for entry in queryset]
>>> bool(queryset)
>>> entry in queryset
>>> list(queryset)
```

注意：简单的打印QuerySet并不会建立缓存，因为__repr__()调用只返回全部查询集的一个切片。

#### 查询JsonField
JSONField 里的查找实现和其它字段类型不太一样，主要因为存在key转换。为了演示，我们将使用下面这个例子：

```python
from django.db import models

class Dog(models.Model):
    name = models.CharField(max_length=200)
    data = models.JSONField(null=True)

    def __str__(self):
        return self.name
```

##### 保存和查询 None 值
因为JSON、Python和SQL三者在空值上的定义不太一样，所以需要谨慎处理。(null、None、NULL)

与其他字段一样，当 None 作为字段值来保存时，将像 SQL 的 NULL 值一样保存。虽然不建议这样做，但可以使用 Value('null') 来存储 JSON 的 null 值。

无论存储哪个值，当从数据库中检索时，Python表示JSON的空值和 SQL 里的 NULL 一样，都是 None 。因此，很难区分它们。

这只适用于 None 值作为字段的顶级值。如果 None 被保存在列表或字典中，它将始终被解释为JSON的 null 值。

当查询时，None 值将一直被解释为JSON的null。要查询SQL的NULL，请使用 isnull参数。

请仔细揣摩下面的例子：
```python
>>> Dog.objects.create(name='Max', data=None)  # SQL NULL.
<Dog: Max>
>>> Dog.objects.create(name='Archie', data=Value('null'))  # JSON null.
<Dog: Archie>
>>> Dog.objects.filter(data=None)
<QuerySet [<Dog: Archie>]>
>>> Dog.objects.filter(data=Value('null'))
<QuerySet [<Dog: Archie>]>
>>> Dog.objects.filter(data__isnull=True)
<QuerySet [<Dog: Max>]>
>>> Dog.objects.filter(data__isnull=False)
<QuerySet [<Dog: Archie>]>

# python的None保存到数据库会变成SQL的NULL
# python的None在查询的时候会查询到JSON的null
# Django需要查询出SQL的NULL要使用isnull（lookuptype）
```

除非你确定要使用SQL 的 NULL 值，否则请考虑设置 null=False 并为空值提供合适的默认值，例如 default=dict 。

注意：保存JSON的 null 值不违反Django的 null=False 。

##### Key, index, 和路径转换
为了查询给定的字典键，请将该键作为查询名：
```python
>>> Dog.objects.create(name='Rufus', data={
...     'breed': 'labrador',
...     'owner': {
...         'name': 'Bob',
...         'other_pets': [{
...             'name': 'Fishy',
...         }],
...     },
... })
<Dog: Rufus>
>>> Dog.objects.create(name='Meg', data={'breed': 'collie', 'owner': None})
<Dog: Meg>
>>> Dog.objects.filter(data__breed='collie')
<QuerySet [<Dog: Meg>]>
```

可以将多个键用双下划线链接起来形成一个路径查询：
```python
>>> Dog.objects.filter(data__owner__name='Bob')
<QuerySet [<Dog: Rufus>]>
```

如果键是个整数，那么它将在列表中被解释成一个索引：
```python
>>> Dog.objects.filter(data__owner__other_pets__0__name='Fishy')
<QuerySet [<Dog: Rufus>]>
```

如果要查询的键与另一个查询的键名冲突，请改用 contains 来查询。

如果查询时缺少键名，请使用 isnull 查询：
```python
>>> Dog.objects.create(name='Shep', data={'breed': 'collie'})
<Dog: Shep>
>>> Dog.objects.filter(data__owner__isnull=True)
<QuerySet [<Dog: Shep>]>
```

上面给出的例子隐式地使用了exact 查找。Key、索引和路径转换也可以用：icontains, endswith, iendswith, iexact, regex, iregex, startswith, istartswith, lt, lte, gt, gte等查询手段。

##### 包含与键查找
JSONField 上的 contains 查找逻辑被重写了。返回的对象是那些给定的键值对都包含在顶级字段中的对象。例如：
```python
>>> Dog.objects.create(name='Rufus', data={'breed': 'labrador', 'owner': 'Bob'})
<Dog: Rufus>
>>> Dog.objects.create(name='Meg', data={'breed': 'collie', 'owner': 'Bob'})
<Dog: Meg>
>>> Dog.objects.create(name='Fred', data={})
<Dog: Fred>
>>> Dog.objects.filter(data__contains={'owner': 'Bob'})
<QuerySet [<Dog: Rufus>, <Dog: Meg>]>
>>> Dog.objects.filter(data__contains={'breed': 'collie'})
<QuerySet [<Dog: Meg>]>
```

##### contained_by
将参数值定义为集合A，每个实例的值定义为集合B，如果B是A的子集，那么这个实例将被选中：
```python
>>> Dog.objects.create(name='Rufus', data={'breed': 'labrador', 'owner': 'Bob'})
<Dog: Rufus>
>>> Dog.objects.create(name='Meg', data={'breed': 'collie', 'owner': 'Bob'})
<Dog: Meg>
>>> Dog.objects.create(name='Fred', data={})
<Dog: Fred>
>>> Dog.objects.filter(data__contained_by={'breed': 'collie', 'owner': 'Bob'})
<QuerySet [<Dog: Meg>, <Dog: Fred>]>
>>> Dog.objects.filter(data__contained_by={'breed': 'collie'})
<QuerySet [<Dog: Fred>]>
```

##### has_key
通过键过滤对象，匹配的键必须位于嵌套的最顶层：
```python
>>> Dog.objects.create(name='Rufus', data={'breed': 'labrador'})
<Dog: Rufus>
>>> Dog.objects.create(name='Meg', data={'breed': 'collie', 'owner': 'Bob'})
<Dog: Meg>
>>> Dog.objects.filter(data__has_key='owner')
<QuerySet [<Dog: Meg>]>
```

##### has_keys
返回同时具备指定的多个键的对象，这些键必须位于最顶层。:
```python
>>> Dog.objects.create(name='Rufus', data={'breed': 'labrador'})
<Dog: Rufus>
>>> Dog.objects.create(name='Meg', data={'breed': 'collie', 'owner': 'Bob'})
<Dog: Meg>
>>> Dog.objects.filter(data__has_keys=['breed', 'owner'])
<QuerySet [<Dog: Meg>]>
```

##### has_any_keys
只要具备指定键列表中的任何一个键，这个对象就会被返回。这些键必须位于最顶层。
```python
>>> Dog.objects.create(name='Rufus', data={'breed': 'labrador'})
<Dog: Rufus>
>>> Dog.objects.create(name='Meg', data={'owner': 'Bob'})
<Dog: Meg>
>>> Dog.objects.filter(data__has_any_keys=['owner', 'breed'])
<QuerySet [<Dog: Rufus>, <Dog: Meg>]>
```


#### Q对象
在类似 filter() 中，查询使用的关键字参数是通过 "AND" 连接起来的。如果你要执行更复杂的查询（例如，由 OR 语句连接的查询），你可以使用 Q 对象。

Q来自django.db.models.Q，用于封装关键字参数的集合，可以作为关键字参数用于filter、exclude和get等函数。 例如：
```python
from django.db.models import Q
Q(question__startswith='What')
```

可以使用&或者|或~来组合Q对象，分别表示与、或、非逻辑。它将返回一个新的Q对象。
```python
Q(question__startswith='Who')|Q(question__startswith='What')
# 这相当于：
WHERE question LIKE 'Who%' OR question LIKE 'What%'

Q(question__startswith='Who') | ~Q(pub_date__year=2005)
```

每个接受关键字参数的查询函数 (例如 filter()， exclude()， get()) 也同时接受一个或多个 Q 对象作为位置（未命名的）参数。若你为查询函数提供了多个 Q 对象参数，这些参数会通过 "AND" 连接。例子:
```python
Poll.objects.get(
Q(question__startswith='Who'),
Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
# 它相当于
# SELECT * from polls WHERE question LIKE 'Who%' AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
```

查询函数能混合使用 Q 对象和关键字参数。所有提供给查询函数的参数（即关键字参数或 Q 对象）均通过 "AND" 连接。然而，若提供了 Q 对象，那么它必须位于所有关键字参数之前。例子:
```python
Poll.objects.get(
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
    question__startswith='Who',
)
```

## 比较对象
要比较两个模型实例，使用标准的 Python 比较操作符，两个等号： ==。实际上，这比较了两个模型实例的主键值。

使用前文的 Entry，以下的两个语句是等效的:
```python
>>> some_entry == other_entry
>>> some_entry.id == other_entry.id
```

如果模型的主键不叫做“id”也没关系，后台总是会使用正确的主键名字进行比较，例如，如果一个模型的主键的名字是“name”，那么下面是相等的：
```python
>>> some_obj == other_obj
>>> some_obj.name == other_obj.name
```

## 删除对象
删除对象使用的是对象的delete()方法。该方法将返回被删除对象的总数量和一个字典，字典包含了每种被删除对象的类型和该类型的数量。如下所示：
```python
>>> e.delete()
(1, {'weblog.Entry': 1})
```

也可以批量删除。每个QuerySet都有一个delete()方法，它能删除该QuerySet的所有成员。例如：
```python
>>> Entry.objects.filter(pub_date__year=2020).delete()
(5, {'webapp.Entry': 5})
```

需要注意的是，有可能不是每一个对象的delete方法都被执行。如果你改写了delete方法，为了确保对象被删除，你必须手动迭代QuerySet进行逐一删除操作。

当Django删除一个对象时，它默认使用SQL的ON DELETE CASCADE约束，也就是说，任何有外键指向要删除对象的对象将一起被删除。例如：
```python
b = Blog.objects.get(pk=1)
# 下面的动作将删除该条Blog和所有的它关联的Entry对象
b.delete()
```
这种级联的行为可以通过的ForeignKey的on_delete参数自定义。

注意，delete()是唯一没有在管理器上暴露出来的方法。这是刻意设计的一个安全机制，用来防止你意外地请求类似Entry.objects.delete()的动作，而不慎删除了所有的条目。如果你确实想删除所有的对象，你必须明确地请求一个完全的查询集，像下面这样：
```python
Entry.objects.all().delete()
```

在多表继承时，如果你只想删除子类的对象，而不想删除父类的数据，可以使用参数keep_parents=True。

注意：有些删除模型实例的方法不会触发delete()方法，比如批量删除和级联删除，为了确保你的自定义代码被执行，请使用pre_delete或者post_delete信号机制来替代delete方法。

## 复制模型实例
虽然没有内置的方法用于复制模型的实例，但还是很容易创建一个新的实例并将原实例的所有字段都拷贝过来。最简单的方法是将原实例的pk设置为None，这会创建一个新的实例copy。示例如下：
```python
blog = Blog(name='My blog', tagline='Blogging is easy')
blog.save() # blog.pk == 1
#
blog.pk = None
blog.save() # blog.pk == 2
```

但是在继承父类的时候，情况会变得复杂，如果有下面一个Blog的子类：
```python
class ThemeBlog(Blog):
    theme = models.CharField(max_length=200)

django_blog = ThemeBlog(name='Django', tagline='Django is easy', theme='python')
django_blog.save() # django_blog.pk == 3
```
基于继承的工作机制，你必须同时将pk和id设为None：
```python
django_blog.pk = None
django_blog.id = None
django_blog.save() # django_blog.pk == 4
```

对于外键和多对多关系，更需要进一步处理。例如，Entry有一个ManyToManyField到Author。 复制条目后，您必须为新条目设置多对多关系，像下面这样：
```pythonentry = Entry.objects.all()[0] # some previous entry
old_authors = entry.authors.all()
entry.pk = None
entry.save()
entry.authors.set(old_authors)
```

对于OneToOneField，还要复制相关对象并将其分配给新对象的字段，以避免违反一对一唯一约束。例如，指定前文复制的 entry:
```python
detail = EntryDetail.objects.all()[0]
detail.pk = None
detail.entry = entry
detail.save()
```

## 批量更新对象
使用update()方法可以批量为QuerySet中所有的对象进行更新操作。
```python
# 更新所有2007年发布的entry的headline
Entry.objects.filter(pub_date__year=2020).update(headline='刘江的Django教程')
```

只可以对普通字段和ForeignKey字段使用这个方法。若要更新一个普通字段，只需提供一个新的常数值。若要更新ForeignKey字段，需设置新值为你想指向的新模型实例。例如：
```python
>>> b = Blog.objects.get(pk=1)
# 修改所有的Entry，让他们都属于b
>>> Entry.objects.all().update(blog=b)
```

update方法会被立刻执行，并返回操作匹配到的行的数目（有可能不等于要更新的行的数量，因为有些行可能已经有这个新值了）。

更新 QuerySet 的唯一限制即它只能操作一个数据表：该模型的主表。你可以基于关联字段进行筛选，但你只能更新模型主表中的列。例子:
```python
>>> b = Blog.objects.get(pk=1)
# Update all the headlines belonging to this Blog.
>>> Entry.objects.select_related().filter(blog=b).update(headline='Everything is the same')
```

要注意的是update()方法会直接转换成一个SQL语句，并立刻批量执行。  
update方法不会运行模型的save()方法，或者产生pre_save或post_save信号（调用save()方法产生）或者服从auto_now字段选项。如果你想保存QuerySet中的每个条目并确保每个实例的save()方法都被调用，你不需要使用任何特殊的函数来处理。只需要迭代它们并调用save()方法：
```python
for item in my_queryset:
    item.save()
```

update方法可以配合F表达式。这对于批量更新同一模型中某个字段特别有用。例如增加Blog中每个Entry的pingback个数：
```python
>>> Entry.objects.all().update(number_of_pingbacks=F('number_of_pingbacks') + 1)
```

然而，与filter和exclude子句中的F()对象不同，在update中你不可以使用F()对象进行跨表操作，你只可以引用正在更新的模型的字段。如果你尝试使用F()对象引入另外一张表的字段，将抛出FieldError异常：
```python
# 这将会导致一个FieldError异常
>>> Entry.objects.update(headline=F('blog__name'))
```

## 关联对象的查询
当你在模型中定义了关联关系（如 ForeignKey， OneToOneField， 或 ManyToManyField），该模型的实例将会自动获取一套 API，能快捷地访问关联对象。  
利用本节一开始的模型，一个Entry对象e可以通过blog属性e.blog获取关联的Blog对象。反过来，Blog对象b可以通过entry_set属性b.entry_set.all()访问与它关联的所有Entry对象。

### 一对多关联

#### 正向查询
直接通过圆点加属性，访问外键对象：
```python
>>> e = Entry.objects.get(id=2)
>>> e.blog # 返回关联的Blog对象
```

要注意的是，对外键的修改，必须调用save方法进行保存，例如：
```python
>>> e = Entry.objects.get(id=2)
>>> e.blog = some_blog
>>> e.save()
```

如果一个外键字段设置有null=True属性，那么可以通过给该字段赋值为None的方法移除外键值：
```python
>>> e = Entry.objects.get(id=2)
>>> e.blog = None
>>> e.save() # "UPDATE blog_entry SET blog_id = NULL ...;"
```

在第一次对一个外键关系进行正向访问的时候，关系对象会被缓存。随后对同样外键关系对象的访问会使用这个缓存，例如：
```python
>>> e = Entry.objects.get(id=2)
>>> print(e.blog)  # 访问数据库，获取实际数据
>>> print(e.blog)  # 不会访问数据库，直接使用缓存的版本
```

请注意QuerySet的select_related()方法会递归地预填充所有的一对多关系到缓存中。例如：
```pyhon
>>> e = Entry.objects.select_related().get(id=2)
>>> print(e.blog)  # 不会访问数据库，直接使用缓存
>>> print(e.blog)  # 不会访问数据库，直接使用缓存
```

#### 反向查询
如果一个模型有ForeignKey，那么该ForeignKey所指向的外键模型的实例可以通过一个管理器进行反向查询，返回源模型的所有实例。默认情况下，这个管理器的名字为FOO_set，其中FOO是源模型的小写名称。该管理器返回的查询集可以用前面提到的方式进行过滤和操作。
```python
>>> b = Blog.objects.get(id=1)
>>> b.entry_set.all() # 返回一些Entry实例，这些实例都关联到博客b
# b.entry_set 本质上是一个管理器
>>> b.entry_set.filter(headline__contains='Lennon')
>>> b.entry_set.count()
```

你可以在ForeignKey字段的定义中，通过设置related_name来重写FOO_set的名字。举例说明，如果你修改Entry模型`blog = ForeignKey(Blog, on_delete=models.CASCADE, related_name=’entries’)`，那么上面的例子会变成下面的样子：
```python
>>> b = Blog.objects.get(id=1)
>>> b.entries.all() 

>>> b.entries.filter(headline__contains='Lennon')
>>> b.entries.count()
```

##### 使用自定义的反向管理器
默认情况下，用于反向关联的RelatedManager是该模型默认管理器的子类。如果你想为一个查询指定一个不同的管理器，你可以使用下面的语法：
```python
from django.db import models

class Entry(models.Model):
    #...
    objects = models.Manager()  # 默认管理器
    entries = EntryManager()    # 自定义管理器

b = Blog.objects.get(id=1)
b.entry_set(manager='entries').all()
```

当然，指定的自定义反向管理器也可以调用它的自定义方法：
```python
b.entry_set(manager='entries').is_published()
```

### 多对多关联
多对多关系的两端都会自动获得访问另一端的API。这些API的工作方式与前面提到的“反向”一对多关系的用法一样。
```python
e = Entry.objects.get(id=3)

e.authors.all() # Returns all Author objects for this Entry.
e.authors.count()
e.authors.filter(name__contains='John')
#
a = Author.objects.get(id=5)
a.entry_set.all() # Returns all Entry objects for this Author.
```
与外键字段中一样，在多对多的字段中也可以指定related_name名。

### 一对一关联
一对一非常类似多对一关系，可以简单的通过模型的属性访问关联的模型。
```python
class EntryDetail(models.Model):
    entry = models.OneToOneField(Entry, on_delete=models.CASCADE)
    details = models.TextField()

ed = EntryDetail.objects.get(id=2)
ed.entry # Returns the related Entry object.
```

不同之处在于反向查询的时候。一对一关系中的关联模型同样具有一个管理器对象，但是该管理器表示一个单一的对象而不是对象的集合：
```python
e = Entry.objects.get(id=2)
e.entrydetail # 返回关联的EntryDetail对象
```

如果没有对象赋值给这个关系，Django将抛出一个DoesNotExist异常。 可以给反向关联进行赋值，方法和正向的关联一样：
```python
e.entrydetail = ed
```

### 反向关联是如何实现的？
一些ORM框架需要你在关系的两端都进行定义。Django的开发者认为这违反了DRY (Don’t Repeat Yourself)原则，所以在Django中你只需要在一端进行定义。

那么这是怎么实现的呢？因为在关联的模型类没有被加载之前，一个模型类根本不知道有哪些类和它关联。

答案在app registry！在Django启动的时候，它会导入所有INSTALLED_APPS中的应用和每个应用中的模型模块。每创建一个新的模型时，Django会自动添加反向的关系到所有关联的模型。如果关联的模型还没有导入，Django将保存关联的记录并在关联的模型导入时添加这些关系。

由于这个原因，将模型所在的应用都定义在INSTALLED_APPS的应用列表中就显得特别重要。否则，反向关联将不能正确工作。

### 通过关联对象进行查询
涉及关联对象的查询与正常值的字段查询遵循同样的规则。当你指定查询需要匹配的值时，你可以使用一个对象实例或者对象的主键值。

例如，如果你有一个id=5的Blog对象b，下面的三个查询将是完全一样的：
```python
Entry.objects.filter(blog=b) # 使用对象实例
Entry.objects.filter(blog=b.id) # 使用实例的id
Entry.objects.filter(blog=5) # 直接使用id
```

### 关联对象的增删改
除了在前面定义的QuerySet方法之外，管理器还有其它方法用于处理关联的对象集合。下面是每个方法的概括:
* add(obj1, obj2, ...)：添加指定的模型对象到关联的对象集中。
* create(**kwargs)：创建一个新的对象，将它保存并放在关联的对象集中。返回新创建的对象。
* remove(obj1, obj2, ...)：从关联的对象集中删除指定的模型对象。
* clear()：清空关联的对象集。
* set([obj1,obj2...])：重置关联的对象集。

若要一次性给关联的对象集赋值，使用set()方法，并给它赋值一个可迭代的对象集合或者一个主键值的列表。例如：
```python
b = Blog.objects.get(id=1)
b.entry_set.set([e1, e2])
# 在这个例子中，e1和e2可以是完整的Entry实例，也可以是整数的主键值。
```
如果clear()方法可用，那么在将可迭代对象中的成员添加到集合中之前，将从entry_set中删除所有已经存在的对象。如果clear()方法不可用，那么将直接添加可迭代对象中的成员而不会删除所有已存在的对象。

这节中的每个反向操作都将立即在数据库内执行。所有的增加、创建和删除操作也将立刻自动地保存到数据库内。

## 使用原生SQL语句
若你发现需要编写的 SQL 查询语句太过复杂，以至于 Django 的数据库映射无法处理，你可以回归手动编写 SQL。Django 针对编写原生 SQL 有几个选项；参考 [执行原生 SQL 查询](https://docs.djangoproject.com/zh-hans/3.1/topics/db/sql/)。

最后，Django 数据库层只是一种访问数据库的接口，理解这点非常重要。你也可以通过其它工具，编程语言或数据库框架访问数据库；Django 并没有对数据库数据库做啥独有的操作。