# 单元测试

## 定义
单元测试就是针对最小的功能单元编写测试代码。Java程序最小的功能单元是方法，因此，对Java程序进行单元测试就是针对单个Java方法的测试。

## JUnit
JUnit是一个开源的Java语言的单元测试框架，专门针对Java设计，使用最广泛。JUnit是事实上的单元测试的标准框架，任何Java开发者都应当学习并使用JUnit编写单元测试。

## 基本步骤
1. 导入Junit相关依赖库
2. 创建一个测试类，类名为“被测试类的类名”加上Test
3. 在测试类中编写测试方法，测试方法要加上@Test注解
4. 在测试方法内部使用Assertion类提供的断言方法
5. 运行测试类/测试方法，查看测试结果

## Fixture
JUnit提供了编写测试前准备、测试后清理的固定代码，我们称之为Fixture。

### 测试方法的前后
有两个标记——@BeforeEach和@AfterEach，测试类中被它们注解的实例方法，分别会在运行每个@Test方法前后自动运行，该实例方法用于操纵实例成员变量。  
方法推荐命名：setUp 和 tearDown

### 测试类的前后
还有两个标记——@BeforeAll和@AfterAll，测试类中被它们注解的静态方法，分别会在运行所有@Test方法前后自动运行，该静态方法用于操纵静态成员变量。  
方法推荐命名：setUpAll 和 tearDownAll

### 运行顺序
```Java
// XxxTest为测试类
invokeBeforeAll(XxxTest.class); // 调用@BeforeAll方法
for (Method testMethod : findTestMethods(XxxTest.class)) { // 取得所有测试方法
    var test = new XxxTest(); // 创建XxxTest实例
    invokeBeforeEach(test); // 调用@BeforeEach方法
    invokeTestMethod(test, testMethod); //调用测试方法
    invokeAfterEach(test); // 调用@AfterEach方法
}
invokeAfterAll(XxxTest.class); // 调用@AfterAll方法
```
我们可以注意到每次运行一个@Test方法前，JUnit首先创建一个XxxTest实例，因此每个@Test方法内部的成员变量都是独立的，不能也无法把成员变量的状态从一个@Test方法带到另一个@Test方法。

## 异常测试
JUnit提供assertThrows()来期望捕获一个指定的异常。第二个参数Executable封装了我们要执行的会产生异常的代码。
```Java
@Test
void testDivideZero() {
    assertThrows(java.lang.ArithmeticException, new Executable() {
        @Override
        public void execute() throws Throwable {
            0 / 0;
        }
    });
}
```

## 条件测试
我们可以给测试方法加上一个@Disabled标记（不推荐使用该标记），JUnit就会不运行该方法，并在测试结果中通过“Skipped”表示。

类似@Disabled这种注解就称为条件测试，JUnit根据不同的条件注解，决定是否运行当前的@Test方法。

### 常见的条件测试标注 
* @EnableOnOs(*{OS.LINUX,OS.MAC}*)：在Linux和Mac操作系统上执行
* @DisabledOnOs(*OS.WINDOWS*)：不再Windows操作系统上执行
* @DisabledOnJre(*JRE.JAVA_8*)：只能在Java9或更高版本执行
* @EnabledIfSystemProperty(*named="os.arch",matches=".\*64.\*"*)：只能在64位操作系统上执行
* @EnabledIfEnvironmentVariable(*named="DEBUG",matches="true"*)：需要传入环境变量DEBUG=true才能执行

## 参数化测试
如果待测试的输入和输出是一组数据：可以把测试数据组织起来，用不同的测试数据调用相同的测试方法。

JUnit提供了一个@ParameterizedTest注解，用来进行参数化测试。

## 基本步骤
1. 定义测试方法以@ParameterizedTest标注，定义形参以接受参数
2. 使用合适的注解来给测试方法传递实参
   * @ValueSource：值来源，传入一个数组
   * @MethodSource：默认通过一个同名的静态方法来提供测试参数，也可以手动指定一个静态方法
   * @CsvSource：以数组的形式，传入多行逗号分割值（一个元素对应一行）
   * @CsvFileSource：在classpath中加载指定的CSV文件并传入

## 其他注解
* @DisplayName：给测试用例起一个易读的名称，代替方法名本身
* @RepeatedTest：重复运行测试用例


## 后记
### 参考
* https://www.liaoxuefeng.com/wiki/1252599548343744/1255945269146912
* https://www.bilibili.com/video/BV1u4411T78k
* https://www.bilibili.com/video/BV11y4y1H7s

### 更多
* https://www.w3cschool.cn/junit/
* https://wiki.jikexueyuan.com/project/junit/environment-setup.html
* https://www.yiibai.com/junit
