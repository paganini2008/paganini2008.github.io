---
layout: post
title: FastJPA: A Graceful Utility Tool of JPA
date: 2021-06-21 08:30:00.000000000 +09:00
---

FastJPA提供了对SpringBoot框架中关于对JPA的操作的二次封装 ，提供了面向对象的方式来操作JPQL/HQL，旨在减少sql语句编写，快速提高开发效率，使代码书写显的更加优雅和增加可读性

### 工具特性：
---------------------------
* 面向对象方式的更新、删除和查询操作
* 查询指定列名和函数列
* 分组查询和过滤
* 列表查询和过滤
* 表连接查询和过滤
* 支持子查询
* 分页查询和过滤

### 安装
-------------------------------
``` xml
<dependency>
      <groupId>com.github.paganini2008.springworld</groupId>
      <artifactId>fastjpa-spring-boot-starter</artifactId>
      <version>2.0.2</version>
</dependency>
```
fastjpa-spring-boot-starter 依赖spring-boot-starter-data-jpa, 实质上是对JPA  Criteria查询API(QBC)的再封装，并设计成流式风格的API(有点类似python的orm框架sqlalchemy) ，使得JPA面向对象查询的API不再难用

### fastjpa 核心接口：
----------------------------
* EntityDao
* Model
* JpaQuery
* JpaPage
* Filter
* Column
* Field
* JpaGroupBy
* JpaSort
* JpaPageResultSet
* JpaQueryResultSet
* JpaUpdate
* JpaDelete
大家有兴趣的话，可以研究其源码

##### 下面通过几个示例来演示一下fastjpa的几个核心接口的用法
比如，现在有3个实体，用户，订单，商品

##### 用户实体
``` java
@Getter
@Setter
@Entity
@Table(name = "demo_user")
public class User {
	
	@Id
	@Column(name = "id", nullable = false, unique = true)
	private Long id;
	
	@Column(name = "name", nullable = false, length = 45)
	private String name;
	
	@Column(name = "phone", nullable = false, length = 45)
	private String phone;
	
	@Column(name = "vip", nullable = true)
	@org.hibernate.annotations.Type(type = "yes_no")
	private Boolean vip;

}
```
##### 商品实体
``` java
@Getter
@Setter
@Entity
@Table(name = "demo_product")
public class Product {

	@Id
	@Column(name = "id", nullable = false, unique = true)
	private Long id;

	@Column(name = "name", nullable = false, length = 45)
	private String name;

	@Column(name = "price", nullable = false, precision = 11, scale = 2)
	private BigDecimal price;

	@Column(name = "origin", nullable = true, length = 225)
	private String origin;

}
```

##### 订单实体
``` java
@Getter
@Setter
@Entity
@Table(name = "demo_order")
public class Order {

	@Id
	@Column(name = "id", nullable = false, unique = true)
	private Long id;

	@Column(name = "discount", nullable = true)
	private Float discount;

	@Column(name = "price", nullable = false, precision = 11, scale = 2)
	private BigDecimal price;

	@OneToOne(targetEntity = Product.class)
	@JoinColumn(nullable = false, name = "product_id", foreignKey = @ForeignKey(name = "none", value = ConstraintMode.NO_CONSTRAINT))
	private Product product;

	@ManyToOne(targetEntity = User.class)
	@JoinColumn(nullable = false, name = "user_id", foreignKey = @ForeignKey(name = "none", value = ConstraintMode.NO_CONSTRAINT))
	private User user;

	@Temporal(TemporalType.TIMESTAMP)
	@Column(name = "create_time", columnDefinition = "timestamp null ")
	private Date createTime;

}
```
然后定义对应的Dao，需要继承fastjpa提供的EntityDao，但如果你不想使用fastjpa, 直接继承JpaRepositoryImplementation就行了
##### UserDao
``` java
public interface UserDao extends EntityDao<User, Long> {

}
```
##### OrderDao
``` java
public interface OrderDao extends EntityDao<Order, Long> {

}
```
##### ProductDao
``` java
public interface ProductDao extends EntityDao<Product, Long> {

}
```
EntityDao是fastjpa 所有API的入口，看一下它的源码：
``` java
@NoRepositoryBean
public interface EntityDao<E, ID> extends JpaRepositoryImplementation<E, ID>, NativeSqlOperations<E> {

	Class<E> getEntityClass();

	boolean exists(Filter filter);

	long count(Filter filter);

	List<E> findAll(Filter filter);

	List<E> findAll(Filter filter, Sort sort);

	Page<E> findAll(Filter filter, Pageable pageable);

	Optional<E> findOne(Filter filter);

	<T extends Comparable<T>> T max(String property, Filter filter, Class<T> requiredType);

	<T extends Comparable<T>> T min(String property, Filter filter, Class<T> requiredType);

	<T extends Number> T avg(String property, Filter filter, Class<T> requiredType);

	<T extends Number> T sum(String property, Filter filter, Class<T> requiredType);

	JpaUpdate<E> update();

	JpaDelete<E> delete();

	JpaQuery<E, E> query();

	<T> JpaQuery<E, T> query(Class<T> resultClass);

	JpaQuery<E, Tuple> multiquery();

	JpaPage<E, E> select();

	<T> JpaPage<E, T> select(Class<T> resultClass);

	JpaPage<E, Tuple> multiselect();

}
```
其中：
update()方法对应的是update操作
delete()方法对应的是delete操作
query()方法对应的是列表操作
select()方法对应的是分页操作
multiquery()方法对应的也是列表操作，但和query()方法不同的是，它返回的是javax.persistence.Tuple类型的数据，它用于封装分组或多表连接的查询的数据结构
multiselect()也是类似的
另外说一下，fastjpa组件还支持本地sql查询的使用，会在文章的末尾粗略介绍一下。

###### 现在，回到前面的例子，继续讲一下如何使用fastjpa的API, 
首先要对这3个实体，分别插入一些数据，并设置相关的关联关系
比如这里，假定一个商品一个订单，一个用户可以下多个订单，这里只是为了演示一下而已

##### Filter
相当于where条件
支持 lt, lte, gt, gte, eq, ne, like, in, between, isNull, notNull等比较操作符
支持 and, or, not 逻辑操作符
###### 比较操作符举例：
``` java
		LogicalFilter filter = Restrictions.gt("price", 50); // 价格大于50元
		productDao.query().filter(filter).selectThis().list().forEach(pro -> {
			System.out.println(pro);
		});
// Hibernate: select product0_.id as id1_1_, product0_.name as name2_1_, product0_.origin as origin3_1_, product0_.price as price4_1_ from demo_product product0_ where product0_.price>50.0
```
类似的还有：
``` java
LogicalFilter filter = Restrictions.between("price", 10, 50);
filter = Restrictions.in("price", Arrays.asList(10,20,30,40,50));
filter =Restrictions.like("name", "%猴头菇%");
filter = Restrictions.eq("orignal", "Shanghai");
```
###### 逻辑操作符举例：
and
``` java
LogicalFilter filter = Restrictions.between("price", 10, 50);
filter = filter.and(Restrictions.like("name", "%猴头菇%"));
filter = filter.and(Restrictions.eq("orignal", "Shanghai"));
// 相当于 where price between (10,50) and name like '%猴头菇%' and orignal='Shanghai'
productDao.query().filter(filter).selectThis().list().forEach(pro -> {
	System.out.println(pro);
});
```
or
``` java
LogicalFilter filter = Restrictions.eq("orignal", "Shanghai");
filter = filter.or(Restrictions.eq("orignal", "New York"));
// 相当于 where orignal='Shanghai' or orignal='New York'
productDao.query().filter(filter).selectThis().list().forEach(pro -> {
	System.out.println(pro);
});
```
not
``` java
LogicalFilter filter = Restrictions.eq("orignal", "Shanghai");
filter = filter.and(Restrictions.eq("orignal", "New York"));
filter = filter.not();
 // 取反，相当于 where orignal!='Shanghai' and orignal!='New York'
productDao.query().filter(filter).selectThis().list().forEach(pro -> {
	System.out.println(pro);
});
```
##### JpaGroupBy
分组，相当于group by
举例：
``` java
productDao.multiquery().groupBy("origin").select(Column.forName("origin"), Fields.count(Fields.toInteger(1)).as("count")).list()
.forEach(t -> {
		System.out.println("origin: "+t.get("origin") + "\tcount: " + t.get("count"));
});
// Hibernate: select product0_.origin as col_0_0_, count(1) as col_1_0_ from demo_product product0_ group by product0_.origin

```
##### Column
表示一个列
举例：
```java
productDao.multiquery().select(Column.forName("name"), Column.forName("price")).list(10).forEach(t -> {
        System.out.println("name: "+t.get("name") + "\t price: " + t.get("price"));
});
// Hibernate: select product0_.name as col_0_0_, product0_.price as col_1_0_ from demo_product product0_ limit ?
```
查询多个列也可以这样：
``` java
productDao.multiquery().select(new String[] { "name", "price" }).list(10).forEach(t -> {
	System.out.println("name: " + t.get("name") + "\t price: " + t.get("price"));
});
// 或者这样：
ColumnList columnList = new ColumnList().addColumn("name").addColumn("price");
productDao.multiquery().select(columnList).list(10).forEach(t -> {
	System.out.println("name: " + t.get("name") + "\t price: " + t.get("price"));
});
```
##### Field
用来表示函数，常量等
举例：
聚合函数
``` java
ColumnList columnList = new ColumnList()
				.addColumn(Fields.max("price", BigDecimal.class), "maxPrice")
				.addColumn(Fields.min("price", BigDecimal.class), "minPrice")
				.addColumn(Fields.avg("price", Double.class), "avgPrice")
				.addColumn(Fields.count(Fields.toInteger(1)), "count")
				.addColumn("origin");
productDao.multiquery().groupBy("origin").select(columnList).setTransformer(Transformers.asBean(ProductVO.class)).list().forEach(vo -> {
	System.out.println(vo);
});
```
常用函数：
concat
``` java
ColumnList columnList = new ColumnList()
	.addColumn(Fields.concat(Fields.concat(Fields.max("price", String.class), "/"), Fields.min("price", String.class)), "repr")
	.addColumn("origin");
productDao.multiquery().groupBy("origin").select(columnList).setTransformer(Transformers.asBean(ProductVO.class)).list().forEach(vo -> {
	System.out.println(vo);
});
// Hibernate: select concat(concat(max(cast(product0_.price as char)), '/'), min(cast(product0_.price as char))) as col_0_0_, product0_.origin as col_1_0_ from demo_product product0_ group by product0_.origin
```
其他常用函数
``` java
ColumnList columnList = new ColumnList()
       .addColumn(Function.build("LOWER", String.class, "name"), "name")
       .addColumn(Function.build("UPPER", String.class, "origin"), "origin");
productDao.multiquery().select(columnList).list(10).forEach(t -> {
	System.out.println("name: " + t.get("name") + "\t origin: " + t.get("origin"));
});
// Hibernate: select lower(product0_.name) as col_0_0_, upper(product0_.origin) as col_1_0_ from demo_product product0_ limit ?
```
Case When
``` java
		IfExpression<String, String> ifExpression = new IfExpression<String, String>(Property.forName("origin", String.class));
		ifExpression = ifExpression.when("Shanghai", "Asia")
				                   .when("Tokyo", "Asia")
				                   .when("New York", "North America")
				                   .when("Washington", "North America")
				                   .otherwise("Other Area");
		ColumnList columnList = new ColumnList().addColumn(ifExpression, "Area")
				                                .addColumn(Fields.count(Fields.toInteger(1)), "Count");
		productDao.multiquery().groupBy(Fields.toInteger(1)).select(columnList).list().forEach(t -> {
			System.out.println("Area: " + t.get(0) + "\t Count: " + t.get(1));
		});
// Hibernate: select case product0_.origin when 'Shanghai' then 'Asia' when 'Tokyo' then 'Asia' when 'New York' then 'North America' when 'Washington' then 'North America' else 'Other Area' end as col_0_0_, count(1) as col_1_0_ from demo_product product0_ group by 1
```
##### 子查询示例
###### 示例1
``` java
		JpaSubQuery<Order,Order> subQuery = orderDao.query().subQuery(Order.class, "o").filter(Restrictions.eq("o", "id", "100")).select("o", "product.id");
		Product product = productDao.query().filter(Restrictions.eq("id", subQuery)).selectThis().first();
		System.out.println(product);
// Hibernate: select product0_.id as id1_1_, product0_.name as name2_1_, product0_.origin as origin3_1_, product0_.price as price4_1_ from demo_product product0_ where product0_.id=(select order1_.product_id from demo_order order1_ where order1_.id=100) limit ?
```
###### 示例2
``` java
		JpaQuery<Order,Order> jpaQuery = orderDao.query();
		JpaSubQuery<Product, BigDecimal> subQuery = jpaQuery.subQuery(Product.class, "p", BigDecimal.class)
				.select(Fields.avg(Property.forName("p", "price")));
		jpaQuery.filter(Restrictions.gte("price", subQuery)).selectThis().list(10).forEach(pro -> {
			System.out.println(pro);
		});
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where order0_.price>=(select avg(product1_.price) from demo_product product1_) limit ?
```
###### 示例3
``` java
		JpaQuery<Order,Order> jpaQuery = orderDao.query();
		JpaSubQuery<Product, BigDecimal> subQuery = jpaQuery.subQuery(Product.class, "p", BigDecimal.class)
				.select(Fields.avg(Property.forName("p", "price")));
		jpaQuery.filter(Restrictions.gte("price", subQuery)).selectThis().list(10).forEach(pro -> {
			System.out.println(pro);
		});
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where 1=1 and (exists (select user1_.id from demo_user user1_ where user1_.name=order0_.receiver)) limit ?

```
##### 排序示例
``` java
		orderDao.query().filter(Restrictions.gte("price", 50)).sort(JpaSort.desc("createTime"), JpaSort.asc("price")).selectThis().list(10)
				.forEach(pro -> {
					System.out.println(pro);
				});
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where order0_.price>=50.0 order by order0_.create_time desc, order0_.price asc limit ?
```

##### 关联查询示例
###### 左连接
``` java
		PageResponse<Tuple> pageResponse = orderDao.multiselect().leftJoin("product", "p")
				.filter(Restrictions.gte("p", "price", 50)).sort(JpaSort.desc("createTime")).selectAlias("p")
				.list(PageRequest.of(10));
		for (PageResponse<Tuple> current : pageResponse) {
			System.out.println("第" + current.getPageNumber() + "页");
			for (Tuple tuple : current.getContent()) {
				System.out.println(Arrays.toString(tuple.toArray()));
			}
		}
// Hibernate: select product1_.id as id1_1_, product1_.name as name2_1_, product1_.origin as origin3_1_, product1_.price as price4_1_ from demo_order order0_ left outer join demo_product product1_ on order0_.product_id=product1_.id where product1_.price>=50.0 order by order0_.create_time desc limit ?, ?
```
###### 右连接
``` java
		ColumnList columnList = new ColumnList();
		columnList.addColumn("id");
		columnList.addColumn("u", "name");
		columnList.addColumn("price");
		columnList.addColumn("createTime");
		PageResponse<Order> pageResponse = orderDao.select().rightJoin("user", "u").filter(Restrictions.gte("price", 50))
				.sort(JpaSort.desc("createTime")).select(columnList).list(PageRequest.of(10));
		for (PageResponse<Order> current : pageResponse) {
			System.out.println("第" + current.getPageNumber() + "页");
			for (Order order : current.getContent()) {
				System.out.println(order);
			}
		}
  // 然而运行上面代码，JPA会报错，因为JPA目前尚不支持Right Join!
```
###### 内连接
``` java
		ColumnList columnList = new ColumnList();
		columnList.addColumn("id");
		columnList.addColumn("p","name");
		columnList.addColumn(Property.forName("p", "price"),"originalPrice");
		columnList.addColumn("price");
		columnList.addColumn("createTime");
		PageResponse<Tuple> pageResponse = orderDao.multiselect().join("product", "p")
				.filter(Restrictions.gte("p", "price", 50)).sort(JpaSort.desc("createTime")).select(columnList)
				.list(PageRequest.of(10));
		for (PageResponse<Tuple> current : pageResponse) {
			System.out.println("第" + current.getPageNumber() + "页");
			for (Tuple tuple : current.getContent()) {
				System.out.println(Arrays.toString(tuple.toArray()));
			}
		}
// Hibernate: select order0_.id as col_0_0_, product1_.name as col_1_0_, product1_.price as col_2_0_, order0_.price as col_3_0_, order0_.create_time as col_4_0_ from demo_order order0_ inner join demo_product product1_ on order0_.product_id=product1_.id where product1_.price>=50.0 order by order0_.create_time desc limit ?, ?
```
##### 列表和分页查询
###### 查询订单列表：
``` java
		orderDao.query().filter(Restrictions.gt("price", 50)).sort(JpaSort.desc("createTime")).selectThis()
				.setTransformer(Transformers.asBean(OrderVO.class, null, (model, order, output) -> {
					Product product = order.getProduct();
					output.setProductName(product.getName());
					output.setOrigin(product.getOrigin());
					User user = order.getUser();
					output.setUsername(user.getName());
					output.setPhone(user.getPhone());
				})).list(50).forEach(vo -> {
					System.out.println(vo);
				});
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where order0_.price>50.0 order by order0_.create_time desc limit ?
```

###### 分页查询订单：
``` java
		PageResponse<OrderVO> pageResponse = orderDao.select().filter(Restrictions.gt("price", 50)).sort(JpaSort.desc("createTime"))
				.selectThis().setTransformer(Transformers.asBean(OrderVO.class, null, (model, order, output) -> {
					Product product = order.getProduct();
					output.setProductName(product.getName());
					output.setOrigin(product.getOrigin());
					User user = order.getUser();
					output.setUsername(user.getName());
					output.setPhone(user.getPhone());
				})).list(PageRequest.of(1, 10));
		for (PageResponse<OrderVO> current : pageResponse) {
			System.out.println("第" + current.getPageNumber() + "页");
			for (OrderVO vo : current.getContent()) {
				System.out.println(vo);
			}
		}
// Hibernate: select count(1) as col_0_0_ from demo_order order0_ where order0_.price>50.0
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where order0_.price>50.0 order by order0_.create_time desc limit ?, ?
```
###### 分页查询会出现两条sql, 一条count语句，一条查询语句

##### 删除操作
``` java
int rows = productDao.delete().filter(Restrictions.gt("price", 990)).execute();
System.out.println("Effected rows: " + rows);
// Hibernate: delete from demo_product where price>990.0
```
###### 子查询删除: 
``` java
		JpaSubQuery<Order, Order> subQuery = productDao.delete().subQuery(Order.class);
		subQuery.select("product");
		int rows = productDao.delete().filter(Restrictions.in("id", subQuery).not()).execute();
		System.out.println("Effected rows: " + rows);
// Hibernate: delete from demo_product where id not in  (select order1_.product_id from demo_order order1_)
```
##### 更新操作
``` java
int rows = userDao.update().set("vip", true).filter(Restrictions.eq("vip", false)).execute();
System.out.println("Effected rows: " + rows);
// Hibernate: update demo_user set vip=? where vip=?
```
###### 子查询更新:
``` java
		JpaSubQuery<Order, Order> subQuery = userDao.update().subQuery(Order.class).filter(Restrictions.gte("price", 500)).select("user");
		int rows = userDao.update().set("vip", true).filter(Restrictions.in("id", subQuery)).execute();
		System.out.println("Effected rows: " + rows);
// update demo_user set vip=? where id in (select order1_.user_id from demo_order order1_ where order1_.price>=500.0)
```
### 本地查询示例
---------------------------------
对于fastjpa操作JPA对象难以满足实际业务的场景，还是建议直接使用sql, 即本地查询
###### 简单查询返回关联实体
``` java
ResultSetSlice<Order> resultSetSlice = orderDao.select("select * from demo_order where price>?", new Object[] { 50 });
		PageResponse<Order> pageResponse = resultSetSlice.list(PageRequest.of(1, 10));
		for (PageResponse<Order> current : pageResponse) {
			System.out.println("第" + current.getPageNumber() + "页");
			for (Order order : current.getContent()) {
				System.out.println("Order Id: "+order.getId() + ", Product Name: " + order.getProduct().getName() + ", Username: " + order.getUser().getName());
			}
		}
```
###### 分组查询及分页
``` java
		ResultSetSlice<Map<String, Object>> resultSetSlice = orderDao.selectForMap(
				"select origin,max(price) as maxPrice,min(price) as minPrice,avg(price) as avgPrice from demo_product group by origin",
				new Object[0]);
		PageResponse<Map<String, Object>> pageResponse = resultSetSlice.list(PageRequest.of(1, 5));
		for (PageResponse<Map<String, Object>> current : pageResponse) {
			System.out.println("第" + current.getPageNumber() + "页");
			for (Map<String, Object> vo : current.getContent()) {
				System.out.println(vo);
			}
		}
```


事实上，fastjpa是为了提升开发人员使用JPA的效率而出现的，如果出现使用fastjpa不能满足业务需求的情况，请果断使用本地sql

最后附上启用JPA配置的核心代码, 可以直接食用
``` java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(repositoryFactoryBeanClass = EntityDaoFactoryBean.class, basePackages = { "com.demo.jpalearning.dao" })
public class JpaConfig {

	public static final String PRIMARY_ENTITY_FACTORY_BEAN_NAME = "entityManagerFactory";

	@Primary
	@Bean(PRIMARY_ENTITY_FACTORY_BEAN_NAME)
	public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource, EntityManagerFactoryBuilder builder,
			JpaProperties jpaProperties, HibernateProperties hibernateProperties) {
		Map<String, Object> properties = hibernateProperties.determineHibernateProperties(jpaProperties.getProperties(),
				new HibernateSettings());
		return builder.dataSource(dataSource).properties(properties).packages("com.demo.jpalearning.entity").build();
	}

	@Bean
	public PlatformTransactionManager transactionManager(EntityManagerFactory emf) {
		JpaTransactionManager transactionManager = new JpaTransactionManager();
		transactionManager.setEntityManagerFactory(emf);
		return transactionManager;
	}

	@Bean
	public PersistenceExceptionTranslationPostProcessor exceptionTranslation() {
		return new PersistenceExceptionTranslationPostProcessor();
	}

}
```
别忘了配置文件添加：
``` properties
#Jpa Configuration
spring.jpa.database=MYSQL
spring.jpa.show-sql=true
spring.jpa.format-sql=false
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming.physical-strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```
源码地址：https://github.com/paganini2008/fastjpa-spring-boot-starter.git

