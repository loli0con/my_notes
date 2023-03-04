# BigDecimal类
float、double这两个基本类型的浮点数容易引起精度丢失，为了能精确表示、计算浮点数，Java提供了BigDecimal类。

## 构造器
BigDecimal类提供了大量的构造器用于创建BigDecimal对象，包括把所有的基本数值型变量转换成一个BigDecimal对象，也包括利用数字字符串、数字字符数组来创建BigDecimal对象。

### BigDecimal(double val)
创建BigDecimal对象时，不要直接使用double浮点数作为构造器参数来调用BigDecimal构造器，否则同样会发生精度丢失的问题。

当程序使用new BigDecimal(0.1)来创建一个BigDecimal对象时，它的值并不是0.1，它实际上等于一个近似0.1的数。这是因为0.1无法准确地表示为double浮点数，所以传入BigDecimal构造器的值不会正好等于0.1（虽然表面上等于该值）。如果使用BigDecimal(String val)构造器的结果是可预知的——写入new BigDecimal("0.1")将创建一个BigDecimal，它正好等于预期的0.1。因此通常建议优先使用基于String的构造器。

如果必须使用double浮点数作为BigDecimal构造器的参数时，不要直接将该double浮点数作为构造器参数创建BigDecimal对象，而是应该通过BigDecimal.valueOf(double value)静态方法来创建BigDecimal对象。