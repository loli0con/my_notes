# 聚合与注解
Django 数据库抽象 API 描述了使用 Django queries 来增删查改单个对象的方法。 然而，有时候你**要获取的值需要根据一组对象聚合后才能得到**。这个主题指南描述了如何使用 Django queries 来生成和返回聚合值的方法。

首先认识两个单词：
* aggregate: [ˈæɡrɪɡət ] ，聚合。做一些统计方面的工作。返回的是聚合后的数据字典。关键在于为整个QuerySet生成聚合。
* annotate: [ˈænəteɪt ]，注解。为返回的查询集添加一些额外的数据。返回的依然是查询集。关键在于为QuerySet中的每一个条目生成聚合。

本节以下面的模型为例，来记录多个网上书店的库存：
```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()

class Publisher(models.Model):
    name = models.CharField(max_length=300)

class Book(models.Model):
    name = models.CharField(max_length=300)
    pages = models.IntegerField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    rating = models.FloatField()
    authors = models.ManyToManyField(Author)  # 多对多，作者
    publisher = models.ForeignKey(Publisher, on_delete=models.CASCADE)  # 多对一，出版社
    pubdate = models.DateField()

class Store(models.Model):
    name = models.CharField(max_length=300)
    books = models.ManyToManyField(Book)  # 多对多，书籍
```

## 速查表
下面是根据以上模型执行常见的聚合查询范例，注意阅读其中的注释文字：
```python
# 书籍的总数
>>> Book.objects.count()
2452

# BaloneyPress出版社出版的书籍总数
>>> Book.objects.filter(publisher__name='BaloneyPress').count()
73

# 所有书籍的平均价格
# 要注意！Avg，Count等聚合工具是由Django提供的，不是Python内置的，也不是你自己编写的。
>>> from django.db.models import Avg
>>> Book.objects.all().aggregate(Avg('price'))
{'price__avg': 34.35}   # 本来返回的是查询集，聚合后返回的是一个数据字典，字典的键名是有规律的

# 所有书籍中最高的价格
>>> from django.db.models import Max
>>> Book.objects.all().aggregate(Max('price'))
{'price__max': Decimal('81.20')}

# 所有书籍中最高价和平均价的差
>>> from django.db.models import FloatField
>>> Book.objects.aggregate(
...     price_diff=Max('price', output_field=FloatField()) - Avg('price'))
{'price_diff': 46.85}


# 下面是annonte注解的用法

# 每一个被过滤出来的出版社对象都被附加了一个"num_books"属性，这个属性就是所谓的注释
# 和aggregate不同，annotate返回的依然是查询集，添加了私货的查询集。
>>> from django.db.models import Count
>>> pubs = Publisher.objects.annotate(num_books=Count('book'))
>>> pubs
<QuerySet [<Publisher: BaloneyPress>, <Publisher: SalamiPress>, ...]>
>>> pubs[0].num_books
73

# 每一个出版商都被附加了两个额外的属性，分别表示好评率大于5和好评率小于等于5的书籍的总数
>>> from django.db.models import Q
# 统计每个出版社中好评率大于5的书籍的数量
>>> above_5 = Count('book', filter=Q(book__rating__gt=5))
>>> below_5 = Count('book', filter=Q(book__rating__lte=5))

# 要理解这里的链式调用含义。annotate不是filter，不会增减查询集的元素。
# 所以Publisher.objects实际上等于pubs=Publisher.objects.all()
# 第一个annotate为pubs增加below_5属性，第二个annotate又再次增加above_5属性
# 虽然是链式调用，但不是过滤行为，而是追加行为
>>> pubs = Publisher.objects.annotate(below_5=below_5).annotate(above_5=above_5)
>>> pubs[0].above_5
23
>>> pubs[0].below_5
12

# 这个比较复杂。首先获取了所有的出版社。其次统计每个出版社的发行书籍数量，保存在num_books属性中。
# 然后对所有的出版社进行排序，根据num_books属性进行反向由多到少排序，最后切片获取前5个出版社。
# The top 5 publishers, in order by number of books.
>>> pubs = Publisher.objects.annotate(num_books=Count('book')).order_by('-num_books')[:5]
>>> pubs[0].num_books
1323
```

## 内置聚合函数
Django提供了一系列[内置聚合函数](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/#aggregation-functions)，它们都位于django.db.models模块中。我们在使用aggregate和annotate的时候，需要使用这些内置的统计方法，不能自己瞎写一个类似Avg的方法。

对空的查询集进行聚合返回的是None而不是0。唯一的例外是Count聚合，它返回的是0。

所有的聚合函数都接收以下参数：
* expressions：引用模型上字段的一个字符串，或者一个query expression。
* output_field：可选参数，用于指定你要返回哪些模型字段的值，即表示返回值对应的的模型字段。
* filter：一个可选的Q对象，用于过滤聚合的对象。
* **extra：可以给聚合函数生成的SQL提供额外信息的关键字参数，可以为聚合生成的 SQL 提供额外的上下文。

### Avg
`class Avg(expression, output_field=None, distinct=False, filter=None, **extra)`

返回给定表达式的平均值，它必须是数值，除非指定不同的output_field。如果 distinct=True，Avg 返回非重复值的平均值。
```
默认的别名：<field>__avg
返回类型：float（或指定任何output_field的类型）
```

### Count
`class Count(expression, distinct=False, filter=None, **extra)`

返回与expression相关的对象的个数。

```
默认的别名：<field>__count
返回类型：int
有一个可选的参数：distinct。如果distinct=True，Count 将只计算唯一的实例。默认值为False。
```

### Max
`class Max(expression, output_field=None, filter=None, **extra)`

返回expression的最大值。
```
默认的别名：<field>__max
返回类型：与输入字段的类型相同，如果提供则为output_field类型
```

### Min
`class Min(expression, output_field=None, filter=None, **extra）`

返回expression的最小值。
```
默认的别名：<field>__min
返回类型：与输入字段的类型相同，如果提供则为output_field类型
```

### StdDev
`class StdDev(expression, output_field=None, sample=False, filter=None, **extra)`

返回expression的标准差（统计学术语）。
```
默认的别名：<field>__stddev
返回类型：float
有一个可选的参数：sample。默认情况下，返回群体的标准差。如果sample=True，返回样本的标准差。
```

### Sum
`class Sum(expression, output_field=None, distinct=False, filter=None, **extra)`

计算expression的所有值的和。
```
默认的别名：<field>__sum
返回类型：与输入字段的类型相同，如果提供则为output_field类型
```

### Variance
`class Variance(expression, output_field=None, sample=False, filter=None, **extra)`

返回expression的方差。
```
默认的别名：<field>__variance
返回类型：float
有一个可选的参数：sample。默认情况下，返回总体方差。 但是，如果 sample=True，则返回值将是样本方差。
```

## 在 QuerySet 上生成聚合
Django 提供了两种生成聚合的方法。第一种方法是为整个 QuerySet 生成汇总值，也就是aggregate聚合。

这里的核心是：对查询集的整体统计！

比如你想要计算所有在售书的平均价格。Django 的查询语法提供了一种用来描述所有图书集合的方法：
```python
>>> Book.objects.all()
```

可以通过在 QuerySet 后添加 aggregate() 聚合子句来计算 QuerySet 对象的汇总值。
```python
>>> from django.db.models import Avg
>>> Book.objects.all().aggregate(Avg('price'))   # 计算平均价
{'price__avg': 34.35}
```

事实上，本例中的 all() 是多余的，所以可以简化成这样的:
```python
>>> Book.objects.aggregate(Avg('price'))
{'price__avg': 34.35}
```

传递给 aggregate() 的参数描述了我们想要做什么类型的统计，以及统计的对象和设定。在这个例子里，要计算的就是 Book 模型上的 price 字段的平均值。

aggregate() 是 QuerySet 的一个终结子句，使用后将返回“名称：值”的字典，其中“名称”就是聚合值的键名，“值”就是计算出的聚合结果。默认情况下，“名称”会根据字段名和聚合函数名自动生成，比如price__avg。如果你想指定聚合值的键名，可以在编写聚合子句的时候指定，比如下面的average_price：
```python
>>> Book.objects.aggregate(average_price=Avg('price'))
{'average_price': 34.35}
```

如果你想生成更多的聚合内容，你需要在 aggregate() 子句中加入其它参数即可，以逗号分隔。比如假设我们也想知道所有书中最高和最低的价格，我们可以写这样的查询：
```python
>>> from django.db.models import Avg, Max, Min
>>> Book.objects.aggregate(Avg('price'), Max('price'), Min('price'))
{'price__avg': 34.35, 'price__max': Decimal('81.20'), 'price__min': Decimal('12.99')}
```

## 为 QuerySet 中的每一个条目生成聚合
汇总生成值的另一个办法是为 QuerySet 的每一个对象生成独立汇总，也就是annotate注解。

这里的核心是：对查询集中每个对象的单独统计。

比如，如果你想检索书籍列表，你可能想知道每一本书有多少作者。每一本书与作者有多对多的关系，我们想在 QuerySet 中为每一本书总结这个关系。

使用 annotate() 子句可以生成每一个对象的汇总。当指定 annotate() 子句，QuerySet 中的每一个对象将对指定值进行汇总。

这些汇总语法规则与 aggregate() 子句的规则相同。annotate() 的每一个参数描述了一个要计算的聚合。比如，注解（annotate）每本书的作者数量：
```python
# 统计每本书有多少个作者
>>> from django.db.models import Count
>>> q = Book.objects.annotate(Count('authors'))


# 查询集第一个对象
>>> q[0]
<Book: The Definitive Guide to Django>
>>> q[0].authors__count
2

# 查询集的第二个对象
>>> q[1]
<Book: Practical Django Projects>
>>> q[1].authors__count
1
```

与 aggregate() 一样，注解的名称是根据聚合函数和被聚合的字段名自动生成的。当你在指定注解的时候，你可以通过提供一个别名重写这个默认名：
```python
>>> q = Book.objects.annotate(num_authors=Count('authors'))
>>> q[0].num_authors
1
>>> q[1].num_authors
1
```

与 aggregate() 不同的是，annotate() 不是终结子句。annotate() 子句的输出依然是个 QuerySet，可以被链式调用；这个 QuerySet 可以被其他的 QuerySet API操作，包括 filter(),order_by()，甚至可以再次annotate()，如一开始的例子中所示。

## 组合多个聚合时可能的bug
使用 annotate() 组合多个聚合将产生错误的结果，因为它使用连接(joins)而不是子查询，下面演示了错误的发生：
```python
>>> book = Book.objects.first()
>>> book.authors.count()
2
>>> book.store_set.count()
3
>>> q = Book.objects.annotate(Count('authors'), Count('store'))
>>> q[0].authors__count
6
>>> q[0].store__count
6
```

对大部分聚合来说，没办法避免这个问题，但是，Count 聚合可以使用 distinct 参数来避免：
```python
>>> q = Book.objects.annotate(Count('authors', distinct=True), Count('store', distinct=True))
>>> q[0].authors__count
2
>>> q[0].store__count
3
```

想要搞清楚你的查询发生了什么问题，可以 QuerySet 中检查一下query 属性。

## 连接(Joins)和聚合
到目前为止，我们已经处理了被查询模型字段的聚合。然而，有时候想聚合的值属于你正在查询模型的关联模型。

在聚合函数里面指定聚合的字段时，Django 允许你在过滤相关字段的时候使用相同的双下划线表示法。Django 将处理任何需要检索和聚合的关联值的表连接(table joins)。

比如，要寻找每个书店提供的书籍价格区间，你可以使用这个注解(annotation)：
```python
>>> from django.db.models import Max, Min
>>> Store.objects.annotate(min_price=Min('books__price'), max_price=Max('books__price'))
```

Django 会检索 Store 模型，连接（通过多对多关系） Book 模型，并且聚合书籍模型的价格字段来获取最大最小值。

相同规则应用于 aggregate() 从句。如果你想知道所有书店正在销售的所有书籍的最低最高价，你可以使用这个聚合：
```python
>>> Store.objects.aggregate(min_price=Min('books__price'), max_price=Max('books__price'))
```

关系链接的深度可以尽可能深。比如，要提取所出售的书籍中最年轻的作者年龄，你可以写这样的查询：
```python
>>> Store.objects.aggregate(youngest_age=Min('books__authors__age'))
```

## 反向关系
类似于 跨关系查询 ，你正在查询的在模型和模型字段上的聚合和注解(annotations)可以包含反向关系。关系模型的小写名和双下划线也可以用在这里。

比如注解每个出版社发行的书籍总数：
```python
>>> from django.db.models import Avg, Count, Min, Sum
>>> Publisher.objects.annotate(Count('book'))
# (查询结果里的每一个 Publisher 会有多余的属性—— book__count 。)
```

我们也可以找出最古老的一本：
```python
>>> Publisher.objects.aggregate(oldest_pubdate=Min('book__pubdate'))
# (结果字典中会有一个叫 'oldest_pubdate' 的键。如果没有指定这样的别名，它将会是一个很长的名字 'book__pubdate__min' 。)
```

它不仅仅用于外键，它也适用于多对多关系。
比如，查询每个作者所编写书籍的总页数：
```python
>>> Author.objects.annotate(total_pages=Sum('book__pages'))
# （结果集里的每一个 Author 会有一个额外的属性——total_pages）如果没有指定这样的别名，它将会是一个很长的名字 book__pages__sum）
```

或查询所有作者写的所有书籍的平均评分：
```python
>>> Author.objects.aggregate(average_rating=Avg('book__rating'))
# （结果字典会有一个叫 'average_rating' 的键。如果没有指定这样的别名，它将会是一个很长的名字 'book__rating__avg'。）
```

## 聚合和其他 QuerySet 子句
### filter() 和 exclude()
聚合的同时也可以过滤。其实本质上就是先过滤出一个查询子集，而不是all()，然后在此基础上进行聚合或注解。

比如统计每本，以Django开头的书籍 的作者 的数量：
```python
>>> from django.db.models import Avg, Count
>>> Book.objects.filter(name__startswith="Django").annotate(num_authors=Count('authors'))
```

下面则是统计以Django开头的书籍的平均价格：
```python
>>> Book.objects.filter(name__startswith="Django").aggregate(Avg('price'))
```

### 过滤注解
不但可以过滤查询集，注解过的值也可以使用过滤器。注解的别名可以和任何其他模型字段一样使用 filter() 和 exclude() 子句。

比如，要生成多名作者的书籍列表，可以使用这种查询：
```python
>>> Book.objects.annotate(num_authors=Count('authors')).filter(num_authors__gt=1)
```

要这么理解，首先生成了一个带有注解的查询集，然后在这个查询集的基础上过滤掉单个作者编写的书，生成一个包含注解的过滤查询集。一定要注意annotate和filter的顺序和参数的写法。

你甚至可以在聚合函数中使用 filter 语句。比如：
```python
>>> highly_rated = Count('book', filter=Q(book__rating__gte=7))
>>> Author.objects.annotate(num_books=Count('book'), highly_rated_books=highly_rated)
```
这会获得一个作者的查询集，每个作者都会获得两个注解，一个是它写过的书的数量，一个是它写过的书中获得好评率高于等于7的数量。每一个Author 都会有 num_books 和 highly_rated_books 两个属性。

建议：避免在单个注解和聚合中使用 filter 语句。使用 QuerySet.filter() 来过滤会更高效。聚合 函数内的filter 语句只在使用具有不同条件的相同关系的两个或以上的聚合时有用。

### annotate() 和 filter() 子句的顺序
当开发一个涉及 annotate() 和 filter() 子句的复杂查询时，要特别注意应用于 QuerySet 的子句的顺序。

当一个 annotate() 子句应用于查询，会根据查询状态来计算注解，直到请求的注解为止。这实际上意味着 filter() 和 annotate() 不是可交换的操作。

比如：
* 出版者A有两本评分4和5的书。
* 出版者B有两本评分1和4的书。
* 出版者C有一本评分1的书。

下面就是 Count 聚合的例子：
```python
>>> a, b = Publisher.objects.annotate(num_books=Count('book', distinct=True)).filter(book__rating__gt=3.0)
>>> a, a.num_books
(<Publisher: A>, 2)
>>> b, b.num_books
(<Publisher: B>, 2)

>>> a, b = Publisher.objects.filter(book__rating__gt=3.0).annotate(num_books=Count('book'))
>>> a, a.num_books
(<Publisher: A>, 2)
>>> b, b.num_books
(<Publisher: B>, 1)
```
两个查询返回出版者列表，这些出版者至少有一本评分3的书，因此排除了C。

在第一个查询里，注解优先于过滤器，因此过滤器没有影响注解。distinct=True 用来避免 前面说的查询bug。

第二个查询每个出版者评分3以上的书籍数量。过滤器优先于注解，因此过滤器约束计算注解时考虑的对象。

这里是另一个关于 Avg 聚合的例子：
```python
>>> a, b = Publisher.objects.annotate(avg_rating=Avg('book__rating')).filter(book__rating__gt=3.0)
>>> a, a.avg_rating
(<Publisher: A>, 4.5)  # (5+4)/2
>>> b, b.avg_rating
(<Publisher: B>, 2.5)  # (1+4)/2

>>> a, b = Publisher.objects.filter(book__rating__gt=3.0).annotate(avg_rating=Avg('book__rating'))
>>> a, a.avg_rating
(<Publisher: A>, 4.5)  # (5+4)/2
>>> b, b.avg_rating
(<Publisher: B>, 4.0)  # 4/1 (book with rating 1 excluded)
```

第一个查询请求至少有一本评分3以上的书籍的出版者的书籍平均分。第二个查询只请求评分3以上的作者书籍的平均评分。

我们通常很难凭直觉了解ORM如何将复杂的查询集转化为SQL查询，因此当有疑问时，请使用str(queryset.query) 检查SQL，并编写大量的测试。

### order_by()
注解可以当做基本排序来使用。

比如，通过书籍的作者数量来对书籍的 QuerySet 排序，可以使用下面的查询：
```python
>>> Book.objects.annotate(num_authors=Count('authors')).order_by('num_authors')
```

### values()
通常，注解值会添加到每个对象上，即一个被注解的 QuerySet 将会为初始 QuerySet 的每个对象返回一个结果集。然而，当使用 values() 子句来对结果集进行约束时，生成注解值的方法会稍有不同。不是在原始 QuerySet 中对每个对象添加注解并返回，而是根据定义在 values() 子句中的字段组合先对结果进行分组，再对每个单独的分组进行注解，这个注解值是根据分组中所有的对象计算得到的。

下面是一个关于作者的查询例子，查询每个作者所著书的平均评分：
```python
>>> Author.objects.annotate(average_rating=Avg('book__rating'))
```

这段代码返回的是数据库中的所有作者及其所著书的平均评分。

但是如果你使用 values() 子句，结果会稍有不同：
```python
>>> Author.objects.values('name').annotate(average_rating=Avg('book__rating'))
```

在这个例子中，作者会按名字分组，所以你只能得到不重名的作者分组的注解值。这意味着如果你有两个作者同名，那么他们原本各自的查询结果将被合并到同一个结果中；两个作者的所有评分都将被计算为一个平均分。

### annotate() 和 values() 的顺序
和使用 filter() 一样，作用于某个查询的 annotate() 和 values() 子句的顺序非常重要。如果 values() 子句在 annotate() 之前，就会根据 values() 子句产生的分组来计算注解。

然而如果 annotate() 子句在 values() 之前，就会根据整个查询集生成注解。这种情况下，values() 子句只能限制输出的字段。

举个例子，如果我们颠倒上个例子中 values() 和 annotate() 的顺序：
```python
>>> Author.objects.annotate(average_rating=Avg('book__rating')).values('name', 'average_rating')
```

这段代码将为每个作者添加一个唯一注解，但只有作者姓名和 average_rating 注解会返回在输出结果中。

你应该也会注意 average_rating 已经明确包含在返回的值列表中。这是必需的，因为 values() 和 annotate() 子句的顺序。

如果 values() 子句在 annotate() 子句之前，任何注解将自动添加在结果集中。然而，如果 values() 子句应用在 annotate() 子句之后，则需要显式包含聚合列。

### 与默认排序或 order_by() 交互
在选择输出数据的时候，将使用在查询集的 order_by() 中提及的字段（或模型上的默认排序中使用的字段），即使它们不是其他在 values() 中指定的调用。这些额外的字段用于将“相似”的结果分组在一起，它们可以使其他相同的结果行看起来是分开的。尤其是在数数的时候。

举个例子，假设你有这样的模型：
```python
from django.db import models

class Item(models.Model):
    name = models.CharField(max_length=10)
    data = models.IntegerField()

    class Meta:
        ordering = ["name"]
```

这里的重点是在 name 字段上的默认排序。如果你想计算每一个不同的 data 值出现次数，你可以这样做：
```python
# Warning: not quite correct!
Item.objects.values("data").annotate(Count("id"))
```

它将通过共同的 data 值对 Item 对象进行分组，然后在每一组里计算 id 值的数量。只可惜它不能正常工作。通过 name 进行的默认排序也在分组中起作用，因此这个查询将按不同的 (data, name) 分组，这不是你想要的。相反，你可以构建这样的查询集：
```python
Item.objects.values("data").annotate(Count("id")).order_by()
```

清除任何查询中的排序。你也可以通过 data 排序，没有任何有害影响，因为它已经在查询中发挥了作用。

这个行为与 distinct() 的查询文档指出的行为相同，一般规则是一样的：通常情况下，你不希望额外的列在结果中发挥作用，因此要清除排序，或者至少确保它只限于您在 values() 调用中选择的那些字段。

你可能会问为什么 Django 没有移除无关的列。主要原因就是与 distinct() 和其他地方的一致性:Django从不删除你指定的排序约束（我们不能改变其他方法的行为，因为这会违反我们的应用编程接口的稳定性政策）。

### 聚合注解
你也可以在注解结果上生成聚合。当你定义 aggregate() 子句时，你提供的聚合可以引用任何定义在查询中 annotate() 子句的别名。

比如，如果你想计算每本书的平均作者数，首先使用作者数注解书籍集合，然后引用注解字段聚合作者数：
```python
>>> from django.db.models import Avg, Count
>>> Book.objects.annotate(num_authors=Count('authors')).aggregate(Avg('num_authors'))
{'num_authors__avg': 1.66}
```