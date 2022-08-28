# #{}和${}的区别
-   `${}`是 Properties 文件中的变量占位符，它可以用于标签属性值和 sql 内部，属于静态文本替换，比如${driver}会被静态替换为`com.mysql.jdbc. Driver`。
-   `#{}`是 sql 的参数占位符，MyBatis 会将 sql 中的`#{}`替换为? 号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的? 号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`。

# Xml文件中的标签

除了常见的select、insert、update、delete之外，还有很多如<resultMap>、<parameterMap>、<sql>等。

# 通常一个xml映射文件，都会对应一个Dao接口，这个Dao接口的工作原理是什么，Dao接口里的方法参数不同时，能重载吗？
Dao接口中的Mapper接口的全限名就是映射文件中的namespace的值，接口的方法名，就是映射文件中`MappedStatement`的id值，接口方法内的参数，就是传递给sql的参数。 `Mapper` 接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个 `MappedStatement` ，举例： `com.mybatis3.mappers. StudentDao.findStudentById` ，可以唯一找到 namespace 为 `com.mybatis3.mappers. StudentDao` 下面 `id = findStudentById` 的 `MappedStatement` 。在 MyBatis 中，每一个 `<select>` 、 `<insert>` 、 `<update>` 、 `<delete>` 标签，都会被解析为一个 `MappedStatement` 对象。
理论上Dao接口的方法是可以重载的，但是mybatis的xml里面的ID不允许重复。所以Dao接口中可以有多个重载方法，但是多个接口对应的映射必须只有一个。

# MyBatis的分页
1. 使用RowBounds对象进行分页，是针对ResultSet结果执行的内存分页，不是物理分页
2. 可以直接在sql内书写带有物理分页的参数来完成物理分页
3. 使用分页插件，原理是使用Mybatis提供的插件接口，实现自定义插件，在插件的拦截方法内重写sql，添加limit n，m的语句。

# MyBatis的动态sql
MyBatis动态sql可以让我们在xml文件内，以标签的形式编写动态sql，完成逻辑判断和动态拼接sql的功能，mybatis提供了9种动态sql标签`trim|where|set|foreach|if|choose|when|otherwise|bind`。

# MyBatis如何将sql执行结果封装为目标对象并返回

第一种使用`<resultMap>`标签，逐一定义列名和对象属性名之间的映射关系。第二种是使用sql列的别名功能，将列别名书写为对象属性名，mybatis会忽略大小写，智能找到对应的对象属性名。

# MyBatis的多表关联查询
1. 单独发送一个sql去查询关联对象，付给主对象，然后返回主对象。
2. 使用嵌套查询，嵌套查询的含义是使用join查询，一部分是a对象的属性值，另一部分是关联对象b的属性值，好处是只发一个sql查询，就可以把主对象和其关联对象查出来

# MyBatis延迟加载，以及实现原理
仅支持assocation关联对象和collection关联集合对象的延迟加载，启用延迟加载`lazyLoadingEnable=true`,使用 `CGLIB` 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用 `a.getB().getName()` ，拦截器 `invoke()` 方法发现 `a.getB()` 是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 `a.getB().getName()` 方法的调用。这就是延迟加载的基本原理。

# MyBatis的xml映射文件中，不同的文件，其中的id是否可以重复？
如果配置了namespace，则可以重复；如果没有配置，name就不能
原理是namespace+id是作为`Map<String, MappedStatement>`的key使用的，如果没有namespace，则id重复就会导致数据互相覆盖。

# MyBatis的执行器
1. `SimpleExecutor`：每执行一次update或select，就开启一个Statement对象，用完立即关闭
2. `ReuseExecutor`:执行 update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内，供下一次使用。简言之，就是重复使用 Statement 对象。
3. `BatchExecutor`:执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理。与 JDBC 批处理相同。
在MyBatis配置文件中，可以指定默认的执行器类型，也可以手动给`DefaultSqlSessionFactory`的创建sqlsession的方式传递ExecutorType类型参数。

# MyBatis是否可以映射Enum枚举类
MyBatis 可以映射枚举类，不单可以映射枚举类，MyBatis 可以映射任何对象到表的一列上。映射方式为自定义一个 `TypeHandler` ，实现 `TypeHandler` 的 `setParameter()` 和 `getResult()` 接口方法。 `TypeHandler` 有两个作用，一是完成从 javaType 至 jdbcType 的转换，二是完成 jdbcType 至 javaType 的转换，体现为 `setParameter()` 和 `getResult()` 两个方法，分别代表设置 sql 问号占位符参数和获取列查询结果

# MyBatis的xml映射文件和MyBatis内部数据结构之间的映射关系
在 Xml 映射文件中， `<parameterMap>` 标签会被解析为 `ParameterMap` 对象，其每个子元素会被解析为 ParameterMapping 对象。 `<resultMap>` 标签会被解析为 `ResultMap` 对象，其每个子元素会被解析为 `ResultMapping` 对象。每一个 `<select>、<insert>、<update>、<delete>` 标签均会被解析为 `MappedStatement` 对象，标签内的 sql 会被解析为 BoundSql 对象。

