---
layout: post
title: FastJDBC: A Quick and Convenient JDBC Development Kit
date: 2021-06-19 15:32:24.000000000 +09:00
---

FastJDBC提供了对spring框架中NamedJdbcTemplate类的二次封装，提供一种基于注解方式的API配置方式去操作sql，而不是调用NamedJdbcTemplate的方法去传递sql语句。

### 安装
------------------------
``` xml
		<dependency>
			<groupId>com.github.paganini2008.springdessert</groupId>
			<artifactId>fastjdbc-spring-boot-starter</artifactId>
			<version>2.0.3</version>
		</dependency>
```
（以最新版为准）

### 快速开始
------------------------
下面举几个例子，看看fastjdbc-spring-boot-starter是怎么使用的
``` java
@Dao
public interface UserDao {

	@Insert("insert into tb_user(username,password,age) values (:username,:password,:age)")
	int saveUser(@Example User user);

	@Update("update tb_user set username=:username, password=:password where id=:id")
	int updateUser(@Example User user);

	@Update("delete from tb_user where id=:id")
	int deleteUser(@Arg("id") int id);
	
	@Batch("insert into tb_user(username,password,age) values (:username,:password,:age)")
	int batchSaveUser(@Args List<User> userList);

	@Get("select * from tb_user where id=:id")
	User getById(@Arg int id);
	
	@Query("select * from tb_user order by create_time desc")
	List<Map<String, Object>> queryUser();

    @Query("select * from tb_user where 1=1 @sql order by create_time desc")
	List<Map<String, Object>> queryUserByCondition(@Sql String whereCondition, @Example Map<String,Object> queryExample);

	@Select("select * from tb_user order by create_time desc")
	ResultSetSlice<User> selectUser();

}
```

API很简单，但需要的注意事项：
1. @Insert 返回的是主键ID, 可以是int或long类型的
2. @Update 返回的是受影响的行数，它可以执行insert, update, delete 语句
3. @Batch 返回的也是受影响的行数
4. @Get 可以返回对象也可以返回单个值(包装类型或基础类型)，设置属性javaType=true即可
5. @Example 参数可以是Pojo对象也可以是Map, @Arg表示一个参数，@Args表示多个参数，用于批处理
5. @Query 返回列表，其中@Sql表示是动态sql, 比如你可以根据查询条件动态拼装sql
7. @Query和@Select很像，@Query，是不分页的，@Select既支持分页，又支持列表，返回ResultSetSlice对象，这个对象异常强大，有兴趣的朋友可以研究一下
8. sql语句的写法和Spring框架中NamedParameterJdbcTemplate的写法一致，实质上还是通过它来执行sql语句的
（API具体用法可以参考源码）

最后，在你Spring Boot应用中，在你自己的Configuration类上中加入Dao扫描器即可，比如：
``` java
@DaoScan(basePackages = "com.yourcompany.project.base.dao")
@Configuration(proxyBeanMethods = false)
public class YourConfiguration {

}
```
git地址：https://github.com/paganini2008/springdessert.git


