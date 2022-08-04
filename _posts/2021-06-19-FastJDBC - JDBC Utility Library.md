---
layout: post
title: FastJDBC - Quick development kit based on spring-boot-starter-jdbc
date: 2021-06-19 15:32:24.000000000 +09:00
---

<code>fastjdbc-spring-boot-starter</code>, which is a <code>jdbc</code> quick development kit based on the <code>SpringBoot</code> framework. It is actually a secondary encapsulation of the <code>NamedJdbcTemplate</code> provided by the spring framework, and provides an annotation-based API configuration method to operate SQL. At the same time, you can still use <code>spring's spring-boot-starter-data-jdbc</code> function.

##  Compatibility

* JDK 1.8 (or later)
* <code>SpringBoot Framework 2.2.x </code>(or later)

## Install

``` xml
<dependency>
	<groupId>com.github.paganini2008.springdessert</groupId>
	<artifactId>fastjdbc-spring-boot-starter</artifactId>
	<version>2.0.3</version>
</dependency>
```

## Quick Start

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

**Description:**

1. @Insert returns the primary key ID, which can be of type int or long
2. @Update returns the number of affected rows, it can execute insert, update, delete statements
3. @Batch returns the number of affected rows
4. @Get can return an object or a single value, just set the attribute <code>javaType=true</code>
5. @Query and @Select are very similar. @Query returns a list without pagination. @Select supports both pagination and lists. It returns a <code>ResultSetSlice</code> object. This object is extremely powerful. Friends who are interested can study it.
6. The <code>sql</code> statement is written in the same way as the <code>NamedParameterJdbcTemplate</code> in the Spring framework, and the <code>sql</code> statement is actually executed through it.


Finally，add <code>@DaoScan</code> into your code and make it work, for example, <code>XXXConfiguration.java</code>

``` java
@DaoScan(basePackages = "com.yourcompany.project.base.dao")
@Configuration(proxyBeanMethods = false)
public class XXXConfiguration {

}
```

Git Repository：https://github.com/paganini2008/springdessert.git


