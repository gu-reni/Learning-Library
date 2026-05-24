# Mybatis-牛客面经八股

> 来源：牛客网  |  共 9 题

## 3. 在MyBatis中$和#有什么区别
### 

### 一句话总结
 在MyBatis中，#{}会预编译生成PrepareStatement防止SQL注入，参数会被转义；${}直接拼接SQL字符串，存在注入风险。前者适用于参数值替换，后者用于动态表名/列名等需原生SQL的场景。优先使用#{}确保安全性。
 
### 详细解析
 
 在 MyBatis 中，#{}和${}是两种参数替换语法，但它们的底层实现和适用场景有本质区别。以下是核心差异总结： 
 
---
 
#### **1. 处理方式**
 
| 语法 | 处理机制 | 示例（参数name = "Alice"） |
| --- | --- | --- |
| #{} | 预编译占位符（PreparedStatement） | WHERE name = ?→ 参数值"Alice" |
| ${} | 字符串直接替换（字符串拼接） | WHERE name = Alice→ 可能引发 SQL 错误 |
 
---
 
#### **2. SQL 注入风险**
 
| 语法 | 安全性 | 风险场景 |
| --- | --- | --- |
| #{} | 安全（自动转义参数值） | 无 |
| ${} | 不安全（直接拼接原始值） | 若参数来自用户输入，可能被注入攻击。 |
 
---
 
#### **3. 使用场景**
 
| 语法 | 适用场景 | 示例 |
| --- | --- | --- |
| #{} | 动态参数值（如 WHERE 条件、INSERT 值） | WHERE id = #{id} |
| ${} | 动态 SQL 片段（如表名、列名、排序字段） | ORDER BY ${sortField} ${sortOrder} |
 
---
 
#### **4. 参数类型处理**
 
| 语法 | 数据类型处理 | 示例（数值age = 25） |
| --- | --- | --- |
| #{} | 自动匹配类型（如数字不加引号） | WHERE age = 25（无引号） |
| ${} | 直接替换为字符串（可能导致类型错误） | WHERE age = '25'（可能引发类型错误） |
 
---
 
#### **5. 性能**
 
| 语法 | 性能影响 |
| --- | --- |
| #{} | 高（预编译 SQL 可复用） |
| ${} | 低（每次生成新 SQL，需重新解析） |
 
---
 
#### **示例对比场景 1：安全参数传递** 
```xml
<!-- 安全：使用 #{}, 生成预编译 SQL -->
<select id="findUser" resultType="User">
 SELECT * FROM user WHERE name = #{name}
</select>
```
 
 执行时 SQL： 
 
```sql
SELECT * FROM user WHERE name = ? -- 参数值 "Alice" 自动转义
```
 **场景 2：动态排序字段（需谨慎校验）** 
```xml
<!-- 使用 ${} 动态拼接 SQL 片段 -->
<select id="findUsers" resultType="User">
 SELECT * FROM user ORDER BY ${sortField} ${sortOrder}
</select>
```
 
 参数sortField = "age",sortOrder = "DESC"时，生成： 
 
```sql
SELECT * FROM user ORDER BY age DESC
```
 
---
 
#### **总结**
 
| 特性 | #{} | ${} |
| --- | --- | --- |
| 安全性 | 防 SQL 注入 | 需手动校验参数 |
| 参数类型 | 自动类型匹配 | 直接替换为字符串 |
| 适用场景 | 动态值（WHERE/INSERT/UPDATE） | 动态 SQL 片段（表名、排序等） |
| 性能 | 高（预编译复用） | 低（每次生成新 SQL） |

## 6. 介绍一下MyBatis的缓存机制
### 

### 一句话总结
 MyBatis采用两级缓存机制：
 1. **一级缓存**：SqlSession级别，默认开启，同一会话中相同查询直接复用缓存结果，执行更新操作或关闭会话时自动清空。
 2. **二级缓存**：Mapper级别，需手动开启，多个SqlSession共享，通过序列化机制实现跨会话缓存，支持LRU等淘汰策略。
 两者均通过装饰器模式实现，可通过接口扩展集成第三方缓存（如Redis）。
 
### 详细解析
 
 MyBatis 缓存分为**一级缓存**和**二级缓存**，两者在作用范围、生命周期和管理方式上存在显著差异。以下是详细的介绍： 
 
---
 
#### **1. 一级缓存（Local Cache）核心特性作用范围**：SqlSession级别（同一个数据库会话）。 **默认开启**：无需额外配置。 **生命周期**：与SqlSession绑定，会话关闭或执行更新操作（增删改）时自动清空。 **共享性**：仅对当前SqlSession可见，其他会话无法访问。 **工作原理缓存命中**：在同一个SqlSession中，若多次执行**相同的 SQL 和参数**，MyBatis 会直接从缓存中返回结果，避免重复查询数据库。 **缓存失效**： 执行INSERT、UPDATE、DELETE操作。 手动调用sqlSession.clearCache()。 提交事务（sqlSession.commit()）或回滚事务（sqlSession.rollback()）。 **示例** 
```java
SqlSession sqlSession = sqlSessionFactory.openSession();
UserMapper mapper = sqlSession.getMapper(UserMapper.class);

// 第一次查询，访问数据库
User user1 = mapper.selectUserById(1); 

// 第二次查询，命中一级缓存（不访问数据库）
User user2 = mapper.selectUserById(1); 

// 执行更新操作，清空缓存
mapper.updateUser(user1);
sqlSession.commit();

// 第三次查询，缓存已失效，重新访问数据库
User user3 = mapper.selectUserById(1); 

sqlSession.close();
```
 
---
 
#### **2. 二级缓存（Second Level Cache）核心特性作用范围**：Mapper级别（跨SqlSession）。 **手动开启**：需在 XML 或注解中显式配置。 **生命周期**：与应用生命周期一致，直到缓存被主动清除或配置过期。 **共享性**：多个SqlSession共享同一 Mapper 的缓存。 **配置方式全局开启**：在mybatis-config.xml中启用二级缓存： 
```xml
<settings>
 <setting name="cacheEnabled" value="true"/>
</settings>
```
 **Mapper 级配置**：在 Mapper XML 中添加<cache/>标签： 
```xml
<mapper namespace="com.example.UserMapper">
 <cache eviction="LRU" flushInterval="60000" size="1024"/>
</mapper>
```
 **参数说明**： eviction：缓存淘汰策略（默认LRU，可选FIFO、SOFT、WEAK）。 flushInterval：自动刷新间隔（毫秒）。 size：缓存最大对象数量。 **工作原理缓存命中**：多个SqlSession执行相同 SQL 时，若二级缓存存在数据，直接返回结果。 **缓存失效**： 执行INSERT、UPDATE、DELETE操作（同一 Mapper）。 手动调用sqlSession.clearCache()或通过配置自动刷新。 **示例** 
```xml
<!-- UserMapper.xml -->
<mapper namespace="com.example.UserMapper">
 <cache eviction="LRU"/>

 <select id="selectUserById" resultType="User" useCache="true">
 SELECT * FROM user WHERE id = #{id}
 </select>
</mapper>
```
 
---
 
#### **3. 缓存优先级与执行流程**
 
 当同时启用一级和二级缓存时，MyBatis 按以下顺序查询数据： 
 **一级缓存**：优先检查当前SqlSession的缓存。 **二级缓存**：若一级缓存未命中，检查二级缓存。 **数据库查询**：若两级缓存均未命中，执行 SQL 查询，并将结果写入缓存。 
---
 
#### **4. 缓存使用注意事项适用场景一级缓存**：适合单次会话内重复查询（如循环中多次查询同一数据）。 **二级缓存**：适合**读多写少**且数据实时性要求不高的场景（如配置表、静态数据）。 **常见问题脏读**： 若多个SqlSession修改同一数据，二级缓存可能返回过期数据。 **解决方案**：合理设置flushInterval或在更新操作后手动清除缓存。 **序列化问题**： 二级缓存默认将对象序列化存储，实体类需实现Serializable接口。 **分布式环境**： 默认二级缓存是单机缓存，分布式系统中需集成 Redis、Ehcache 等分布式缓存框架。 
---
 
#### **5. 扩展：自定义缓存**
 
 MyBatis 支持集成第三方缓存库（如 Redis、Ehcache）替换默认的 PerpetualCache。 
 **步骤**（以 Redis 为例）： 添加 Redis 依赖： 
```xml
<dependency>
 <groupId>org.mybatis.caches</groupId>
 <artifactId>mybatis-redis</artifactId>
 <version>1.0.0-beta2</version>
</dependency>
```
 配置 Mapper 使用 Redis 缓存： 
```xml
<mapper namespace="com.example.UserMapper">
 <cache type="org.mybatis.caches.redis.RedisCache"/>
</mapper>
```
 
---
 
#### **6. 总结**
 
| 缓存类型 | 作用范围 | 配置方式 | 适用场景 | 注意事项 |
| --- | --- | --- | --- | --- |
| 一级缓存 | SqlSession | 默认开启 | 会话内重复查询 | 更新操作自动失效 |
| 二级缓存 | Mapper | 需显式配置<cache/> | 跨会话共享的静态数据 | 避免脏读，需序列化支持 |
| 自定义缓存 | 全局/分布式 | 集成第三方库 | 高并发、分布式系统 | 需解决数据一致性问题 |

## 2. MyBatis里如何实现一对多关联查询？
### 

### 一句话总结

 在MyBatis中实现一对多关联查询可通过两种方式：1）在XML映射文件中使用`<collection>`标签，通过嵌套结果集（联表查询）或嵌套查询（分次查询）关联子表数据；2）使用注解方式通过`@Results`和`@Result`注解，结合`@Many`指定子查询方法。需在实体类中定义包含多个子对象的集合属性，并通过column参数传递关联字段。

### 详细解析

 在 MyBatis 中实现一对多关联查询，主要通过 嵌套查询（分步查询） 和 嵌套结果（联表查询） 两种方式实现。以下是具体实现方法和示例：

---

 一、嵌套查询（分步查询）

原理：先查询主实体，再根据主实体的关联字段查询子实体集合。

适用场景：子数据量较小或需要延迟加载的场景。

 实现步骤：

 - 定义实体类 主实体类中需包含子实体的集合属性： ```cpp public class User { private Long id; private String name; private List orders; // 子实体集合 // getters/setters } ```

 - 编写 Mapper XML • 主查询：查询主实体并触发子查询。 • 子查询：根据主实体 ID 查询子实体列表。 

```c
<!-- 主查询 -->
<select id="selectUserWithOrders" resultMap="userResultMap">
 SELECT * FROM user WHERE id = #{id}
</select>

<!-- 子查询 -->
<select id="selectOrdersByUserId" resultType="Order">
 SELECT * FROM order WHERE user_id = #{userId}
</select>

<!-- 结果映射 -->
<resultMap id="userResultMap" type="User">
 <id column="id" property="id"/>
 <result column="name" property="name"/>
 <!-- 嵌套查询配置 -->
 <collection property="orders" ofType="Order" 
 select="selectOrdersByUserId" 
 column="id"/> <!-- 传递主实体ID到子查询 -->
</resultMap>
```

 - 调用查询方法 ```cpp User user = sqlSession.selectOne("selectUserWithOrders", 1); List orders = user.getOrders(); // 自动触发子查询 ```

 优点：逻辑清晰，适合分步加载。

缺点：可能产生 N+1 查询问题（主查询 1 次 + 子查询 N 次）。

---

 二、嵌套结果（联表查询）

原理：通过单次 SQL JOIN 查询获取所有数据，直接映射到主实体和子实体集合。

适用场景：需要一次性加载关联数据，避免多次查询。

 实现步骤：

 - 定义实体类 与嵌套查询相同，主实体需包含子实体集合。

 - 编写 Mapper XML • 单次 JOIN 查询：关联主表和子表。 • 结果映射：通过collection标签映射子实体集合。 

```c
<select id="selectUserWithOrdersJoin" resultMap="userJoinResultMap">
 SELECT 
 u.id as user_id, u.name as user_name,
 o.id as order_id, o.order_no as order_no
 FROM user u
 LEFT JOIN order o ON u.id = o.user_id
 WHERE u.id = #{id}
</select>

<resultMap id="userJoinResultMap" type="User">
 <id column="user_id" property="id"/>
 <result column="user_name" property="name"/>
 <!-- 嵌套结果映射 -->
 <collection property="orders" ofType="Order">
 <id column="order_id" property="id"/>
 <result column="order_no" property="orderNo"/>
 </collection>
</resultMap>
```

 - 调用查询方法 ```cpp User user = sqlSession.selectOne("selectUserWithOrdersJoin", 1); List orders = user.getOrders(); // 直接获取关联数据 ```

 优点：减少数据库查询次数，性能更高。

缺点：SQL 复杂度较高，需手动处理字段别名。

---

 三、注解方式实现（MyBatis 注解驱动）

原理：通过@Results和@Many注解配置关联关系。

 示例代码：

```cpp
public interface UserMapper {
 @Select("SELECT * FROM user WHERE id = #{id}")
 @Results({
 @Result(property = "id", column = "id"),
 @Result(property = "name", column = "name"),
 // 配置一对多关联
 @Result(property = "orders", 
 column = "id",
 many = @Many(select = "selectOrdersByUserId"))
 })
 User selectUserWithOrders(@Param("id") Long id);
 @Select("SELECT * FROM order WHERE user_id = #{userId}")
 List selectOrdersByUserId(@Param("userId") Long userId);
}
```

 优点：代码简洁，适合注解驱动开发。

缺点：复杂关联时配置较繁琐。

---

 四、对比总结

| 方式 | 嵌套查询 | 嵌套结果 | 注解方式 |
| --- | --- | --- | --- |
| 查询次数 | 多次（1+N） | 单次 | 同嵌套查询 |
| 性能 | 较低（依赖子查询） | 较高 | 中等 |
| 适用场景 | 子数据量小、延迟加载 | 需一次性加载关联数据 | 简单关联逻辑 |
| 配置复杂度 | 中等 | 较高（需处理字段映射） | 低 |

## 4. Mybatis如何防止sql注入。
### 

### 一句话总结

 Mybatis通过预编译机制和参数绑定防止SQL注入。使用#{}占位符时，参数会被转义并作为预编译参数传递，避免拼接SQL语句。动态SQL标签（如<if>）会自动处理参数安全，框架内部采用PreparedStatement确保参数与指令分离。避免使用${}进行字符串拼接可降低注入风险。

### 详细解析

 在 MyBatis 中，防止 SQL 注入的核心是正确使用参数占位符（#{}）和避免直接拼接未经验证的用户输入。以下是具体实现方法和最佳实践：

---

#### **1. 优先使用#{}（预编译占位符）原理**：

#{}会被 MyBatis 转换为PreparedStatement的参数占位符（?），底层通过 JDBC 的预编译机制对参数值进行类型检查和转义，确保输入内容被当作数据而非 SQL 代码解析。

 **示例**：

```xml
<!-- 安全：参数值会被预编译处理 -->
<select id="findUser" resultType="User">
 SELECT * FROM user WHERE name = #{name}
</select>
```

 执行时生成的 SQL：

```sql
SELECT * FROM user WHERE name = ? -- 参数值会被安全转义
```

---

#### **2. 严格限制${}（字符串替换）的使用风险**：

${}会直接将参数值以字符串形式拼接到 SQL 中，若参数来自用户输入，可能导致 SQL 注入。

 **安全使用场景**：

 - 动态表名、列名（需校验合法性）。

 - 动态排序字段（如ORDER BY ${sortField}，需校验字段名）。

 **示例**：

```xml
<!-- 动态表名（需校验 tableName 合法性） -->
<select id="queryData" resultType="map">
 SELECT * FROM ${tableName} WHERE id = #{id}
</select>
```

 **校验方法**：

 - **白名单校验**：确保${}的参数值只能是预定义的合法值。 ```java // 示例：校验表名是否合法 public void setTableName(String tableName) { if (!Arrays.asList("user", "order").contains(tableName)) { throw new IllegalArgumentException("Invalid table name"); } this.tableName = tableName; } ```

---

#### **3. 避免在 SQL 中直接拼接用户输入错误示例**：

```xml
<!-- 危险：直接拼接用户输入 -->
<select id="findUser" resultType="User">
 SELECT * FROM user WHERE name = '${name}'
</select>
```

 若用户输入name = "admin' OR '1'='1"，最终 SQL 会变为：

```sql
SELECT * FROM user WHERE name = 'admin' OR '1'='1' -- 返回所有数据
```

 **修复方法**：

 改用#{}：

```xml
<select id="findUser" resultType="User">
 SELECT * FROM user WHERE name = #{name}
</select>
```

---

#### **4. 使用 MyBatis 的动态 SQL 标签**

 通过<if>、<where>、<choose>等标签安全拼接条件，避免手动拼接 SQL。

 **示例**：

```xml
<select id="findUser" resultType="User">
 SELECT * FROM user
 <where>
 <if test="name != null">
 AND name = #{name}
 </if>
 <if test="age != null">
 AND age = #{age}
 </if>
 </where>
</select>
```

---

#### **5. 全局配置防御策略**

 在mybatis-config.xml中启用安全配置：

```xml
<settings>
 <!-- 开启驼峰命名映射（避免手动写列名时出错） -->
 <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

---

#### **6. 总结**

| 防御手段 | 说明 |
| --- | --- |
| 优先使用#{} | 利用预编译机制对参数值转义，防止 SQL 注入。 |
| 严格校验${}输入 | 仅在必要时使用${}，并通过白名单确保参数值的合法性。 |
| 避免手动拼接 SQL | 使用 MyBatis 动态 SQL 标签（如<if>、<where>）安全拼接条件。 |
| 代码审查与工具 | 通过规范和自动化工具避免低级错误。 |

---

#### **7. SQL 注入测试案例攻击输入**：

```java
String name = "admin' OR '1'='1";
```

 **安全代码**：

```xml
<select id="findUser" resultType="User">
 SELECT * FROM user WHERE name = #{name}
</select>
```

 **最终 SQL**：

```sql
SELECT * FROM user WHERE name = 'admin'' OR ''1''=''1' -- 参数被转义，视为普通字符串
```

 通过以上方法，MyBatis 能有效防御绝大多数 SQL 注入攻击。

## 5. 说一下 MyBatis 和 MyBatis Plus 的区别。
### 

### 一句话总结

 MyBatis是基础ORM框架，需手动编写SQL与配置；MyBatis Plus是其增强工具，提供通用Mapper、自动生成SQL、内置分页等功能，简化CRUD操作开发。后者基于前者扩展，减少代码量但保留MyBatis灵活性。

### 详细解析

 MyBatis 和 MyBatis-Plus 是 Java 生态中常用的持久层框架。以下是它们的核心区别与联系：

---

#### **一、核心区别**

| 特性 | MyBatis | MyBatis-Plus |
| --- | --- | --- |
| SQL 编写 | 需手动编写所有 SQL（XML 或注解） | 内置通用 CRUD，单表操作无需写 SQL |
| 条件构造器 | 不支持，需手写动态 SQL | 提供QueryWrapper、LambdaQueryWrapper等，支持链式调用 |
| 代码生成 | 无内置支持 | 内置代码生成器，可快速生成 Model、Mapper 等层代码 |
| Lambda 支持 | 不支持 | 支持 Lambda 表达式，避免硬编码字段名 |
| 主键策略 | 需手动配置 | 支持 4 种主键策略（含分布式 ID 生成器） |
| 分页插件 | 需手动实现或集成第三方插件 | 内置物理分页插件，简化分页逻辑 |
| SQL 注入防护 | 依赖开发者使用#{}占位符 | 内置 SQL 注入剥离器，自动过滤危险字符 |
| 实体映射注解 | 需手动配置 XML 映射 | 支持@TableName、@TableId等注解，简化表与字段映射 |
| 全局拦截与插件 | 需自定义拦截器 | 内置全局拦截插件（如防全表删除/更新） |
| 性能优化 | 依赖手动优化 SQL | 支持批量操作、二级缓存等优化 |

---

#### **二、主要联系**

 - **基础兼容性** MyBatis-Plus 是 MyBatis 的增强工具，完全兼容原生 MyBatis 的所有特性，引入 MyBatis-Plus 不会影响原有 MyBatis 项目的运行。

 - **依赖关系** MyBatis-Plus 基于 MyBatis 开发，依赖MyBatis和MyBatis-Spring，可无缝整合到 Spring Boot 等框架中。

 - **ORM 特性** 两者均属于半自动化 ORM 框架，需通过配置实现对象与数据库的映射，但 MyBatis-Plus 进一步简化了映射配置。

 - **动态 SQL 支持** MyBatis-Plus 保留了 MyBatis 的动态 SQL 标签（如<if>、<foreach>），并在此基础上扩展了更便捷的条件构造方式。

---

#### **三、适用场景对比**

| 场景 | 推荐框架 | 理由 |
| --- | --- | --- |
| 复杂 SQL 或存储过程 | MyBatis | 灵活控制 SQL，适合对 SQL 性能要求高的场景 |
| 快速开发单表 CRUD | MyBatis-Plus | 内置通用方法，减少重复代码 |
| 微服务或分布式项目 | MyBatis-Plus | 支持分布式 ID、批量操作等优化 |
| 需要高度定制化 | MyBatis | 原生 MyBatis 更灵活，无额外约束 |

####

## 7. MyBatis如何实现分页查询？
### 

### 一句话总结

 MyBatis实现分页主要有三种方式：1. 使用RowBounds对象进行内存分页（适用于小数据量）；2. 在SQL中直接编写LIMIT/OFFSET语句（如MySQL）；3. 通过分页插件（如PageHelper）自动拦截SQL并添加分页逻辑。物理分页推荐使用插件或数据库方言实现，避免内存溢出风险。

### 详细解析

 在 MyBatis 中实现分页查询，主要有以下几种方式，各有优缺点和适用场景：

---

#### **1. 原生 MyBatis 分页（手动参数传递）**

 通过 SQL 的LIMIT和OFFSET（或数据库方言如 Oracle 的ROWNUM）手动分页。

 **实现步骤**：

 - **实体类定义**： ```java public class PageParam { private int pageNum; // 当前页码 private int pageSize; // 每页数量 // Getters & Setters } ```

 - **Mapper XML**： ```xml <select id="selectUsersByPage" resultType="User"> SELECT * FROM user LIMIT #{pageSize} OFFSET #{offset} </select> ```

 - **Mapper 接口**： ```java public interface UserMapper { List<User> selectUsersByPage(@Param("pageSize") int pageSize, @Param("offset") int offset); } ```

 - **调用代码**： ```java int pageNum = 2; // 第2页 int pageSize = 10; int offset = (pageNum - 1) * pageSize; // 计算偏移量 List<User> users = userMapper.selectUsersByPage(pageSize, offset); ```

 **特点**：

 - **优点**：简单直接，无需第三方依赖。

 - **缺点**：需手动计算偏移量，不同数据库需调整 SQL 方言（如 Oracle 用ROWNUM）。

---

#### **2. 使用RowBounds（逻辑分页）**

 通过 MyBatis 内置的RowBounds对象实现逻辑分页（内存分页）。

 **实现步骤**：

 - **Mapper 接口**： ```java List<User> selectAllUsers(RowBounds rowBounds); ```

 - **XML SQL**： ```xml <select id="selectAllUsers" resultType="User"> SELECT * FROM user </select> ```

 - **调用代码**： ```java int pageNum = 2; int pageSize = 10; RowBounds rowBounds = new RowBounds((pageNum - 1) * pageSize, pageSize); List<User> users = userMapper.selectAllUsers(rowBounds); ```

 **特点**：

 - **优点**：代码简单，统一分页方式。

 - **缺点**：本质是内存分页（先查询全部数据，再截取片段），大数据量时性能差。

---

#### **3. 使用分页插件（推荐：PageHelper）**

 通过第三方插件（如 PageHelper）实现物理分页，自动改写 SQL。

 **实现步骤**：

 - **添加依赖**： ```xml <dependency> <groupId>com.github.pagehelper</groupId> <artifactId>pagehelper</artifactId> <version>5.3.2</version> </dependency> ```

 - **配置拦截器**（在mybatis-config.xml中）： ```xml <plugins> <plugin interceptor="com.github.pagehelper.PageInterceptor"> <property name="helperDialect" value="mysql"/> <!-- 指定数据库方言 --> </plugin> </plugins> ```

 - **Mapper XML**： ```xml <select id="selectAllUsers" resultType="User"> SELECT * FROM user </select> ```

 - **调用代码**： ```java int pageNum = 2; int pageSize = 10; PageHelper.startPage(pageNum, pageSize); // 开启分页 List<User> users = userMapper.selectAllUsers(); PageInfo<User> pageInfo = new PageInfo<>(users); ```

 **结果说明**：

 - PageInfo对象包含分页信息： ```java pageInfo.getTotal(); // 总记录数 pageInfo.getPages(); // 总页数 pageInfo.getPageNum(); // 当前页码 pageInfo.getList(); // 当前页数据 ```

 **特点**：

 - **优点**：自动生成分页 SQL，支持多种数据库，性能高。

 - **缺点**：需引入第三方库。

---

#### **4. MyBatis-Plus 分页（集成方案）**

 若项目使用 **MyBatis-Plus**，可直接使用其内置分页插件。

 **实现步骤**：

 - **添加依赖**： ```xml <dependency> <groupId>com.baomidou</groupId> <artifactId>mybatis-plus-boot-starter</artifactId> <version>3.5.3.1</version> </dependency> ```

 - **配置分页插件**（Spring Boot）： ```java @Configuration public class MyBatisPlusConfig { @Bean public MybatisPlusInterceptor mybatisPlusInterceptor() { MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor(); interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL)); return interceptor; } } ```

 - **Mapper 接口**： ```java public interface UserMapper extends BaseMapper<User> { // 无需定义分页方法 } ```

 - **调用代码**： ```java int pageNum = 2; int pageSize = 10; Page<User> page = new Page<>(pageNum, pageSize); IPage<User> userPage = userMapper.selectPage(page, null); List<User> users = userPage.getRecords(); long total = userPage.getTotal(); ```

 **特点**：

 - **优点**：与 MyBatis-Plus 深度集成，支持 Lambda 表达式。

 - **缺点**：需引入 MyBatis-Plus 依赖。

---

#### **5. 分页方式对比**

| 方式 | 性能 | 复杂度 | 适用场景 | 依赖 |
| --- | --- | --- | --- | --- |
| 原生手动分页 | 高 | 高 | 简单 SQL 或需精确控制 SQL | 无 |
| RowBounds | 低 | 低 | 小数据量 | 无 |
| PageHelper | 高 | 中 | 复杂 SQL，需物理分页 | PageHelper |
| MyBatis-Plus 分页 | 高 | 低 | 已使用 MyBatis-Plus 的项目 | MyBatis-Plus |

####

## 8. 讲一下Mybatis中动态sql的作用。
### 

### 一句话总结

 Mybatis动态SQL用于根据不同条件灵活拼接SQL语句，避免手动拼接的繁琐和错误。通过标签（如`<if>`、`<choose>`、`<foreach>`）实现逻辑判断、循环遍历等操作，简化复杂查询的编写。适用于多条件查询、批量操作等场景，提升代码可读性和维护性，同时减少重复代码。

### 详细解析

 MyBatis 的动态 SQL 是其核心特性之一，通过灵活的标签和表达式，允许开发者根据运行时条件动态生成 SQL 语句。以下是其核心作用及具体应用场景：

---

#### **一、动态 SQL 的核心作用**

 - **灵活的条件查询** • 按需拼接 SQL：根据参数是否为空动态添加WHERE条件，避免冗余的AND或OR。 ```xml <select id="findUser" resultType="User"> SELECT * FROM user <where> <if test="name != null">AND name = #{name}</if> <if test="age != null">AND age = #{age}</if> </where> </select> ``` 效果：若name和age均为空，则生成SELECT * FROM user；若仅name有值，则生成WHERE name = ?。

 - **减少重复代码** • 避免多条件分支硬编码：通过<choose>/<when>/<otherwise>实现多条件互斥选择，替代多个if-else判断。 ```xml <select id="findUserByType"> SELECT * FROM user WHERE role = <choose> <when test="role == 'admin'">'ADMIN'</when> <when test="role == 'user'">'USER'</when> <otherwise>'GUEST'</otherwise> </choose> </select> ``` 优势：代码简洁，逻辑清晰，维护成本低。

 - **动态批量操作** • 集合遍历生成 SQL：通过<foreach>标签遍历集合，生成IN语句或批量插入。 ```xml <select id="selectByIds"> SELECT * FROM user WHERE id IN <foreach collection="ids" item="id" open="(" separator="," close=")"> #{id} </foreach> </select> ``` 应用场景：批量查询、批量更新或删除。

 - **防止 SQL 注入** • 参数化绑定：使用#{}占位符自动转义特殊字符，避免拼接字符串导致的注入风险。 ```xml <bind name="pattern" value="'%' + name + '%'" /> SELECT * FROM user WHERE name LIKE #{pattern} ``` 安全性：即使参数含恶意字符（如' OR 1=1 --），也会被转义为普通字符串。

 - **动态更新与字段控制** • 按需更新字段：通过<set>标签动态生成UPDATE语句，避免冗余字段和末尾逗号。 ```xml <update id="updateUser"> UPDATE user <set> <if test="name != null">name = #{name},</if> <if test="age != null">age = #{age},</if> </set> WHERE id = #{id} </update> ``` 效果：仅更新非空字段，减少数据库写入量。

---

#### **二、典型应用场景**

| 场景 | 动态 SQL 方案 | 效果 |
| --- | --- | --- |
| 多条件查询 | <if>+<where> | 按需生成WHERE子句，避免冗余条件 |
| 分页查询 | <foreach>+ 分页插件 | 动态生成LIMIT和OFFSET |
| 批量插入/更新 | <foreach>遍历集合 | 单条 SQL 处理多条数据，减少数据库交互 |
| 多表关联查询 | <choose>动态选择关联表 | 根据业务需求切换关联逻辑 |
| 动态排序 | <if>控制ORDER BY字段和方向 | 支持用户自定义排序规则 |

---

#### **三、动态 SQL 的执行原理**

 - OGNL 表达式解析：MyBatis 使用 OGNL（Object-Graph Navigation Language）解析动态标签中的条件表达式（如#{name}），将参数映射到 SQL 中。

 - SqlNode 组合：动态标签被解析为SqlNode对象（如IfSqlNode、WhereSqlNode），通过组合模式构建完整的SqlSource，最终生成可执行的 SQL。

 - 预编译与参数绑定：生成的 SQL 仍使用PreparedStatement进行参数绑定，确保安全性和性能。

#### **四、实际场景示例场景：多条件搜索用户** 

```xml
<select id="searchUsers" resultType="User">
 SELECT * FROM user
 <where>
 <if test="name != null">
 AND name LIKE CONCAT('%', #{name}, '%')
 </if>
 <if test="minAge != null">
 AND age >= #{minAge}
 </if>
 <if test="roles != null and roles.size() > 0">
 AND role IN
 <foreach collection="roles" item="role" open="(" close=")" separator=",">
 #{role}
 </foreach>
 </if>
 </where>
 ORDER BY create_time DESC
</select>
```

## 9. 讲一下Mybatis的插件原理。
### 

### 一句话总结

 Mybatis插件基于拦截器机制，通过动态代理实现。用户实现Interceptor接口并指定拦截的方法，Mybatis在启动时通过责任链模式为Executor、StatementHandler等核心组件生成代理对象。执行目标方法时，插件按配置顺序逐层代理，最终调用原始方法。该机制可用于SQL改写、日志增强等功能扩展。

### 详细解析

 MyBatis 的插件机制是其核心扩展能力之一，通过动态代理和拦截器模型，允许开发者在 SQL 执行的关键节点插入自定义逻辑（如日志记录、性能监控、SQL 改写等）。以下是其核心原理和实现细节的深度解析：

---

 **一、插件运行原理的核心架构** 

 - **拦截器接口（Interceptor）** • 定义：插件需实现Interceptor接口，包含三个核心方法： ◦intercept(Invocation invocation)：拦截目标方法，执行自定义逻辑。 ◦plugin(Object target)：生成代理对象，包装目标对象。 ◦setProperties(Properties properties)：设置插件属性（通过 XML 配置传递参数）。 • 作用：拦截器是插件的核心逻辑载体，决定何时触发拦截及如何处理。

 - **动态代理机制** • 代理对象生成：MyBatis 通过 Java 动态代理（Proxy.newProxyInstance）为被拦截的核心组件（如Executor、StatementHandler）生成代理类。 • 方法拦截：当调用代理对象的方法时，会触发InvocationHandler的invoke方法，进而执行插件的intercept方法。

 - **插件链（Interceptor Chain）** • 多插件协作：多个插件可按配置顺序拦截同一方法，形成链式调用。每个插件的intercept方法依次执行，最终调用invocation.proceed()传递到下一个插件或原始方法。

---

 **二、插件执行流程详解** 

 - **插件注册** • 配置方式：在mybatis-config.xml中通过<plugins>标签注册插件，或使用@Intercepts注解声明拦截目标。 • 示例配置： ```xml <plugins> <plugin interceptor="com.example.LoggingInterceptor"> <property name="logLevel" value="DEBUG"/> </plugin> </plugins> ```

 - **目标对象包装** • 代理生成时机：MyBatis 初始化时扫描所有插件，根据@Signature注解匹配目标类和方法，生成代理对象。 • 核心组件拦截：插件可作用于以下四个核心接口： ◦Executor：SQL 执行器（如update、query方法）。 ◦StatementHandler：SQL 语句处理器（如预编译、参数绑定）。 ◦ParameterHandler：参数处理器（参数设置）。 ◦ResultSetHandler：结果集处理器（结果映射）。

 - **方法拦截与逻辑增强** • 拦截点选择：通过@Signature注解指定拦截目标： ```java @Intercepts({ @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class}) }) ``` • 逻辑插入：在intercept方法中，可通过invocation.getArgs()获取方法参数，通过invocation.proceed()调用原始方法，实现日志记录、SQL 改写等操作。

---

 **三、插件开发示例** 

 - **日志记录插件** ```java @Intercepts({ @Signature(type = Executor.class, method = "update", args = {MappedStatement.class, Object.class}) }) public class SqlLogInterceptor implements Interceptor { @Override public Object intercept(Invocation invocation) throws Throwable { MappedStatement ms = (MappedStatement) invocation.getArgs()[0]; Object parameter = invocation.getArgs()[1]; String sql = ms.getBoundSql(parameter).getSql(); System.out.println("Executing SQL: " + sql); long start = System.currentTimeMillis(); Object result = invocation.proceed(); System.out.println("Execution time: " + (System.currentTimeMillis() - start) + "ms"); return result; } @Override public Object plugin(Object target) { return Plugin.wrap(target, this); } } ```

 - **分页插件（简化版）** ```java @Intercepts({ @Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class, Integer.class}) }) public class PaginationInterceptor implements Interceptor { @Override public Object intercept(Invocation invocation) throws Throwable { StatementHandler handler = (StatementHandler) invocation.getTarget(); MappedStatement ms = handler.getMappedStatement(); Object parameter = handler.getParameterHandler().getParameterObject(); // 动态改写 SQL 添加分页逻辑（如 LIMIT/OFFSET） BoundSql boundSql = handler.getBoundSql(); String originalSql = boundSql.getSql(); String pageSql = originalSql + " LIMIT 0, 10"; // 示例分页 // 通过反射修改 BoundSql 中的 SQL Field field = boundSql.getClass().getDeclaredField("sql"); field.setAccessible(true); field.set(boundSql, pageSql); return invocation.proceed(); } } ```

---

 **四、插件应用场景** 

| 场景 | 实现方式 | 典型插件 |
| --- | --- | --- |
| SQL 日志记录 | 拦截Executor.update和query方法 | p6spy、MyBatis-Log4j |
| 分页查询 | 拦截StatementHandler.prepare方法 | PageHelper |
| 性能监控 | 统计 SQL 执行时间 | MyBatis-Performance |
| 数据加密/解密 | 拦截ParameterHandler.setParameters | Jasypt-MyBatis |
| 自动事务管理 | 拦截Executor.commit/rollback方法 | 自定义事务插件 |

## 1. 讲一下Mybatis的底层原理。
### 

### 一句话总结

 Mybatis底层基于JDBC封装，核心流程为：通过配置文件或注解建立SQL与对象映射，运行时动态生成SQL（解析#{}/${}参数、动态标签），利用接口代理机制将方法调用转化为JDBC操作（Executor执行SQL），通过ResultSetHandler将结果集反射映射为Java对象，并支持一级/二级缓存提升性能，结合插件机制扩展功能（如分页）。

### 详细解析

 MyBatis 作为一款流行的持久层框架，其底层原理围绕 **JDBC 封装**、**动态代理**、**反射机制**和**组件协作**展开。以下是其核心原理的详细解析：

---

#### 一、核心组件与协作机制

 MyBatis 通过多个核心组件实现 SQL 的解析、执行与结果映射，各组件职责如下：

 - **SqlSessionFactoryBuilder作用**：解析配置文件（mybatis-config.xml）和映射文件（Mapper.xml），构建SqlSessionFactory实例。

 - **流程**：加载全局配置（数据源、事务管理器、插件等）和映射文件中的 SQL 定义，生成Configuration对象存储所有配置信息。

 - **SqlSessionFactory作用**：创建SqlSession实例，管理数据库连接池和全局配置。

 - **线程安全**：全局唯一，通常通过单例模式管理。

 - **SqlSession作用**：与数据库交互的会话接口，提供 CRUD 方法（如selectOne()、insert()）。

 - **生命周期**：每次数据库操作需创建新的SqlSession，操作完成后关闭以释放资源。

 - **Executor作用**：执行 SQL 的核心组件，负责 SQL 预编译、参数绑定、结果处理等。

 - **类型**： SimpleExecutor：默认执行器，每次执行 SQL 开启新连接。

 - ReuseExecutor：重用预编译的Statement。

 - BatchExecutor：批量执行 SQL，提升性能。

 - **MappedStatement作用**：封装映射文件中的 SQL 语句、参数映射规则（ParameterMap）和结果映射规则（ResultMap）。

 - **存储位置**：Configuration对象维护所有MappedStatement的缓存。

 - **MapperProxy（动态代理）作用**：通过 JDK 动态代理为 Mapper 接口生成代理对象，将接口方法调用转为SqlSession的 SQL 执行。

 - **流程**：调用getMapper()时，根据接口方法名查找对应的MappedStatement，并通过Executor执行 SQL。

---

#### 二、执行流程详解

 MyBatis 的 SQL 执行流程分为以下步骤：

 - **配置解析与初始化** 解析mybatis-config.xml和Mapper.xml，加载数据源、插件、类型处理器等配置到Configuration对象。

 - **创建 SqlSession** 通过SqlSessionFactory.openSession()创建SqlSession，内部通过Executor管理 JDBC 连接和事务。

 - **Mapper 接口动态代理** 调用sqlSession.getMapper(UserMapper.class)生成代理对象MapperProxy，拦截接口方法调用。

 - **SQL 解析与执行参数处理**：根据方法参数生成BoundSql，替换 SQL 中的#{}占位符为预编译参数。

 - **SQL 执行**：通过PreparedStatement执行 SQL，利用 JDBC 返回ResultSet。

 - **结果映射反射机制**：根据ResultMap定义，通过反射将ResultSet的列值映射到 Java 对象的属性。

 - **延迟加载**：对关联对象使用动态代理，在首次访问时触发额外查询。

 - **事务与资源释放** 通过sqlSession.commit()或rollback()管理事务，最终关闭SqlSession释放连接。

---

#### 三、关键机制解析

 - **动态 SQL实现方式**：通过<if>、<foreach>等标签动态拼接 SQL，底层使用 OGNL 表达式解析条件。

 - **示例**：条件查询时，根据参数动态生成WHERE子句。

 - **缓存机制一级缓存**：SqlSession级别，默认开启，同一会话内相同 SQL 直接返回缓存结果。

 - **二级缓存**：Mapper级别，需手动开启，跨会话共享数据（需实体类实现Serializable）。

 - **插件机制（Interceptor）作用**：拦截Executor、StatementHandler等组件的核心方法，实现日志、分页、加密等扩展功能。

 - **实现**：通过动态代理和责任链模式，在方法执行前后插入自定义逻辑。

 - **事务管理JDBC 事务**：默认使用Connection的提交/回滚机制。

 - **集成 Spring**：可通过DataSourceTransactionManager实现声明式事务。

---

#### 四、与 JDBC 的对比

| 特性 | MyBatis | JDBC |
| --- | --- | --- |
| SQL 管理 | XML/注解配置，动态 SQL 支持 | 硬编码 SQL，灵活性低 |
| 结果映射 | 自动反射映射或自定义ResultMap | 手动遍历ResultSet赋值 |
| 事务控制 | 通过SqlSession或 Spring 集成管理 | 手动管理Connection事务 |
| 性能优化 | 缓存机制、批处理支持 | 需手动优化 |
| 代码量 | 减少重复代码，提升开发效率 | 冗余代码多，维护成本高 |
