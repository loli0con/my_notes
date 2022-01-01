# Bean Validation
Bean Validation是Java定义的一套基于注解的数据校验规范。

## 参考
1. https://zhuanlan.zhihu.com/p/42818634
2. https://blog.csdn.net/w57685321/article/details/106783433
3. https://segmentfault.com/a/1190000023917625
4. https://yuyang.run/translate/hibernate-validator/7.0.1/index.html

## 用途
"数据校验"是这个比较常见的工作，在日常的开发中贯穿于代码的各个层次，从上层的View层到下底层的数据层，为了保证程序的正确运行以及数据的正确性，开发者通常会在不同层次间做数据校验而且这些校验通常是重复的，为了实现代码的复用性，通常会把校验的逻辑写在被校验对象上。

Bean Validation就是为了解决这样的问题，它定义了一套元数据模型和API对JavaBean实现校验，默认是以注解作为元数据，可以通过XML重写或者拓展元数据，通常来说注解的方式可以实现比较简单逻辑的校验，而复杂校验就需要通过XML来描述。可以说Bean Validation是JavaBean的一个拓展，也就是说它布局于哪一层的代码，不局限于Web应用还是端应用。

## 关系
* [Bean Validation API](https://mvnrepository.com/artifact/javax.validation/validation-api)：Java定义的规范(旧版)
* [Jakarta Bean Validation API](https://mvnrepository.com/artifact/jakarta.validation/jakarta.validation-api)：Java定义的规范(新版)
* [Apache Commons Validator](https://mvnrepository.com/artifact/commons-validator/commons-validator)：Apache提供的实现
* [Hibernate Validator Engine](https://mvnrepository.com/artifact/org.hibernate.validator/hibernate-validator)：Hibernate提供的实现([旧版](https://mvnrepository.com/artifact/org.hibernate/hibernate-validator)/[新版](https://mvnrepository.com/artifact/org.hibernate.validator/hibernate-validator))
* [Spring Boot Starter Validation](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation)：Spring Boot依赖了Hibernate的实现

## 约束
https://yuyang.run/translate/hibernate-validator/7.0.1/index.html#validator-defineconstraints-spec
### 基本约束
下面是 Jakarta Bean Validation API 中指定的所有约束的列表。所有这些约束都适用于字段/属性级别，在 Jakarta Bean Validation规范中没有定义类级别约束。

* @AssertFalse
  * 检查带注解的元素是否为false
  * 支持的数据类型：Boolean，boolean
* @AssertTrue
  * 检查带注解的元素是否为true
  * 支持的数据类型：Boolean，boolean
* @DecimalMax(value=，inclusive=)
  * 当 inclusive = false 时，检查带注解的值是否小于指定的最大值。否则，该值是否小于或等于指定的最大值。参数值是根据 BigDecimal 字符串表示形式的值。
  * 支持的数据类型：BigDecimal， BigInteger， CharSequence， byte， short， int， long 以及各自的基本类型包装器; Hibernate Validator还支持 JSR 354 API 中的 Number 和 javax.money.MonetaryAmount 的实现。
* @DecimalMin(value=，inclusive=)
  * 当 inclusive = false 时，检查带注解的值是否大于指定的最小值。否则，该值是否大于或等于指定的最小值。参数值是根据 BigDecimal 字符串表示形式的值。
  * 支持的数据类型：BigDecimal， BigInteger， CharSequence， byte， short， int， long 以及各自的基本类型包装器; Hibernate Validator还支持 JSR 354 API 中的 Number 和 javax.money.MonetaryAmount 的实现。
* @Digits(integer=，fraction=)
  * 检查被注解的值是否最多有 integer 位整数和 fraction 位小数
  * 支持的数据类型：BigDecimal， BigInteger， CharSequence， byte， short， int， long a以及各自的基本类型包装器; Hibernate Validator还支持 JSR 354 API 中的 Number 和 javax.money.MonetaryAmount 的实现。
* @Email
  * 检查指定的字符序列是否为有效的电子邮件地址。可选参数 regexp 和 flag 允许指定电子邮件必须匹配的附加正则表达式(包括正则表达式标志)。
  * 支持的数据类型：CharSequence
* @Future
  * 检查注解的日期是否在将来
  * 支持的数据类型：java.util.Date， java.util.Calendar， java.time.Instant， java.time.LocalDate， java.time.LocalDateTime， java.time.LocalTime， java.time.MonthDay， java.time.OffsetDateTime， java.time.OffsetTime， java.time.Year， java.time.YearMonth， java.time.ZonedDateTime， java.time.chrono.HijrahDate， java.time.chrono.JapaneseDate， java.time.chrono.MinguoDate， java.time.chrono.ThaiBuddhistDate; Hibernate Validator还支持 Joda Time 中的 ReadablePartial 和 ReadableInstant 实现
* @FutureOrPresent
  * 检查注解的日期是现在或者将来
  * 支持的数据类型：java.util.Date， java.util.Calendar， java.time.Instant， java.time.LocalDate， java.time.LocalDateTime， java.time.LocalTime， java.time.MonthDay， java.time.OffsetDateTime， java.time.OffsetTime， java.time.Year， java.time.YearMonth， java.time.ZonedDateTime， java.time.chrono.HijrahDate， java.time.chrono.JapaneseDate， java.time.chrono.MinguoDate， java.time.chrono.ThaiBuddhistDate; Hibernate Validator还支持 Joda Time 中的 ReadablePartial 和 ReadableInstant 实现
* @Max(value=)
  * 检查注解的值是否小于或等于指定的最大值
  * 支持的数据类型：BigDecimal， BigInteger， byte， short， int， long 以及原始类型的各个包装器; Hibernate Validator还支持 CharSequence 的子类 (字符序列表示的数值被求值)，以及 Number 和 javax.money.MonetaryAmount 的子类。
* @Min(value=)
  * 检查注解的值是否高于或等于指定的最小值
  * 支持的数据类型：BigDecimal， BigInteger， byte， short， int， long 以及原始类型的各个包装器; Hibernate Validator还支持 CharSequence 的子类 (字符序列表示的数值被求值)，以及 Number 和 javax.money.MonetaryAmount 的子类。
* @NotBlank
  * 检查带注解的字符序列是否为空，去除首尾的空格后的长度是否大于0。 与 @NotEmpty 的不同之处在于，这个约束只能应用于字符序列，并且尾随的空格被忽略。
  * 支持的数据类型：CharSequence
* @NotEmpty
  * 检查带注解的元素是否为 null 或 ""
  * 支持的数据类型：CharSequence， Collection， Map and arrays
* @NotNull
  * 检查注解的值是否为 null
  * 支持的数据类型：Any type
* @Negative
  * 检查元素是否严格为负。零值无效。
  * 支持的数据类型：BigDecimal， BigInteger， byte， short， int， long 和各自的基本类型包装器;Hibernate Validator还支持 CharSequence 的子类 (字符序列表示的数值被求值)，以及 Number 和 javax.money.MonetaryAmount 的子类。
* @NegativeOrZero
  * 检查元素是否为负数或零。
  * 支持的数据类型：BigDecimal， BigInteger， byte， short， int， long 和各自的基本类型包装器;Hibernate Validator还支持 CharSequence 的子类 (字符序列表示的数值被求值)，以及 Number 和 javax.money.MonetaryAmount 的子类。
* @Null
  * 检查注解的值是否为 null
  * 支持的数据类型：Any type
* @Past
  * 检查注解的日期是否在过去
  * 支持的数据类型：java.util.Date，java.util.Calendar， java.time.Instant， java.time.LocalDate， java.time.LocalDateTime， java.time.LocalTime， java.time.MonthDay， java.time.OffsetDateTime， java.time.OffsetTime， java.time.Year， java.time.YearMonth， java.time.ZonedDateTime， java.time.chrono.HijrahDate， java.time.chrono.JapaneseDate， java.time.chrono.MinguoDate， java.time.chrono.ThaiBuddhistDate; Hibernate Validator还支持 Joda Time 中的 ReadablePartial 和 ReadableInstant 实现
* @PastOrPresent
  * 检查注解的日期是过去或者现在
  * 支持的数据类型：java.util.Date，java.util.Calendar， java.time.Instant， java.time.LocalDate， java.time.LocalDateTime， java.time.LocalTime， java.time.MonthDay， java.time.OffsetDateTime， java.time.OffsetTime， java.time.Year， java.time.YearMonth， java.time.ZonedDateTime， java.time.chrono.HijrahDate， java.time.chrono.JapaneseDate， java.time.chrono.MinguoDate， java.time.chrono.ThaiBuddhistDate; Hibernate Validator还支持 Joda Time 中的 ReadablePartial 和 ReadableInstant 实现
* @Pattern(regex=， flags=)
  * 考虑给定的标志匹配，检查带注解的字符串是否与正则表达式 regex 匹配
  * 支持的数据类型：CharSequence
* @Positive
  * 检查元素是否为严格正数。零值无效。
  * 支持的数据类型：BigDecimal， BigInteger， byte， short， int， long 和各自的基本类型包装器;Hibernate Validator还支持 CharSequence 的子类 (字符序列表示的数值被求值)，以及 Number 和 javax.money.MonetaryAmount 的子类。
* @PositiveOrZero
  * 检查元素是正数或者零。
  * 支持的数据类型：BigDecimal， BigInteger， byte， short， int， long 和各自的基本类型包装器;Hibernate Validator还支持 CharSequence 的子类 (字符序列表示的数值被求值)，以及 Number 和 javax.money.MonetaryAmount 的子类。
* @Size(min=， max=)
  * 检查带注解的元素的大小是否介于最小和最大(包括)之间
  * 支持的数据类型：CharSequence， Collection， Map and arrays


### 附加约束
除了 Jakarta Bean Validation API 定义的约束之外，Hibernate Validator 还提供了下面列出的几个有用的自定义约束。除了 @ScriptAssert 可以用于类级别的约束，其他的约束只适用于字段/属性级别。

* @CreditCardNumber(ignoreNonDigitCharacters=)
  * 检查带注解的字符序列是否通过 Luhn 校验和测试。注意，此校验旨在检查用户错误，而不是信用卡的有效性！参见 Anatomy of a credit card number. ignoreNonDigitCharacters 允许忽略非数字字符。默认值为 false。
  * 支持的数据类型：CharSequence
* @Currency(value=)
  * 检查带注解的 javax.money.MonetaryAmount 的货币单位是否是指定货币单位的一部分。
  * 支持的数据类型：javax.money.MonetaryAmount JSR 354 API 的实现类
* @DurationMax(days=， hours=， minutes=， seconds=， millis=， nanos=， inclusive=)
  * 注解了 java.time.Duration 元素不大于由注解参数构造的元素。如果将 inclusive 标志设置为 true，则允许相等。
  * 支持的数据类型：java.time.Duration
* @DurationMin(days=， hours=， minutes=， seconds=， millis=， nanos=， inclusive=)
  * 注解了 java.time.Duration 元素不小于由注解参数构造的元素。如果将 inclusive 标志设置为 true，则允许相等。
  * 支持的数据类型：java.time.Duration
* @EAN
  * 检查带注解的字符序列是否为有效的 EAN 条形码。类型决定了条形码的类型。默认值是 EAN-13。
  * 支持的数据类型：CharSequence
* @ISBN
  * 检查带注解的字符序列是否为有效的 ISBN 条形码。类型决定了条形码的类型。默认值是 ISBN-13。
  * 支持的数据类型：CharSequence
* @Length(min=， max=)
  * 校验带注解的字符序列长度是否在 min 和 max 之间
  * 支持的数据类型：CharSequence
* @CodePointLength(min=， max=， normalizationStrategy=)
  * 校验带注解的字符序列的代码点长度是否在最小值和最大值之间。如果设置了 normalizationStrategy ，则校验规范化值。
  * 支持的数据类型：CharSequence
* @LuhnCheck(startIndex= ， endIndex=， checkDigitIndex=， ignoreNonDigitCharacters=)
  * 检查带注解的字符序列中的数字是否通过 Luhn 校验和算法 (参见 Luhn algorithm). startIndex 和 endIndex 只允许在指定的子字符串上运行算法。 checkDigitIndex 允许使用字符序列中的任意数字作为校验数字。如果未指定，则假定检查数字是指定范围的一部分。最后但并非最不重要的是，ignoreNonDigitCharacters 允许忽略非数字字符。
  * 支持的数据类型：CharSequence
* @Mod10Check(multiplier=， weight=， startIndex=， endIndex=， checkDigitIndex=， ignoreNonDigitCharacters=)
  * 检查带注解的字符序列中的数字是否通过了通用的 mod 10校验和算法。 multiplier 决定奇数的乘数(缺省为3) ，加权为偶数(缺省为1)。 startIndex 和 endIndex 只允许在指定的子字符串上运行算法。 checkDigitIndex 允许使用字符序列中的任意数字作为校验数字。如果未指定，则假定检查数字是指定范围的一部分。最后但并非最不重要的是，ignoreNonDigitCharacters 允许忽略非数字字符。
  * 支持的数据类型：CharSequence
* @Mod11Check(threshold=， startIndex=， endIndex=， checkDigitIndex=， ignoreNonDigitCharacters=， treatCheck10As=， treatCheck11As=)
  * 检查带注解的字符序列中的数字是否通过 mod 11校验和算法。 threshold`指定 mod11乘数增长的阈值; 如果没有指定值，乘数将无限增长。 `treatCheck10As 和 treatCheck11As 分别指定当 mod 11校验和等于10或11时使用的校验数字。默认值分别为 x 和0。 startIndex， endIndex checkDigitIndex and ignoreNonDigitCharacters 的语义与 `@Mod10Check`中的相同。
  * 支持的数据类型：CharSequence
* @Normalized(form=)
  * 校验注解的字符序列是否根据给定的 form 进行了规范化。
  * 支持的数据类型：CharSequence
* @Range(min=， max=)
  * 检查注解值是否位于指定的最小值和最大值之间
  * 支持的数据类型：BigDecimal， BigInteger， CharSequence， byte， short， int， long 以及基元类型的相应包装器
* @ScriptAssert(lang=， script=， alias=， reportOn=)
  * 检查是否可以根据带注解的元素成功地计算给定的脚本。为了使用这个约束，JSR 223(“ Java TM 平台的脚本编程”)定义的 Java 脚本 API 的实现必须是类路径的一部分。要求值的表达式可以使用任何脚本或表达式语言编写，在类路径中可以找到与 JSR 223兼容的引擎。即使这是一个类级别的约束，也可以使用 reportOn 属性报告特定属性(而不是整个对象)上的约束冲突。
  * 支持的数据类型：Any type
* @UniqueElements
  * 检查带注解的集合是否只包含唯一元素。相等是使用 equals() 方法确定的。默认消息不包含重复元素的列表，但是可以通过重写消息并使用{ duplicates }消息参数来包含它。重复元素列表也包含在约束违反的动态有效负载中。
  * 支持的数据类型：Collection
* @URL(protocol=， host=， port=， regexp=， flags=)
  * 根据 RFC2396检查带注解的字符序列是否为有效的 URL。如果指定了任何可选参数 protocol， host or port ，则相应的 URL 片段必须匹配指定的值。可选参数 regexp 和 flags 允许指定 URL 必须匹配的附加正则表达式(包括正则表达式标志)。根据默认情况，此约束使用 java. net. URL 构造函数校验给定字符串是否表示有效 URL。还有一个基于正则表达式的版本—— RegexpURLValidator ——可以通过 XML ( 参见 Section 8.2, “通过 constraint-mappings 映射约束”) 或 (参见 Section 12.15.2, “以编程方式添加约束定义”).
  * 支持的数据类型：CharSequence

#### 特殊国家约束
Hibernate Validator 还提供了一些国家特有的约束，例如用于校验社会安全号码。
