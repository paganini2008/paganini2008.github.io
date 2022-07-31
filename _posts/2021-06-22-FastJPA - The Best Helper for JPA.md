---
layout: post
title: FastJPA - The Best Helper for JPA
date: 2021-06-22 08:30:00.000000000 +09:00
---

A very cool JPA developping tookit, which provides an API  to use  JPA core class with a  fluent style. It aims to promote to use JPA in  object-oriented way rather than to  write JQL or HQL in your code. In fact, <code>fastjpa-spring-boot-starter</code> is  rather easier than <code>spring-boot-starter-data-jpa</code> in API's complexity

## Install

``` xml
<dependency>
  <groupId>com.github.paganini2008.springworld</groupId>
  <artifactId>fastjpa-spring-boot-starter</artifactId>
  <version>2.0.1</version>
</dependency>

```

## Compatibility

* Jdk1.8 (or later)
* <code>SpringBoot</code> Framework 2.2.x  (or later)
* Hibernate 5.x (or later)


## Core API

- EntityDao
- Model
- JpaQuery
- JpaPage
- Filter
- Column
- Field
- JpaGroupBy
- JpaSort
- JpaPageResultSet
- JpaQueryResultSet
- JpaUpdate
- JpaDelete


## Quick Start

#### Entity Class

<code>User.java</code>

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

<code>Product.java</code>

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

<code>Order.java</code>

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

#### Repository Class

<code>UserDao.java</code>

``` java
public interface UserDao extends EntityDao<User, Long> {
}
```

<code>OrderDao.java</code>

``` java
public interface OrderDao extends EntityDao<Order, Long> {
}
```

<code>ProductDao.java</code>

``` java
public interface ProductDao extends EntityDao<Product, Long> {
}
```



**EntityDao Source Code**

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


#### Filter
Equivalent to where condition in select statement
* Comparison operators supported: lt, lte, gt, gte, eq, ne, like, in, between, isNull, notNull
* Logical operators supported: and, or, not

**Example 1:** 
``` java
LogicalFilter filter = Restrictions.between("price", 10, 50);
filter = Restrictions.in("price", Arrays.asList(10,20,30,40,50));
filter = Restrictions.like("name", "%cat%");
filter = Restrictions.eq("orignal", "Shanghai");
```
**Example 2:**

``` java
LogicalFilter filter = Restrictions.gt("price", 50); 
productDao.query().filter(filter).selectThis().list().forEach(pro -> {
	System.out.println(pro);
});
// Hibernate: select product0_.id as id1_1_, product0_.name as name2_1_, product0_.origin as origin3_1_, product0_.price as price4_1_ from demo_product product0_ where product0_.price>50.0
```
**Example 3:**
``` java
LogicalFilter filter = Restrictions.between("price", 10, 50);
filter = filter.and(Restrictions.like("name", "%cat%"));
filter = filter.and(Restrictions.eq("orignal", "Shanghai"));
// Equivalent to sql: where price between (10,50) and name like '%cat%' and orignal='Shanghai'
productDao.query().filter(filter).selectThis().list().forEach(pro -> {
	System.out.println(pro);
});
```
**Example 4:**
``` java
LogicalFilter filter = Restrictions.eq("orignal", "Shanghai");
filter = filter.or(Restrictions.eq("orignal", "New York"));
// Equivalent to sql: where orignal='Shanghai' or orignal='New York'
productDao.query().filter(filter).selectThis().list().forEach(pro -> {
	System.out.println(pro);
});
```
**Example 5:**
``` java
LogicalFilter filter = Restrictions.eq("orignal", "Shanghai");
filter = filter.and(Restrictions.eq("orignal", "New York"));
filter = filter.not();
 // Equivalent to sql: where orignal!='Shanghai' and orignal!='New York'
productDao.query().filter(filter).selectThis().list().forEach(pro -> {
	System.out.println(pro);
});
```

#### JpaGroupBy
Equivalent to group by in select statement
**Example 6:**
``` java
productDao.multiquery().groupBy("origin").select(Column.forName("origin"), Fields.count(Fields.toInteger(1)).as("count")).list()
.forEach(t -> {
		System.out.println("origin: "+t.get("origin") + "\tcount: " + t.get("count"));
});
// Hibernate: select product0_.origin as col_0_0_, count(1) as col_1_0_ from demo_product product0_ group by product0_.origin

```
#### Column
Represent one column
**Example 7:**
```java
productDao.multiquery().select(Column.forName("name"), Column.forName("price")).list(10).forEach(t -> {
        System.out.println("name: "+t.get("name") + "\t price: " + t.get("price"));
});
// Hibernate: select product0_.name as col_0_0_, product0_.price as col_1_0_ from demo_product product0_ limit ?
```
**Example 8:**
Query multiple columns like this way：
``` java
productDao.multiquery().select(new String[] { "name", "price" }).list(10).forEach(t -> {
	System.out.println("name: " + t.get("name") + "\t price: " + t.get("price"));
});
```
Or like this way：
**Example 9:**
``` java
ColumnList columnList = new ColumnList().addColumn("name").addColumn("price");
productDao.multiquery().select(columnList).list(10).forEach(t -> {
	System.out.println("name: " + t.get("name") + "\t price: " + t.get("price"));
});
```
#### Field
Represent aggregation function or other function, or just a number

##### Aggregation Function:

**Example 10:**
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
###### Other Function: 

**concat**

**Example 10:**
``` java
ColumnList columnList = new ColumnList()
	.addColumn(Fields.concat(Fields.concat(Fields.max("price", String.class), "/"), Fields.min("price", String.class)), "repr")
	.addColumn("origin");
productDao.multiquery().groupBy("origin").select(columnList).setTransformer(Transformers.asBean(ProductVO.class)).list().forEach(vo -> {
	System.out.println(vo);
});
// Hibernate: select concat(concat(max(cast(product0_.price as char)), '/'), min(cast(product0_.price as char))) as col_0_0_, product0_.origin as col_1_0_ from demo_product product0_ group by product0_.origin
```

**lower and upper**

**Example 11:**
``` java
ColumnList columnList = new ColumnList()
       .addColumn(Function.build("LOWER", String.class, "name"), "name")
       .addColumn(Function.build("UPPER", String.class, "origin"), "origin");
productDao.multiquery().select(columnList).list(10).forEach(t -> {
	System.out.println("name: " + t.get("name") + "\t origin: " + t.get("origin"));
});
// Hibernate: select lower(product0_.name) as col_0_0_, upper(product0_.origin) as col_1_0_ from demo_product product0_ limit ?
```

**case when**

**Example 12:**
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

#### Sub Query

**Example 13:**

``` java
		JpaSubQuery<Order,Order> subQuery = orderDao.query().subQuery(Order.class, "o").filter(Restrictions.eq("o", "id", "100")).select("o", "product.id");
		Product product = productDao.query().filter(Restrictions.eq("id", subQuery)).selectThis().first();
		System.out.println(product);
// Hibernate: select product0_.id as id1_1_, product0_.name as name2_1_, product0_.origin as origin3_1_, product0_.price as price4_1_ from demo_product product0_ where product0_.id=(select order1_.product_id from demo_order order1_ where order1_.id=100) limit ?
```
**Example 14:**

``` java
		JpaQuery<Order,Order> jpaQuery = orderDao.query();
		JpaSubQuery<Product, BigDecimal> subQuery = jpaQuery.subQuery(Product.class, "p", BigDecimal.class)
				.select(Fields.avg(Property.forName("p", "price")));
		jpaQuery.filter(Restrictions.gte("price", subQuery)).selectThis().list(10).forEach(pro -> {
			System.out.println(pro);
		});
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where order0_.price>=(select avg(product1_.price) from demo_product product1_) limit ?
```
**Example 15:**

``` java
		JpaQuery<Order,Order> jpaQuery = orderDao.query();
		JpaSubQuery<Product, BigDecimal> subQuery = jpaQuery.subQuery(Product.class, "p", BigDecimal.class)
				.select(Fields.avg(Property.forName("p", "price")));
		jpaQuery.filter(Restrictions.gte("price", subQuery)).selectThis().list(10).forEach(pro -> {
			System.out.println(pro);
		});
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where 1=1 and (exists (select user1_.id from demo_user user1_ where user1_.name=order0_.receiver)) limit ?

```

#### Order

**Example 16:**

``` java
orderDao.query().filter(Restrictions.gte("price", 50)).sort(JpaSort.desc("createTime"), JpaSort.asc("price"))
                .selectThis().list(10)
				.forEach(pro -> {
					System.out.println(pro);
				});
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where order0_.price>=50.0 order by order0_.create_time desc, order0_.price asc limit ?
```

#### Multiple Table Join Query

**Example 17:**

``` java
PageResponse<Tuple> pageResponse = orderDao.multiselect().leftJoin("product", "p")
				.filter(Restrictions.gte("p", "price", 50))
                .sort(JpaSort.desc("createTime")).selectAlias("p")
				.list(PageRequest.of(10));
		for (PageResponse<Tuple> current : pageResponse) {
			System.out.println("Page: " + current.getPageNumber());
			for (Tuple tuple : current.getContent()) {
				System.out.println(Arrays.toString(tuple.toArray()));
			}
		}
// Hibernate: select product1_.id as id1_1_, product1_.name as name2_1_, product1_.origin as origin3_1_, product1_.price as price4_1_ from demo_order order0_ left outer join demo_product product1_ on order0_.product_id=product1_.id where product1_.price>=50.0 order by order0_.create_time desc limit ?, ?
```
**Example 18:**

``` java
		ColumnList columnList = new ColumnList();
		columnList.addColumn("id");
		columnList.addColumn("p","name");
		columnList.addColumn(Property.forName("p", "price"),"originalPrice");
		columnList.addColumn("price");
		columnList.addColumn("createTime");
		PageResponse<Tuple> pageResponse = orderDao.multiselect().join("product", "p")
				.filter(Restrictions.gte("p", "price", 50))
                .sort(JpaSort.desc("createTime")).select(columnList)
				.list(PageRequest.of(10));
		for (PageResponse<Tuple> current : pageResponse) {
			System.out.println("Page: " + current.getPageNumber());
			for (Tuple tuple : current.getContent()) {
				System.out.println(Arrays.toString(tuple.toArray()));
			}
		}
// Hibernate: select order0_.id as col_0_0_, product1_.name as col_1_0_, product1_.price as col_2_0_, order0_.price as col_3_0_, order0_.create_time as col_4_0_ from demo_order order0_ inner join demo_product product1_ on order0_.product_id=product1_.id where product1_.price>=50.0 order by order0_.create_time desc limit ?, ?
```

#### List Query and Paging Query
##### List Query:
**Example 19:**

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
##### Paging Query：

**Example 20:**

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
			System.out.println("PageNumber: " + current.getPageNumber());
			for (OrderVO vo : current.getContent()) {
				System.out.println(vo);
			}
		}
// Hibernate: select count(1) as col_0_0_ from demo_order order0_ where order0_.price>50.0
// Hibernate: select order0_.id as id1_0_, order0_.create_time as create_t2_0_, order0_.discount as discount3_0_, order0_.price as price4_0_, order0_.product_id as product_6_0_, order0_.receiver as receiver5_0_, order0_.user_id as user_id7_0_ from demo_order order0_ where order0_.price>50.0 order by order0_.create_time desc limit ?, ?
```
There will be two sqls shown in console by select method when you do paging query, one is count select statement, another is select statement

#### Delete Operation

**Example 21:**
``` java
int rows = productDao.delete().filter(Restrictions.gt("price", 990)).execute();
System.out.println("Effected rows: " + rows);
// Hibernate: delete from demo_product where price>990.0
```
##### Delete Operation With SubQuery:

**Example 22:**
``` java
		JpaSubQuery<Order, Order> subQuery = productDao.delete().subQuery(Order.class);
		subQuery.select("product");
		int rows = productDao.delete().filter(Restrictions.in("id", subQuery).not()).execute();
		System.out.println("Effected rows: " + rows);
// Hibernate: delete from demo_product where id not in  (select order1_.product_id from demo_order order1_)
```
#### Update Operation

**Example 23:**

``` java
int rows = userDao.update().set("vip", true).filter(Restrictions.eq("vip", false)).execute();
System.out.println("Effected rows: " + rows);
// Hibernate: update demo_user set vip=? where vip=?
```
##### Update Operation With SubQuery:

**Example 24:**
``` java
		JpaSubQuery<Order, Order> subQuery = userDao.update().subQuery(Order.class).filter(Restrictions.gte("price", 500)).select("user");
		int rows = userDao.update().set("vip", true).filter(Restrictions.in("id", subQuery)).execute();
		System.out.println("Effected rows: " + rows);
// update demo_user set vip=? where id in (select order1_.user_id from demo_order order1_ where order1_.price>=500.0)
```
#### Native Query

Notice: If there are some scene that FastJPA cannot meet, you'd better use native SQL directly to query 

**Example 25:**
``` java
ResultSetSlice<Order> resultSetSlice = orderDao.select("select * from demo_order where price>?", new Object[] { 50 });
		PageResponse<Order> pageResponse = resultSetSlice.list(PageRequest.of(1, 10));
		for (PageResponse<Order> current : pageResponse) {
			System.out.println("Page: " + current.getPageNumber());
			for (Order order : current.getContent()) {
				System.out.println("Order Id: "+order.getId() + ", Product Name: " + order.getProduct().getName() + ", Username: " + order.getUser().getName());
			}
		}
```

**Example 26:**

``` java
		ResultSetSlice<Map<String, Object>> resultSetSlice = orderDao.selectForMap(
				"select origin,max(price) as maxPrice,min(price) as minPrice,avg(price) as avgPrice from demo_product group by origin",
				new Object[0]);
		PageResponse<Map<String, Object>> pageResponse = resultSetSlice.list(PageRequest.of(1, 5));
		for (PageResponse<Map<String, Object>> current : pageResponse) {
			System.out.println("Page: " + current.getPageNumber());
			for (Map<String, Object> vo : current.getContent()) {
				System.out.println(vo);
			}
		}
```



JPA Configuration Code:
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

JPA configuration:

<code>application.properties</code>

``` properties
#Jpa Configuration
spring.jpa.database=MYSQL
spring.jpa.show-sql=true
spring.jpa.format-sql=false
spring.jpa.hibernate.ddl-auto=update
spring.jpa.hibernate.naming.physical-strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```

Git repository:  

https://github.com/paganini2008/fastjpa-spring-boot-starter.git

