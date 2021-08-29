---
layout: post
title: FastJDBC - Quickly Developing of Java Programs based on JDBC
date: 2021-06-19 15:32:24.000000000 +09:00
---

FastJDBC encapsulates the <code>NamedJdbcTemplate</code> component of spring framework, and provides another way based on Java annotation to execute SQL, rather than call <code>NamedJdbcTemplate</code> API.

### Install
------------------------
``` xml
		<dependency>
			<groupId>com.github.paganini2008.springdessert</groupId>
			<artifactId>fastjdbc-spring-boot-starter</artifactId>
			<version>2.0.3</version>
		</dependency>
```

### Quick Start
------------------------
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

**API notes:**

1. @Insert returns the primary key ID, which can be of type int or long
2. @Update returns the number of affected rows. It can execute insert, update and delete statements
3. @ Batch also returns the number of affected rows, Set parameter with @Args
4. @ Get can return an POJO or a single value (wrapper type or primitive type, set the property <code>javaType = true</code>). 
5. @Example parameter can be a POJO object or a <code>java.utils.Map</code>, @Arg represents one parameter and @Args represents multiple parameters for @Batch operation
6. @Query returns a list, where @SQL represents dynamic SQL. For example, you can dynamically assemble SQL text according to query conditions
7. @Query is very similar to @Select, but @Query is non paged, @Select supports both paging and list, and returns the <code>ResultSetSlice</code> Object from [devtools-lang](https://paganini2008.github.io/2021/06/Devtools-Lang-Java-Basic-Utility-Tool/), which is very powerful. Interested friends can study it
8. The SQL statement is written in the same way as the <code>NamedParameterJdbctemplate</code> in the spring framework. In essence, it is used to execute SQL statements
(refer to the source code for the specific usage of API)



**Finally**，add <code>@DaoScan</code> into your code and make it work, for example, <code>XXXConfiguration.java</code>

``` java
@DaoScan(basePackages = "com.yourcompany.project.base.dao")
@Configuration(proxyBeanMethods = false)
public class XXXConfiguration {

}
```
Git Repository：https://github.com/paganini2008/springdessert.git


