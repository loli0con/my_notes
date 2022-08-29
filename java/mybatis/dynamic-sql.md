# 动态SQL
Mybatis框架的动态SQL技术是一种根据特定条件动态拼装SQL语句的功能，它存在的意义是为了解决拼接SQL语句字符串时的痛点问题。提供的元素如下：
* if
* choose (when, otherwise)
* trim (where, set)
* foreach

动态SQL 基于 OGNL 的表达式。

## if
if标签可通过test属性的表达式进行判断，若表达式的结果为true，则标签中的内容会执行;反之标签中 的内容不会执行。

```xml
<!--List<Emp> getEmpListByMoreTJ(Emp emp);-->
<select id="getEmpListByMoreTJ" resultType="Emp">
    SELECT * FROM t_emp WHERE 1=1
    <if test="ename != '' and ename != null">
        AND ename = #{ename}
    </if>
    <if test="age != '' and age != null">
        AND age = #{age}
    </if>
    <if test="sex != '' and sex != null">
        AND sex = #{sex}
    </if>
</select>
```

## where、set、trim

### where
where 元素只会在子元素返回任何内容的情况下才插入 “WHERE” 子句。而且，若子句的开头为 “AND” 或 “OR”，where 元素也会将它们去除。where 元素不能去掉条件最后多余的and。
```xml
<!--List<Emp> getEmpListByMoreTJ2(Emp emp);-->
<select id="getEmpListByMoreTJ2" resultType="Emp">
    SELECT * FROM t_emp
    <where>
        <if test="ename != '' and ename != null">
            ename = #{ename}
        </if>
        <if test="age != '' and age != null">
            AND age = #{age}
        </if>
        <if test="sex != '' and sex != null">
            AND sex = #{sex}
        </if>
    </where>
</select>
```

### set
用于动态更新语句的类似解决方案叫做 set。set 元素可以用于动态包含需要更新的列，忽略其它不更新的列。比如：
```xml
<update id="updateEmpIfNecessary">
    UPDATE t_emp
      <set>
        <if test="ename != null">ename=#{ename},</if>
        <if test="age != null">age=#{age},</if>
        <if test="email != null">email=#{email},</if>
        <if test="sex != null">sex=#{sex}</if>
      </set>
    WHERE id=#{id}
</update>
```
这个例子中，set 元素会动态地在行首插入 SET 关键字，并会删掉额外的逗号（这些逗号是在使用条件语句给列赋值时引入的）。


### trim
如果 where/set 元素与你期望的不太一样，你也可以通过自定义 trim 元素来定制 where/set 元素的功能。
```xml
<!--List<Emp> getEmpListByMoreTJ2(Emp emp);-->
<select id="getEmpListByMoreTJ2" resultType="Emp">
    SELECT * FROM t_emp
    <!-- 注意此例中的空格是必要的(prefixOverrides中的AND和OR后面有一个空格) -->
    <trim prefix="WHERE" prefixOverrides="AND |OR ">
        <if test="ename != '' and ename != null">
            ename = #{ename}
        </if>
        <if test="age != '' and age != null">
            AND age = #{age}
        </if>
        <if test="sex != '' and sex != null">
            AND sex = #{sex}
        </if>
    </trim>
</select>

<update id="updateEmpIfNecessary2">
    UPDATE t_emp
      <!-- 通过使用trim元素来达到同样的效果 -->
      <trim prefix="SET" suffixOverrides=",">
        <if test="ename != null">ename=#{ename},</if>
        <if test="age != null">age=#{age},</if>
        <if test="email != null">email=#{email},</if>
        <if test="sex != null">sex=#{sex}</if>
      </trim>
    WHERE id=#{id}
</update>
```
trim会移除所有 prefixOverrides/suffixOverrides 属性中指定的内容，并且插入 prefix/suffix 属性中指定的内容，分别对应前缀和后缀。


## choose、when、otherwise
MyBatis 提供了 choose 元素，它有点像 Java 中的 switch 语句。
```xml
<!--
    （如果）传入了 ename 就按 ename 查找，
    （如果没有传入ename且）传入了 age 就按 age 查找，
    （如果没有传入ename、age且）传入了 sex 就按 sex 查找，
    （如果没有传入ename、age、email且）传入了 email 就按 email 查找。
-->

<!--List<Emp> getEmpListByChoose(Emp emp);-->
<select id="getEmpListByChoose" resultType="Emp">
    SELECT <include refid="empColumns"></include> FROM t_emp
    <where>
        <choose>
            <when test="ename != '' and ename != null">
                ename = #{ename}
            </when>
            <when test="age != '' and age != null">
                age = #{age}
            </when>
            <when test="sex != '' and sex != null">
                sex = #{sex}
            </when>
            <when test="email != '' and email != null">
                email = #{email}
            </when>
        </choose>
    </where>
</select>
```

## foreach
动态 SQL 的另一个常见使用场景是对集合进行遍历（尤其是在构建 IN 条件语句的时候），foreach 元素的功能非常强大，它允许你指定一个集合，声明可以在元素体内使用的集合项（item）和索引（index）变量。它也允许你指定开头与结尾的字符串以及集合项迭代之间的分隔符，而且这个元素也不会错误地添加多余的分隔符。

你可以将任何可迭代对象（如 List、Set 等）、Map 对象或者数组对象作为集合参数传递给 foreach。当使用可迭代对象或者数组时，index 是当前迭代的序号，item 的值是本次迭代获取到的元素。当使用 Map 对象（或者 Map.Entry 对象的集合）时，index 是键，item 是值。

它的属性: 
* collection:设置要循环的数组或集合
* item:表示集合或数组中的每一个数据
* separator:设置循环体之间的分隔符
* open:设置foreach标签中的内容的开始符
* close:设置foreach标签中的内容的结束符

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
    SELECT *
    FROM post p
    <where>
        <foreach item="au" index="idx" collection="authorList"
            open="p.author in (" separator="," close=")" nullable="true">
            #{au}
        </foreach>
    </where>
</select>
```

## sql
sql片段，可以记录一段公共sql片段，在使用的地方通过include标签进行引入。
```xml
<sql id="empColumns">
    eid,ename,age,sex
</sql>
<select id="getEmpList" resultType="Emp">
    SELECT <include refid="empColumns"></include>
    FROM t_emp
</select>
```

## bind
bind 元素允许你在 OGNL 表达式以外创建一个变量，并将其绑定到当前的上下文。比如：
```xml
<select id="selectBlogsLike" resultType="Blog">
    <bind name="pattern" value="'%' + _parameter.getTitle() + '%'" />
    SELECT *
    FROM blog
    WHERE title LIKE #{pattern}
</select>
```

# OGNL
OGNL是Object-Graph Navigation Language的缩写，它是一种功能强大的表达式语言，通过它简单一致的表达式语法，可以存取对象的任意属性，调用对象的方法，遍历整个对象的结构图，实现字段类型转化等功能。

## 参考
* https://jueee.github.io/2020/08/2020-08-15-Ognl%E8%A1%A8%E8%BE%BE%E5%BC%8F%E7%9A%84%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95
* https://commons.apache.org/proper/commons-ognl/language-guide.html