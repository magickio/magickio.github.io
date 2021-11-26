---
title: Mybatis
date: 2021-11-26 21:50:40
tags: Mybatis
---

# MyBatis 接口式编程 Hello World

# 搭建环境

## 搭建数据库MySQL

```json
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(30) DEFAULT NULL,
  `pwd` varchar(30) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;


INSERT INTO `user`(`name`, `pwd`) VALUES ('hong', '123456');
INSERT INTO `user`(`name`, `pwd`) VALUES ('Tom', '123456');
INSERT INTO `user`(`name`, `pwd`) VALUES ('Jerry', '123456');
```

#### Maven

```xml
<dependencies>
    <!-- MySQL驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>
    <!-- MyBatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.2</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.18</version>
    </dependency>
    <!-- 日志 -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>

    <!-- Junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

#### jdbc.properties

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?userSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8
username=root
password=1234
```

#### log4j.properties



```properties
log4j.rootLogger=DEBUG,A1

log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=[%t] [%c]-[%p] %m%n
```



### 编写代码



#### 实体类



```java
package org.hong.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String name;
    private String pwd;
}
```



#### Mapper 接口



```java
package org.hong.mapper;

import org.hong.pojo.User;

import java.util.List;

public interface UserMapper {
    List<User> getAll();
}
```



#### Mapper.xml 文件



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<!--
    namespace:名称空间, 对应接口的全类名
 -->
<mapper namespace="org.hong.mapper.UserMapper">
    <!--
        select: 配置查询
        id: 唯一标识, 对应接口中的方法名
        resultType: 返回值类型, 类的全类名, 如果返回值是集合写集合中泛型的类型
     -->
    <select id="getAll" resultType="org.hong.pojo.User">
        select * from user
    </select>
</mapper>
```



#### MyBatis 核心配置文件



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<!-- configuration: 核心配置文件 -->
<configuration>

    <!-- 
		properties: 引入外部properties文件 必须放在最前面,否则会报错
			resource: 类路径下
			url: 磁盘路径或网络路径
 	-->
    <properties resource="jdbc.properties"/>
    
    <!-- 设置日志输出, 方便观察sql语句和参数 -->
    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>

    <!--
        environments配置项目的运行环境, 可以配置多个
        default: 启用的环境
     -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!-- 数据库连接信息 -->
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 每一个Mapper.xml都需要在MyBatis核心配置文件中注册!!! -->
    <mappers>
        <mapper resource="org/hong/mapper/UserMapper.xml"/>
    </mappers>

</configuration>
```



### 测试用例



```java
package org.hong.test;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.hong.mapper.UserMapper;
import org.hong.pojo.User;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class HelloTest {
    public SqlSessionFactory getSqlSessionFactory() throws IOException {
        // MyBatis全局配置文件路径
        String resource = "mybatis-config.xml";
        // 获取MyBatis全局配置文件的输入流
        InputStream is = Resources.getResourceAsStream(resource);
        // 获取SqlSessionFactory对象
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
        return factory;
    }

    @Test
    public void testHello() throws IOException {
        // 1、获取SqlSessionFactory对象
        SqlSessionFactory factory = getSqlSessionFactory();

        // 2、获取SqlSession对象
        SqlSession openSession = factory.openSession();
        try {
            // 3、获取接口的实现类对象
            // 会为接口自动创建代理对象, 代理对象去执行增删改查方法, sql语句会从mapper.xml中获取
            UserMapper mapper = openSession.getMapper(UserMapper.class);
            List<User> list = mapper.getAll();
            list.forEach(System.out::println);
        } finally {
            // 4、SqlSession代表和数据库的一次对话, 用完必须关闭
            openSession.close();
        }
    }
}
```



### 控制台打印



```shell
## 发送的sql语句就是我们在mapper.xml中配置的sql语句
[org.hong.mapper.UserMapper.getAll]-==>  Preparing: select * from user 
[org.hong.mapper.UserMapper.getAll]-==> Parameters: 
[org.hong.mapper.UserMapper.getAll]-<==      Total: 4
User(id=1, name=hong, pwd=123456)
User(id=2, name=Tom, pwd=123456)
User(id=3, name=Jerry, pwd=123456)
```

# CRUD 初体验

接着上面的来！！！



### MyBatisUtil 工具类



```java
package org.hong.util;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

public class MyBatisUtil {

    private static SqlSessionFactory sqlSessionFactory;

    static {
        try {
            // 获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。
    // SqlSession 提供了在数据库执行 SQL 命令所需的所有方法
    public static SqlSession getSqlSession(){
        // openSession(): 此方式打开SQL会话, 事务是开启状态
        // openSession(true): 此方式打开SQL会话, 事务是关闭状态
        return sqlSessionFactory.openSession();
    }

    public static SqlSessionFactory getSqlSessionFactory() {
        return sqlSessionFactory;
    }
}
```



### save



**接口方法**



```java
// 保存
int save(User user);
```



**方法映射**



```xml
<!--
    inserte: 配置insert语句
        id: 对应的方法名
        parameterType: 指定参数类型为pojo, 可以直接写属性名获得属性值, 优先调用getting方法, 如果没有getting方法则直接从属性中取值
 -->
<insert id="save" parameterType="org.hong.pojo.User">
    insert into user(name, pwd) values(#{name}, #{pwd})
</insert>
```



**测试用例**



```java
@Test
public void save(){
    // 1.获取sqlSession对象
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    // 2.获取需要的mapper接口的代理对象
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    // 3.调用对应的方法执行操作
    User user = new User();
    user.setName("SAVE");
    user.setPwd("123");
    int save = mapper.save(user);
    System.out.println(save);
    System.out.println(user);
    // 4.提交事务
    sqlSession.commit();
    // 5.关闭sqlSession
    sqlSession.close();
}
```



**控制台打印**



```shell
## 发送的sql语句
[main] [org.hong.mapper.UserMapper.save]-[DEBUG] ==>  Preparing: insert into user(name, pwd) values(?, ?) 
## 预编译放入的参数值, 从传入的pojo对象中取出对应的属性值
[main] [org.hong.mapper.UserMapper.save]-[DEBUG] ==> Parameters: SAVE(String), 123(String)
## 影响行数
[main] [org.hong.mapper.UserMapper.save]-[DEBUG] <==    Updates: 1
## 编写的save方法返回的值int类型, 意义是数据库影响行数
1
## 插入后的对象打印, id值并没有回填到对象中, 因为我们没有开启这个功能, 后面会说到
User(id=null, name=SAVE, pwd=123)
```



### get



**接口方法**



```java
// 根据id查询
User get(int id);
```



**方法映射**



```xml
<!-- 方法参数是int类型的, 所以没有写parameterType属性 -->
<select id="get" resultType="org.hong.pojo.User">
    <!-- 插入id值 -->
    select * from user where id = #{id}
</select>
```



**测试用例**



```java
@Test
public void get(){
    // 1.获取sqlSession对象
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    // 2.获取需要的mapper接口的代理对象
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    // 3.调用对应的方法执行操作
    User user = mapper.get(1);
    System.out.println(user);
    // 4.提交事务
    sqlSession.commit();
    // 5.关闭sqlSession
    sqlSession.close();
}
```



**控制台打印**



```shell
[main] [org.hong.mapper.UserMapper.get]-[DEBUG] ==>  Preparing: select * from user where id = ? 
[main] [org.hong.mapper.UserMapper.get]-[DEBUG] ==> Parameters: 1(Integer)
[main] [org.hong.mapper.UserMapper.get]-[DEBUG] <==      Total: 1
User(id=1, name=hong, pwd=123456)
```



### update



**接口方法**



```java
// 修改
int update(User user);
```



**方法映射**



```xml
<update id="update" parameterType="org.hong.pojo.User">
    update user set name = #{name}, pwd = #{pwd} where id = #{id}
</update>
```



**测试用例**



```java
@Test
public void update(){
    // 1.获取sqlSession对象
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    // 2.获取需要的mapper接口的代理对象
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    // 3.调用对应的方法执行操作
    User user = mapper.get(1);
    user.setName("谢禹宏");
    int update = mapper.update(user);
    System.out.println(user);
    // 4.提交事务
    sqlSession.commit();
    // 5.关闭sqlSession
    sqlSession.close();
}
```



**控制台打印**



```shell
## 查询sql
[main] [org.hong.mapper.UserMapper.get]-[DEBUG] ==>  Preparing: select * from user where id = ? 
[main] [org.hong.mapper.UserMapper.get]-[DEBUG] ==> Parameters: 1(Integer)
[main] [org.hong.mapper.UserMapper.get]-[DEBUG] <==      Total: 1
## 修改sql
[main] [org.hong.mapper.UserMapper.update]-[DEBUG] ==>  Preparing: update user set name = ?, pwd = ? where id = ? 
[main] [org.hong.mapper.UserMapper.update]-[DEBUG] ==> Parameters: 谢禹宏(String), 123456(String), 1(Integer)
[main] [org.hong.mapper.UserMapper.update]-[DEBUG] <==    Updates: 1
User(id=1, name=谢禹宏, pwd=123456)
```



### delete



**接口方法**



```java
// 删除
boolean delete(int id);
```



**方法映射**



```xml
<delete id="delete">
    delete from user where id = #{id}
</delete>
```



**测试用例**



```java
@Test
public void delete(){
    // 1.获取sqlSession对象
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    // 2.获取需要的mapper接口的代理对象
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    // 3.调用对应的方法执行操作
    boolean delete = mapper.delete(10);
    System.out.println(delete);
    // 4.提交事务
    sqlSession.commit();
    // 5.关闭sqlSession
    sqlSession.close();
}
```



**控制台打印**



```shell
[main] [org.hong.mapper.UserMapper.delete]-[DEBUG] ==>  Preparing: delete from user where id = ? 
[main] [org.hong.mapper.UserMapper.delete]-[DEBUG] ==> Parameters: 10(Integer)
[main] [org.hong.mapper.UserMapper.delete]-[DEBUG] <==    Updates: 0
## 因为我们删除一个不存在的数据, 影响行数为0, 所以MyBatis返回false
false
```



### 最终版



#### Mapper 接口



```java
package org.hong.mapper;

import org.hong.pojo.User;


public interface UserMapper {
    // 保存
    int save(User user);

    // 根据id查询
    User get(int id);

    // 修改
    int update(User user);

    // 删除
    boolean delete(int id);
}
```



#### Mapper.xml



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.hong.mapper.UserMapper">

    <!--
        inserte: 配置insert语句
            id: 对应的方法名
            parameterType: 指定参数类型为pojo, 可以直接写属性名获得属性值, 优先调用getting方法, 如果没有getting方法则直接从属性中取值
     -->
    <insert id="save" parameterType="org.hong.pojo.User">
        insert into user(name, pwd) values(#{name}, #{pwd})
    </insert>
    <!-- 方法参数是int类型的, 所以没有写parameterType属性 -->
    <select id="get" resultType="org.hong.pojo.User">
        <!-- 插入id值 -->
        select * from user where id = #{id}
    </select>
    <update id="update" parameterType="org.hong.pojo.User">
        update user set name = #{name}, pwd = #{pwd} where id = #{id}
    </update>
    <delete id="delete">
        delete from user where id = #{id}
    </delete>
</mapper>
```



#### 测试用例



```java
package org.hong.test;

import org.apache.ibatis.session.SqlSession;
import org.hong.mapper.UserMapper;
import org.hong.pojo.User;
import org.hong.util.MyBatisUtil;
import org.junit.Test;

public class CRUDTest {
    @Test
    public void save(){
        // 1.获取sqlSession对象
        SqlSession sqlSession = MyBatisUtil.getSqlSession();
        // 2.获取需要的mapper接口的代理对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        // 3.调用对应的方法执行操作
        User user = new User();
        user.setName("SAVE ID");
        user.setPwd("123");
        int save = mapper.save(user);
        System.out.println(save);
        System.out.println(user);
        // 4.提交事务
        sqlSession.commit();
        // 5.关闭sqlSession
        sqlSession.close();
    }

    @Test
    public void get(){
        // 1.获取sqlSession对象
        SqlSession sqlSession = MyBatisUtil.getSqlSession();
        // 2.获取需要的mapper接口的代理对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        // 3.调用对应的方法执行操作
        User user = mapper.get(1);
        System.out.println(user);
        // 4.提交事务
        sqlSession.commit();
        // 5.关闭sqlSession
        sqlSession.close();
    }

    @Test
    public void update(){
        // 1.获取sqlSession对象
        SqlSession sqlSession = MyBatisUtil.getSqlSession();
        // 2.获取需要的mapper接口的代理对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        // 3.调用对应的方法执行操作
        User user = mapper.get(1);
        user.setName("谢禹宏");
        int update = mapper.update(user);
        System.out.println(user);
        // 4.提交事务
        sqlSession.commit();
        // 5.关闭sqlSession
        sqlSession.close();
    }

    @Test
    public void delete(){
        // 1.获取sqlSession对象
        SqlSession sqlSession = MyBatisUtil.getSqlSession();
        // 2.获取需要的mapper接口的代理对象
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        // 3.调用对应的方法执行操作
        boolean delete = mapper.delete(10);
        System.out.println(delete);
        // 4.提交事务
        sqlSession.commit();
        // 5.关闭sqlSession
        sqlSession.close();
    }
}
```

# save 回填主键值

在上面 save 的基础上进行更改



### 获取自增主键的值



**接口方法**



```java
// 保存
int save(User user);
```



**方法映射**



```xml
<!--
    useGeneratedKeys="true": 开启获取自增主键的策略
    keyColumn: 指定数据库主键的列名
    keyProperty: 指定对应的主键属性, ps(获取到主键值后, 将这个值封装给javaBean的哪个属性)
 -->
<insert id="save" parameterType="org.hong.pojo.User" useGeneratedKeys="true" keyColumn="id" keyProperty="id">
    insert into user(name, pwd) values(#{name}, #{pwd})
</insert>
```



**测试用例**



```java
@Test
public void save(){
    // 1.获取sqlSession对象
    SqlSession sqlSession = MyBatisUtil.getSqlSession();
    // 2.获取需要的mapper接口的代理对象
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    // 3.调用对应的方法执行操作
    User user = new User();
    user.setName("SAVE ID");
    user.setPwd("123");
    int save = mapper.save(user);
    System.out.println(save);
    System.out.println(user);
    // 4.提交事务
    sqlSession.commit();
    // 5.关闭sqlSession
    sqlSession.close();
}
```



**控制台打印**



```shell
## 发送的sql语句
[main] [org.hong.mapper.UserMapper.save]-[DEBUG] ==>  Preparing: insert into user(name, pwd) values(?, ?) 
[main] [org.hong.mapper.UserMapper.save]-[DEBUG] ==> Parameters: SAVE ID(String), 123(String)
[main] [org.hong.mapper.UserMapper.save]-[DEBUG] <==    Updates: 1
1
## 主键回填到了对象中
User(id=4, name=SAVE ID, pwd=123)
```



### 获取Oracle序列的值



这里就不具体演示了，看不懂就算了，反正用的也不多。



```xml
<insert id="addEmp">
	<!-- 
		selectKey: 配置查询主键的sql语句
            keyProperty:查出的主键值封装给javaBean的哪个属性
            order: 
                BEFORE:当前sql在插入sql之前运行
                AFTER:当前sql在插入sql之后运行
            resultType:查出的数据的返回值类型

            BEFORE运行顺序:
                先运行selectKey查询id的sql;查出id值封装给javaBean的id属性
                再运行插入的sql;就可以取出id属性对应的值
            AFTER运行顺序:
                先运行插入的sql（从序列中取出新值作为id）;
                再运行selectKey查询id的sql, 回填到javaBean的id属性中
	 -->
	<selectKey keyProperty="id" order="BEFORE" resultType="Integer">
		<!-- 编写查询主键的sql语句 -->
		select EMPLOYEES_SEQ.nextval from dual 
	</selectKey>
	<!-- 插入时的主键是从序列中拿到的 -->
	<!-- BEFORE:-->
	insert into employees(EMPLOYEE_ID,LAST_NAME,EMAIL) 
	values(#{id},#{lastName},#{email}) 
</insert>
```

