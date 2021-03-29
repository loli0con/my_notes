# Field
本文档包含Field类的所有API参考，是对模型字段的详尽描述。

## 字段选项
字段选项在实例化字段类时作为参数传入。  
以下参数对所以字段类型均有效，且是可选的。
```python
...
my_field = models.XxxxxxField(
    max_length=30,  # 不是通用的选项
    null=False,
    blank=False,
    choices=[
        ("实际值1", "可读名称1"),
        ("实际值2", "可读名称2"),
        ...
    ]
    )
...
```
> 字段会涉及到一个概念——验证，在这里先不展开。这里简单描述一下：Django使用你定义的模型类，按照规则生成一个表单。这个表单可以在前端页面上给用户进行填写（前端页面会有验证）。Django获取到用户所填的表单后，根据规则进行验证（后端也会进行验证）。

### null
默认值：False  
如果是 True， Django 将在数据库中存储空值为 NULL。

对于保存字符串类型数据的字段，请尽量避免将此参数设为True，那样会导致两种‘没有数据’的情况，一种是`NULL`，另一种是空字符串`''`。Django 的惯例是使用空字符串而不是 NULL。

该选项将作用于**数据库**层面

### blank
默认值：False  
如果是 True ，该字段允许为空。

该选项将作用于**验证**层面，例如*表单验证*

### choices
该字段接收一个序列作为参数，该参数被称为“选择”。序列里的元素是二元组，每个元组中的第一个元素是要在模型上设置的实际值（数据库里的值），第二个元素是人可读的名称（页面上显示的值）。

如果给定了选择，它们会被**模型验证**强制执行，默认的表单部件将是一个带有这些选择的选择框（选择框标签），而不是标准的文本字段。

注意：每当 choices 的顺序变动时将会创建新的迁移。
#### get_FOO_display
对于每一个设置了 choice 的模型字段，Django 会添加一个方法来检索字段当前值的可读名称。该方法是**get_FOO_display()**，其中FOO是模型字段的名字。
```python
from django.db import models

class Person(models.Model):
    SHIRT_SIZES = (
        ('S', 'Small'),
        ('M', 'Medium'),
        ('L', 'Large'),
    )
    name = models.CharField(max_length=60)
    shirt_size = models.CharField(max_length=2, choices=SHIRT_SIZES)
```
```python
>>> p = Person(name="Fred Flintstone", shirt_size="L")  # shirt_size="L" 给模型字段shirt_size设置了选项，即设置choice
>>> p.save()
>>> p.shirt_size
'L'
>>> p.get_shirt_size_display()
'Large'
```

#### 动态选择
选择(*choices参数*)可以是任何序列对象——不一定是列表或元组。这让你可以动态地构造选择。

#### 前端表现
除非 blank=False 与 default 一起设置在字段上，否则包含 "---------" 的标签将与选择框一起呈现。要覆盖这种行为，可以在 choices 中添加一个包含 None 的元组，例如 (None, 'Your String For Display') 。

#### 枚举类型
Django 还提供了枚举类型，你可以通过将其子类化来简洁地定义选择。  
枚举类型有：TextChoices、IntegerChoices 和 Choices 类。

```python
from django.utils.translation import gettext_lazy as _

class Student(models.Model):

    # 枚举类
    class YearInSchool(models.TextChoices):
        FRESHMAN = 'FR', _('Freshman')
        SOPHOMORE = 'SO', _('Sophomore')
        JUNIOR = 'JR', _('Junior')
        SENIOR = 'SR', _('Senior')
        GRADUATE = 'GR', _('Graduate')

    year_in_school = models.CharField(
        max_length=2,
        choices=YearInSchool.choices,
        default=YearInSchool.FRESHMAN,
    )

    def is_upperclass(self):
        return self.year_in_school in {
            self.YearInSchool.JUNIOR,
            self.YearInSchool.SENIOR,
        }
```
> [gettext_lazy示例介绍](https://blog.csdn.net/u011519550/article/details/105037945)，用于国际化翻译的，和本例的内容其实没关系

##### 特性
* 枚举成员值是构造具体数据类型时要使用的参数元组。Django 支持在这个元组的末尾添加一个额外作为人可读的名称的字符串值 label。label 可以是一个惰性的可翻译字符串。因此，在大多数情况下，成员值将是一个 (value, label) 二元组。如果没有提供元组，或者最后一项不是（惰性）字符串，label 从成员名自动生成（生成规则：用空格代替下划线并使用标题大小写）。
* 在值上添加 .label 属性，以返回人类可读的名称。
* 在枚举类中添加了一些自定义属性—— .choices、.labels、.values 和 .names ——以便于访问枚举的这些单独部分的列表。在字段定义中，使用 .choices 作为一个合适的值传递给 choices。
* 强制使用 enum.unique() 是为了确保不能多次定义值。在选择一个字段时，不太可能会出现这种情况。

> 惰性的可翻译字符串参见上面的“gettext_lazy示例介绍”  

##### 详细示例
![model_field+截屏2021-03-10 21.21.41](https://raw.githubusercontent.com/loli0con/picgo/master/images/model_field%2B%E6%88%AA%E5%B1%8F2021-03-10%2021.21.41.png%2B2021-03-10-21-23-00)

![model_field+截屏2021-03-10 21.25.28](https://raw.githubusercontent.com/loli0con/picgo/master/images/model_field%2B%E6%88%AA%E5%B1%8F2021-03-10%2021.25.28.png%2B2021-03-10-21-29-18)

![model_field+截屏2021-03-10 21.27.00](https://raw.githubusercontent.com/loli0con/picgo/master/images/model_field%2B%E6%88%AA%E5%B1%8F2021-03-10%2021.27.00.png%2B2021-03-10-21-29-18)

##### 其他用法
###### 使用 Enum Functional API 
需要注意的是，标签是自动生成的
```python
>>> MedalType = models.TextChoices('MedalType', 'GOLD SILVER BRONZE')
>>> MedalType.choices
[('GOLD', 'Gold'), ('SILVER', 'Silver'), ('BRONZE', 'Bronze')]
>>> Place = models.IntegerChoices('Place', 'FIRST SECOND THIRD')
>>> Place.choices
[(1, 'First'), (2, 'Second'), (3, 'Third')]
```
###### 支持 int 或 str 以外的具体数据类型
将 Choices 和所需的具体数据类型子类化
```python
class MoonLandings(datetime.date, models.Choices):
    APOLLO_11 = 1969, 7, 20, 'Apollo 11 (Eagle)'
    APOLLO_12 = 1969, 11, 19, 'Apollo 12 (Intrepid)'
    APOLLO_14 = 1971, 2, 5, 'Apollo 14 (Antares)'
    APOLLO_15 = 1971, 7, 30, 'Apollo 15 (Falcon)'
    APOLLO_16 = 1972, 4, 21, 'Apollo 16 (Orion)'
    APOLLO_17 = 1972, 12, 11, 'Apollo 17 (Challenger)'
```
![model_field+截屏2021-03-10 21.33.11](https://raw.githubusercontent.com/loli0con/picgo/master/images/model_field%2B%E6%88%AA%E5%B1%8F2021-03-10%2021.33.11.png%2B2021-03-10-21-33-51)
##### 注意事项
因为具有具体数据类型的枚举要求所有值都与类型相匹配，所以不能通过创建一个值为 None 的成员来覆盖*空白标签*。相反，在类上设置 \_\_empty\_\_ 属性：
```python
class Answer(models.IntegerChoices):
    NO = 0, _('No')
    YES = 1, _('Yes')

    __empty__ = _('(Unknown)')
```

### db_column
字段要使用的数据库列名。如果没有给出列名，将使用字段名（也就是该字段在模型类中的属性名）。

### db_index
默认值：False  
如果是 True，将为该字段创建数据库索引。

### db_tablespace
如果这个字段有索引，那么要为这个字段的索引使用的**数据库表空间**的名称。默认是项目的 DEFAULT_INDEX_TABLESPACE 设置（如果有设置），或者是模型的 db_tablespace （如果有）。如果后端不支持索引的表空间，则忽略此选项。
> Django文档里面提及表空间的时候，一般都配合PostgreSQL来说明，这里也贴一个PostgreSQL的文档来简述一下表空间。http://www.postgres.cn/docs/9.4/manage-ag-tablespaces.html

### default
该字段的默认值。可以是一个值或者是个可调用的对象，如果是个可调用对象，每次实例化模型时都会调用该对象。

默认值不能是一个可更改的对象（模型实例、list、set 等），因为对该对象同一实例的引用将被用作所有新模型实例的缺省值。  
参考：https://zhuanlan.zhihu.com/p/61553610  
相反，将所需的默认值包裹在一个可调用对象中。

lambda 不能用于 default 等字段选项，因为它们不能被迁移序列化。

对于像 ForeignKey 这样映射到模型实例的字段，默认应该是它们引用的字段的值（默认是 pk 除非 to_field 被设置了），而不是模型实例。

当创建新的模型实例且没有为该字段提供值时，使用默认值。当字段是主键时，当字段设置为``None``时，也使用默认值。

### editable
默认值：True  
如果是 False，该字段将不会在管理或任何其他 ModelForm 中显示。在*模型验证*中也会跳过。
> 管理：Django自带的admin

### error_messages
error_messages 参数可以让你覆盖该字段引发的默认消息。传入一个与你想覆盖的错误信息相匹配的键值的字典。

错误信息键包括 null、blank、invalid、invalid_choice、unique 和 unique_for_date。

> Django在验证该字段时如果失败了，则会引起异常，也就是上报错误。上报错误的时候会有默认的消息（对该错误的详细描述），使用error_messages参数可以覆盖默认的消息。

### help_text
额外的“帮助”文本，随表单控件一同显示。即便你的字段未用于表单，它对于生成文档也是很有用的。

请注意，在自动生成的表格中，这个值不是 HTML 转义的。如果你愿意的话，你可以在 help_text 中加入 HTML。

### primary_key
默认值：False  
如果是 True，将该字段设置为该模型的主键。

如果你没有为模型中的任何字段指定 primary_key=True，Django 会自动添加一个 AutoField 来保存主键，所以你不需要为你的任何字段设置 primary_key=True，除非你想覆盖默认的主键行为。

primary_key=True 意味着 null=False 和 unique=True。一个模型只允许有一个主键。

主键字段是只读的。如果您改变了现有对象的主键值，然后将其保存，则会在旧对象旁边创建一个新对象。

### unique
默认值：False  
如果是 True，这个字段必须在整个表中保持值唯一。

这是在数据库级别和模型验证中强制执行的。如果你试图保存一个在 unique 字段中存在重复值的模型，模型的 save() 方法将引发 django.db.IntegrityError。

除了 ManyToManyField 和 OneToOneField 之外，该选项对所有字段类型有效。

请注意，当 unique 为 True 时，你不需要指定 db_index，因为 unique 意味着创建一个索引。

### unique_for_date
将其设置为 DateField 或 DateTimeField 的名称，要求该字段的日期字段值是唯一的。

例如，如果你的字段 title 有 unique_for_date="pub_date"，那么 Django 就不允许输入两条相同 title 和 pub_date 的记录。

> 举个栗子，如果你有一个名叫title的字段，并设置了参数unique_for_date="pub_date"，那么Django将不允许有两个模型对象具备同样的title和pub_date。有点类似联合约束。

请注意，如果将其设置为指向 DateTimeField，则只考虑该字段的日期部分。此外，当 USE_TZ 为 True 时，检查将在对象保存时的 当前时区 中进行。

这在模型验证过程中由 Model.validate_unique() 强制执行，但在数据库级别上不执行。如果任何 unique_for_date 约束涉及的字段不属于 ModelForm （例如，如果其中一个字段被列在``exclude``中，或者有 editable=False ）， Model.validate_unique() 将跳过对该特定约束的验证。

### unique_for_month
像 unique_for_date 一样，但要求字段对月份是唯一的。

### unique_for_year
像 unique_for_date 一样，但要求字段对年份是唯一的。

### verbose_name
为字段设置一个人类可读，更加直观的别名。如果没有给定详细名称，Django 会使用字段的属性名自动创建，并将下划线转换为空格。

### validators
要为该字段运行的验证器列表。

## 字段类型

### 整型
> 先从一个最基本的IntegerField讲起。

IntegerField 所代表的类型的是32位带符号的整数，即它支持存储值在 2^31-1 到 2^31 范围内的整数。

它使用 MinValueValidator 和 MaxValueValidator 根据默认数据库支持的值来验证输入。

#### Range 范围
IntegerField还有两个类似的“兄弟”，SmallIntegerField和BigIntegerField，它们分别对应着16位带符号的整数和64位带符号的整数。

#### Auto 自增长
从IntegerField的概念衍生出来，AutoField是一个具备自动递增功能的正整数。它会根据可用的ID自动递增，常用于主键。

类似地还有SmallAutoField和BigAutoField。
#### Positive 正数
* PositiveSmallIntegerField：16位无符号整数
* PositiveIntegerField：32位无符号整数
* PositiveBigIntegerField：64位无符号整数

### 浮点型
FloatField表示浮点数，类似python里面的float。

DecimalField表示固定精度的十进制数，类似python里面的Decimal。  
有两个必要的参数：
* DecimalField.max_digits：数字中允许的最大位数。请注意，这个数字必须大于或等于 decimal_places。
* DecimalField.decimal_places：与数字一起存储的小数位数。

### 字符型
#### 基础
CharField表示一个字符串字段，适用于小到大的字符串。  
有一个额外的必要参数：
CharField.max_length：字段的最大长度（以字符为单位）。max_length 是在数据库层和 Django 的验证中使用 MaxLengthValidator 强制执行的。

#### 拓展
> 这里的拓展，指的是数据保存在数据库时使用字符串来进行存储，所以它们可以说是“基于字符串的底层，拥有更多的功能”。
##### TextField
一个大的文本字段。该字段的默认表单部件是一个 Textarea。  
如果你指定了 max_length 属性，它将反映在自动生成的表单字段的 Textarea 部件中。但是，它并没有在模型或数据库层面被强制执行。使用一个 CharField 来实现。

##### EmailField
一个 CharField，使用 EmailValidator 来检查该值是否为有效的电子邮件地址。

##### URLField
URL 的 CharField，由 URLValidator 验证。

##### SlugField
Slug 是一个报纸术语。slug 是一个简短的标签，只包含字母、数字、下划线或连字符。它们一般用于 URL 中。  
它使用 validate_slug 或 validate_unicode_slug 进行验证。

###### 属性
* SlugField.allow_unicode：如果是 True，该字段除了接受 ASCII 字母外，还接受 Unicode 字母。默认值为 False。

##### UUIDField
一个用于存储通用唯一标识符的字段。使用 Python 的 UUID 类。

通用唯一标识符是 primary_key 的 AutoField 的一个很好的替代方案。数据库不会为你生成 UUID，所以建议使用 default ：
```python
import uuid
from django.db import models

class MyUUIDModel(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    # other fields

    # 请注意，一个可调用对象（省略括号）被传递给 default，而不是 UUID 的实例。
```

##### GenericIPAddressField
IPv4 或 IPv6 地址。  
如果允许空值，就必须允许 null 值，因为空值会被存储为 null。

###### 属性
* GenericIPAddressField.protocol：将有效输入限制为指定协议。接受的值是 'both' （默认）、'IPv4' 或 'IPv6'。匹配是不分大小写的。

* GenericIPAddressField.unpack_ipv4：解压 IPv4 映射地址，如 ::fffff:192.0.2.1。如果启用该选项，该地址将被解压为 192.0.2.1。默认为禁用。只有当 protocol 设置为 'both' 时才会启用。

##### JSONField
一个用于存储 JSON 编码数据的字段。在 Python 中，数据以其 Python 本地格式表示：字典、列表、字符串、数字、布尔值和 None。

###### 属性
* JSONField.encoder：一个可选的 json.JSONEncoder 子类，用于序列化标准 JSON 序列化器不支持的数据类型（例如 datetime.datetime 或 UUID ）。例如，你可以使用 DjangoJSONEncoder 类。默认为 json.JSONEncoder。
* JSONField.decoder：一个可选的 json.JSONDecoder 子类，用于反序列化从数据库中获取的值。该值将采用自定义编码器选择的格式（通常是字符串）。你的反序列化可能需要考虑到你无法确定输入类型的事实。例如，你有可能返回一个 datetime，实际上是一个字符串，而这个字符串恰好与 datetime 选择的格式相同。默认为 json.JSONDecoder。

##### FileField
一个文件上传字段。

默认情况下，该字段在HTML中表现为一个ClearableFileInput标签。在数据库内，我们实际保存的是一个字符串类型，默认最大长度100，可以通过max_length参数自定义。真实的文件是保存在服务器的文件系统内的。

有两个可选参数：
* FileField.upload_to：这个属性提供了一种设置上传目录和文件名的方式，可以有两种设置方式。
    ```python
    # 方式1：指定一个字符串值或一个 Path，它可能包含 strftime() 格式，它将被文件上传的日期／时间所代替（这样上传的文件就不会填满指定的目录）。
    class MyModel1(models.Model):
        # 文件被传至`MEDIA_ROOT/uploads`目录，MEDIA_ROOT由你在settings文件中设置
        upload = models.FileField(upload_to='uploads/')
        # 或者
        # 被传到`MEDIA_ROOT/uploads/2015/01/30`目录，增加了一个时间划分
        upload = models.FileField(upload_to='uploads/%Y/%m/%d/')

    # 方式2：一个可调用对象，如函数。这个函数将被调用以获得上传路径，包括文件名。这个可调用对象必须接受两个参数，并返回一个 Unix 风格的路径（带斜线），以便传给存储系统。
    def user_directory_path(instance, filename):
        #文件上传到MEDIA_ROOT/user_<id>/<filename>目录中
        return 'user_{0}/{1}'.format(instance.user.id, filename)

    class MyModel(models.Model):
        upload = models.FileField(upload_to=user_directory_path)
    ```
    ![model_field+截屏2021-03-13 20.34.24](https://raw.githubusercontent.com/loli0con/picgo/master/images/model_field%2B%E6%88%AA%E5%B1%8F2021-03-13%2020.34.24.png%2B2021-03-13-20-34-55)
* FileField.storage：一个存储对象，或是一个返回存储对象的可调用对象。它处理你的文件的存储和检索。

###### 使用FileField或者ImageField字段的步骤
1. 在settings文件中，配置MEDIA_ROOT，作为你上传文件在服务器中的基本路径（为了性能考虑，这些文件不会被储存在数据库中）。再配置个MEDIA_URL，作为公用URL，指向上传文件的基本路径。请确保Web服务器的用户账号对该目录具有写的权限。
2. 添加FileField或者ImageField字段到你的模型中，定义好upload_to参数，文件最终会放在MEDIA_ROOT目录的“upload_to”子目录中。
3. 所有真正被保存在数据库中的，只是指向你上传文件路径的字符串而已。可以通过url属性，在Django的模板中方便的访问这些文件。例如，假设你有一个ImageField字段，名叫mug_shot，那么在Django模板的HTML文件中，可以使用{{ object.mug_shot.url }}来获取该文件。其中的object用你具体的对象名称代替。
4. 可以通过name和size属性，获取文件的名称和大小信息。

###### FieldFile
当你访问一个模型上的 FileField 时，你会得到一个 FieldFile 的实例作为访问底层文件的代理。  
通过这个代理，我们可以进行一些文件操作，主要如下：
* `FieldFile.name`：获取文件名
* `FieldFile.size`：获取文件大小
* `FieldFile.url`：用于访问该文件的url
* `FieldFile.open(mode='rb')`：以指定的 mode 打开或重新打开与该实例相关的文件。与标准的 Python open() 方法不同，它不返回一个文件描述符。因为在访问底层文件时，底层文件是隐式打开的，所以除了重置底层文件的指针或改变 mode 之外，可能没有必要调用这个方法。
* `FieldFile.close()`：关闭文件
* `FieldFile.save(name, content, save=True)`：这个方法接收一个文件名和文件内容，并将它们传递给字段的存储类，然后将存储的文件与模型字段关联。如果你想手动将文件数据与模型上的 FileField 实例关联起来，那么 save() 方法用来持久化该文件数据。
取两个必要的参数：name 是文件的名称，content 是包含文件内容的对象。 可选的 save 参数控制在与该字段相关联的文件被更改后是否保存模型实例，默认为 True。
注意 content 参数应该是 django.core.files.File 的实例，而不是 Python 内置的文件对象。你可以从现有的 Python 文件对象构造一个 File 或者你可以从 Python 字符串中构建。
    ```python
    from django.core.files import File
    # Open an existing file using Python's built-in open()
    f = open('/path/to/hello.world')
    myfile = File(f)

    from django.core.files.base import ContentFile
    myfile = ContentFile("hello world")
    ```
* `FieldFile.delete(save=True)`：删除文件

##### ImageField
继承 FileField 的所有属性和方法，但也验证上传的对象是有效的图像。

除了 FileField 的特殊属性外， ImageField 也有 height 和 width 属性。  
为了方便查询这些属性，ImageField 有两个额外的可选参数：
* ImageField.height_field：每次保存模型实例时将自动填充图像的高度。
* ImageField.width_field：每次保存模型实例时将自动填充图像的宽度。

##### FilePathField
一个 CharField，其选择仅限于文件系统中某个目录下的文件名。
> 这句话有点绕，我用一个场景来解释，我想限定这个这个字段只能表示某个目录下的某个文件的名字。一个普通的CharField，可以表示任意的字符串，而这个FilePathField只能表示某个文件名，且还要是在“限定的目录下的”。

###### 参数
有一些特殊的参数，其中第一个参数是必须的：
1. FilePathField.path：一个目录的绝对文件系统路径，这个 FilePathField 应从该目录中获取其选择。path 也可以是一个可调用对象，可以是在运行时动态设置路径的函数。
2. FilePathField.match：一个正则表达式，作为一个字符串， FilePathField 将用于过滤文件名。请注意，正则表达式将被应用于基本文件名，而不是完整的路径。
3. FilePathField.recursive：True 或 False。默认为 False。指定是否包含 path 的所有子目录。即是否在 path 中的子目录是否有效，默认为False即表示对子目录不生效。
4. FilePathField.allow_files：True 或 False。 默认值是 True。 指定是否应该包含指定位置的文件。 这个或 allow_folders 必须是 True。
5. FilePathField.allow_folders： True 或 False。 默认为 False。 指定是否应该包含指定位置的文件夹。 这个或 allow_files 必须是 True。

### 字节
BinaryField
一个用于存储原始二进制数据的字段。可以指定为 bytes、bytearray 或 memoryview。

默认情况下，BinaryField 将 ediditable 设置为 False。

BinaryField 有一个额外的可选参数：
BinaryField.max_length，
字段的最大长度（以字符为单位）。最大长度在 Django 的验证中使用 MaxLengthValidator 强制执行。

### 时间日期
#### DateField
一个日期，在 Python 中用一个 datetime.date 实例表示。有一些额外的、可选的参数。
##### 参数
auto_now_add、auto_now 和 default 选项是相互排斥的。这些选项的任何组合都会导致错误。
###### DateField.auto_now
每次保存对象时，自动将该字段设置为现在。对于“最后修改”的时间戳很有用。

只有在调用 Model.save() 时，该字段才会自动更新。当以其他方式对其他字段进行更新时，如 QuerySet.update()，该字段不会被更新，尽管你可以在这样的更新中为该字段指定一个自定义值。

###### DateField.auto_now_add
当第一次创建对象时，自动将该字段设置为现在。对创建时间戳很有用。

#### TimeField
一个时间，在 Python 中用 datetime.time 实例表示。接受与 DateField 相同的自动填充选项。

#### DateTimeField
一个日期和时间，在 Python 中用一个 datetime.datetime 实例表示。与 DateField 一样，使用相同的额外参数。

#### DurationField
一个用于存储时间段的字段——在 Python 中用 timedelta 建模。在不同的数据库实现中有不同的表示方法。常用于进行时间之间的加减运算。
> 有坑，PostgreSQL等数据库之间有兼容性问题！

### 布尔
#### BooleanField
一个 true／false 字段。  
当 Field.default 没有定义时，BooleanField 的默认值是 None。

## 关系字段
Django 还定义了一组表示关系的字段，用来表示模型与模型之间的关系。

### ForeignKey
#### 定义
一个多对一的关系，通常被称为外键。  
需要两个位置参数：模型相关的类和 on_delete 选项。
> 外键要定义在‘多’的一方！

多对一字段的变量名一般设置为关联的模型的小写单数，而多对多则一般设置为小写复数。

#### 数据库表现
Django 在字段名后附加 "_id" 来创建数据库列名。  
在 ForeignKey 上会自动创建一个数据库索引，你可以通过设置 db_index 为 False 来禁用它。

#### 未定义模型
如果你需要在一个尚未定义的模型上创建关系，你可以使用模型的名称，而不是模型对象本身：
```python
from django.db import models

class Car(models.Model):
    manufacturer = models.ForeignKey(
        'Manufacturer',
        on_delete=models.CASCADE,
    )
    # ...

class Manufacturer(models.Model):
    # ...
    pass
```

#### 自关联
要创建一个递归关系——一个与自己有多对一关系的对象：使用 `models.ForeignKey('self', on_delete=models.CASCADE)`。

#### 抽象模型
在 抽象模型 上以这种方式定义的关系在模型被子类化为具体模型时得到解析：
![model_field+![model_field+截屏2021-03-14 01.18.15](httpsraw.githubusercontent.comloli0conpicgomasterimagesmodel_field%2B%E6%88%AA%E5%B1%8F2021-03-14%2001.18.15.png%2B2021-03-14-01-18-38)](https://raw.githubusercontent.com/loli0con/picgo/master/images/model_field%2B!%5Bmodel_field%2B%E6%88%AA%E5%B1%8F2021-03-14%2001.18.15%5D(httpsraw.githubusercontent.comloli0conpicgomasterimagesmodel_field%252B%25E6%2588%25AA%25E5%25B1%258F2021-03-14%252001.18.15.png%252B2021-03-14-01-18-38).png%2B2021-03-14-01-19-11)

#### 显式标注应用名
要引用定义在另一个应用程序中的模型，你可以明确地用完整的应用程序标签指定一个模型。下例假设Manufacturer模型存在于production这个app中，则Car模型的定义如下：
```python
class Car(models.Model):
    manufacturer = models.ForeignKey(
        'production.Manufacturer',      # 关键在这里！！
        on_delete=models.CASCADE,
    )
```

#### 参数
##### ForeignKey.on_delete
当一个由 ForeignKey 引用的对象被删除时，Django 将模拟 on_delete 参数所指定的 SQL 约束的行为。  
on_delete 不会在数据库中创建 SQL 约束。

该参数可选的值都内置在django.db.models中（全部为大写），包括：
* CASCADE：模拟SQL语言中的ON DELETE CASCADE约束，将定义有外键的模型对象同时删除！
* PROTECT：阻止上面的删除操作，弹出ProtectedError异常，防止删除被引用对象。
* RESTRICT：通过引发 RestrictedError来防止删除被引用的对象，与 PROTECT 不同的是，如果被引用的对象也引用了一个在同一操作中被删除的不同对象，但通过 CASCADE 关系，则允许删除被引用的对象。
* SET_NULL：将外键字段设为null，只有当字段设置了null=True时，方可使用该值。
* SET_DEFAULT：将外键字段设为默认值。只有当字段设置了default参数时，方可使用。
* DO_NOTHING：什么也不做。如果你的数据库后端强制执行引用完整性，这将导致一个 IntegrityError 除非你手动添加一个 SQL ON DELETE 约束条件到数据库字段。
* SET()：设置为一个传递给SET()的值或者一个回调函数的返回值。注意大小写。

###### RESTRICT举例说明
```python
# 假定有如下模型
class Artist(models.Model):
    name = models.CharField(max_length=10)

class Album(models.Model):
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE)

class Song(models.Model):
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE)
    album = models.ForeignKey(Album, on_delete=models.RESTRICT)
```

> 如果被引用的对象也引用了一个在同一操作中被删除的不同对象，但通过 CASCADE 关系，则允许删除被引用的对象。  
> ➡️将这句话翻译一下➡️  
> 如果被引用的*Album对象*也引用了一个在同一操作中被删除的*Artist对象*，但是通过*Song对象上定义的到artist的CASCADE关系*，则允许删除被引用的*Album*。

```python
>>> artist_one = Artist.objects.create(name='artist one')
>>> artist_two = Artist.objects.create(name='artist two')
>>> album_one = Album.objects.create(artist=artist_one)
>>> album_two = Album.objects.create(artist=artist_two)
>>> song_one = Song.objects.create(artist=artist_one, album=album_one)
>>> song_two = Song.objects.create(artist=artist_one, album=album_two)
>>> album_one.delete()
# Raises RestrictedError.
# album_one受到song_one上RESTRICT关系的影响，无法删除。

>>> artist_two.delete()
# Raises RestrictedError.
# artist_two要删除，则要删除album_two，album_two受到song_two上RESTRICT关系的影响，album_two无法删除，所以artist_two也无法删除；在这里song_two关联的artist是artist_one。

>>> artist_one.delete()
(4, {'Song': 2, 'Album': 1, 'Artist': 1})
# artist_one要删除，则要删除album_one，album_one受到song_one上RESTRICT关系的影响，“原本”album_one无法删除；但是artist_one删除时同时也要删除song_one，通过RESTRICT的作用，“现在”artist_one可以删除，所以整个删除过程顺利地执行了。
```

##### ForeignKey.limit_choices_to
当使用 ModelForm 或管理中渲染该字段时，设置该字段的可用选择限制（默认情况下，查询集中的所有对象都可以选择）。可以使用字典、 Q 对象，或者返回字典或 Q 对象的可调用对象。

例子：
```python
staff_member = models.ForeignKey(
    User,
    on_delete=models.CASCADE,
    limit_choices_to={'is_staff': True},
)
```
这样定义，则ModelForm的staff_member字段列表中，只会出现那些is_staff=True的Users对象，这一功能对于admin后台非常有用。

可以参考下面的方式，使用函数调用：
```python
def limit_pub_date_choices():
    return {'pub_date__lte': datetime.date.utcnow()}

# ...
limit_choices_to = limit_pub_date_choices
# ...
```

##### ForeignKey.related_name
用于从相关对象到这个对象的关系的名称。默认为模型的名称（小写）。这也是 related_query_name 的默认值（用于从目标模型反向过滤名称的名称）。

如果你不希望 Django 创建一个反向关系，可以将 related_name 设置为 '+' 或者以 '+' 结束。

##### ForeignKey.related_query_name
目标模型中反向过滤器的名称。  
它默认为 related_name 或 default_related_name 的值，否则默认为模型的名称。

###### related_name 和 related_query_name
related_name: 反向关联名，默认为以模型的小写加上`_set`作为反向关联名
related_query_name: 反向查询名，这个参数的默认值是定义有外键字段的模型的小写名，如果设置了*related_name*参数，那么就是这个参数值，如果在此基础上还指定了related_query_name的值，则是related_query_name的值。三者依次有优先顺序。

```python
class Tag(models.Model):
    article = models.ForeignKey(
        Article,
        on_delete=models.CASCADE,
        related_name="tagsss",
        related_query_name="taggg",
    )
    name = models.CharField(max_length=255)

# 如果不设置related_query_name的默认值
Article.objects.filter(tag__name="important")

# related_query_name在查询过滤时发挥作用
Article.objects.filter(taggg__name="important")

# 如果不设置related_name的默认值
Article.tag_set.all()

# related_name在反向关联时发挥作用
Article.tagsss.all()
```


##### ForeignKey.to_field
关联对象的字段。默认情况下，Django 使用相关对象的主键。如果你引用了一个不同的字段，这个字段必须有 unique=True。

##### ForeignKey.db_constraint
控制是否应该在数据库中为这个外键创建一个约束。默认值是 True，这几乎是你想要的；将其设置为 False 对数据完整性非常不利。

##### ForeignKey.swappable
控制迁移框架的反应，如果这个 ForeignKey 指向一个可交换的模型。如果它是 True ——默认值-——那么如果 ForeignKey 指向的模型与 settings.AUTH_USER_MODEL 的当前值相匹配（或其他可互换模型配置），则关系将在迁移中使用对配置的引用而不是直接对模型进行存储。
> 暂时没看到迁移框架，[TODO](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#django.db.models.ForeignKey.swappable)

### ManyToManyField
一个多对多的关系。需要一个位置参数：模型相关的类。
它的工作原理与 ForeignKey 完全相同，很多参数都是相同的。  
建议为多对多字段名使用复数形式。  
如果要创建一个关联自己的多对多字段，依然是通过'self'引用。

> 多对多的字段可以定义在任何的一方，请尽量定义在符合人们思维习惯的一方，但不要同时都定义，只能选择一个模型设置该字段（比如我们通常将披萨上的配料字段放在披萨模型中，而不是在配料模型中放置披萨字段）。

#### 数据库表现
在幕后，Django 创建了一个中间连接表来表示多对多的关系。默认情况下，这个表名是使用多对多字段的名称和包含它的模型的表名生成的。由于有些数据库不支持超过一定长度的表名，这些表名将被自动截断，并使用唯一性哈希，例如 author_books_9cdf。你可以使用 db_table 选项手动提供连接表的名称。

ManyToManyField 不支持 validators。  
null 没有效果，因为没有办法在数据库层面要求建立关系。

#### 参数
##### 同ForeignKey的参数
related_name、related_query_name、limit_choices_to、db_constraint、swappable。

注意：
* limit_choices_to 在使用 through 参数指定自定义中间表的 ManyToManyField 上使用时没有效果。
* 同时传递 db_constraint 和 through 会引发错误。

##### ManyToManyField.symmetrical
仅在自身上定义多对多字段关系时。考虑以下模型
```python
from django.db import models

class Person(models.Model):
    friends = models.ManyToManyField("self")
```
当 Django 处理这个模型时，它识别出它本身有一个 ManyToManyField，因此，它没有给 Person 类添加 person_set 属性。因为 ManyToManyField 被认为是对称的，也就是说，如果我是你的朋友，那么你就是我的朋友。

如果你不想让 self 的多对多关系对称，可以将 symmetrical 设置为 False。这样会强制 Django 添加反向关系的描述符，允许 ManyToManyField 关系是非对称的。

##### ManyToManyField.through
Django 会自动生成一个表来管理多对多关系。但是，如果你想手动指定中间表，你可以使用 through 选项来指定代表你要使用的中间表的 Django 模型。

最常见的使用场景是你需要为多对多关系添加额外的数据，比如添加两个人建立QQ好友关系的时间。

如果你没有指定一个显式的 through 模型，你仍然可以使用一个隐式的 through 模型类来直接访问为保持关联而创建的表。它有三个字段来链接模型。

如果源模型和目标模型不同，则会生成以下字段：
* id ：关系的主键。
* \<containing_model\>_id ：声明 ManyToManyField 的模型的 id。
* \<other_model\>_id ：ManyToManyField 指向的模型的 id。

如果 ManyToManyField 指向的来源和目标是相同的模型，下面的字段会生成：
* id ：关系的主键。
* from_\<model\>_id ：指向模型的实例（即源实例）的 id。
* to_\<model\>_id ：关系所指向的实例（即目标模型实例）的 id。

这个类可以像普通模型一样，用于查询给定模型实例的关联记录：

```python
Model.m2mfield.through.objects.all()
```

##### ManyToManyField.through_fields
只有当指定了一个自定义的中间模型时才会使用，Django 通常会决定使用中介模型的哪些字段来自动建立多对多的关系。

例子：
```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=50)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        through='Membership',       ## 自定义中间表
        through_fields=('group', 'person'),
    )

class Membership(models.Model):  # 这就是具体的中间表模型
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    inviter = models.ForeignKey(
        Person,
        on_delete=models.CASCADE,
        related_name="membership_invites",
    )
    invite_reason = models.CharField(max_length=64)
```

Membership模型中包含两个关联Person的外键，Django无法确定到底使用哪个作为和Group关联的对象。所以，在这个例子中，必须显式的指定through_fields参数，用于定义关系。

through_fields参数接收一个二元元组('field1', 'field2')，field1是指向定义有多对多关系的模型的外键字段的名称，这里是Membership中的‘group’字段（注意大小写），另外一个则是指向目标模型的外键字段的名称，这里是Membership中的‘person’，而不是‘inviter’。

再通俗的说，就是through_fields参数指定从中间表模型Membership中选择哪两个字段，作为关系连接字段。

##### ManyToManyField.db_table
要创建的用于存储多对多数据的表的名称。如果没有提供这个表名，Django 将根据以下表名创建一个默认表名：定义关系的模型表和字段本身的名称。

### OneToOneField
一对一的关系。概念上，这类似于 ForeignKey 与 unique=True，但关系的“反向”将直接返回一个单一对象。

需要一个位置参数：模型将与之相关的类。这与 ForeignKey 的工作原理完全相同。

最有用的是作为某种方式“扩展”另一个模型的主键；多表继承 是通过添加一个从子模型到父模型的隐式一对一关系来实现的。

如果没有为 OneToOneField 指定 related_name 参数，Django 将使用当前模型的小写名作为默认值。

举例如下：
```python
from django.conf import settings
from django.db import models

# 两个字段都使用一对一关联到了Django内置的auth模块中的User模型
class MySpecialUser(models.Model):
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
    supervisor = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='supervisor_of',
    )
```

这样下来，你的User模型将拥有下面的属性：
```python
>>> user = User.objects.get(pk=1)
>>> hasattr(user, 'myspecialuser')  # 使用当前模型的小写名作为默认值
True
>>> hasattr(user, 'supervisor_of')  # 指定 related_name 参数
True
```

#### 参数
OneToOneField 接受 ForeignKey 接受的所有额外参数，外加一个额外参数 parent_link。

##### OneToOneField.parent_link
当 True 并用于从另一个*具体的非抽象的模型*继承的模型中时，表示该字段应被用作回到父类的链接，而不是通常通过子类隐含创建的额外 OneToOneField。

## Filed类
[TODO](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#field-api-reference)