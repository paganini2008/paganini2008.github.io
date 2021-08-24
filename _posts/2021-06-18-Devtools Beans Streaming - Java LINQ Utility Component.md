---
layout: post
title: Devtools Beans Streaming - A practical Java LINQ Tool
date: 2021-06-18 20:00:00.000000000 +09:00
---

Devtools Beans Streaming is a solution for common querying or aggregative querying Java Object List. It provides functions similar to LINQ in C sharp

### Install：
---------------------------
``` xml
<dependency>
	<groupId>com.github.paganini2008</groupId>
	<artifactId>devtools-beans-streaming</artifactId>
	<version>2.0.3</version>
</dependency>
```
### Compatibility：
---------------------------
jdk1.8+ (or later)

### Quick Start:
-------------------------
**Step1:** Define a Simple Java Object 
*Product.java*

``` java
@Getter
@Setter
@ToString
public class Product {

	private int id;
	private String name;
	private String location;
	private Date created;
	private Date expired;
	private Float price;
	private BigInteger sales;
	private boolean export;
	private Long number;
	private BigDecimal freight;
	private Style style;
	private Salesman salesman;

	public static enum Style {
		HARD, SOFT;
	}

	@Getter
	@Setter
	@ToString
	public static class Salesman {

		private String name;
		private String password;

		public Salesman(String name, String password) {
			this.name = name;
			this.password = password;
		}

	}

}
```

**Step2:** Define a list, generate a batch of objects, and assign values randomly

``` java
private final List<Product> products = new ArrayList<Product>();

	@Before
	public void testBefore() {
		final String[] users = new String[] { "Petter", "Jack", "Tony" };
		final String[] locations = new String[] { "London", "NewYork", "Tokyo", "HongKong", "Paris" };
		for (int i = 0; i < 10000; i++) {
			Product product = new Product();
			product.setCreated(DateUtils.valueOf(2020, RandomUtils.randomInt(1, 12), RandomUtils.randomInt(1, 28)));
			product.setExpired(DateUtils.addDays(product.getCreated(), 90));
			product.setExport(RandomUtils.randomBoolean());
			product.setFreight(BigDecimal.valueOf(RandomUtils.randomFloat(10, 100)).setScale(2, RoundingMode.HALF_UP));
			product.setId(i + 1);
			product.setLocation(RandomUtils.randomChoice(locations));
			product.setName("Product-" + product.getId());
			product.setNumber(RandomUtils.randomLong(1, 1000));
			product.setPrice(RandomUtils.randomFloat(10, 1000, 2));
			product.setSales(BigInteger.valueOf(RandomUtils.randomLong(10000, 100000)));
			product.setStyle(Style.values()[RandomUtils.randomInt(0, 2)]);
			product.setSalesman(new Product.Salesman(RandomUtils.randomChoice(users), "123456"));
			products.add(product);
		}
	}
```

Here are some examples: 

**Example 1:**

``` java
Predicate<Product> predicate = Restrictions.eq("location", "London");
Selector.from(products).filter(predicate).list().forEach(product -> {
	System.out.println(product);
});
// Equivalent to: select * from Product where location='London'
```
**Example 2:**
``` java
Predicate<Product> predicate = Restrictions.lte("created", new Date());
predicate = predicate.and(Restrictions.eq("salesman.name", "Petter"));
Selector.from(products).filter(predicate).list().forEach(product -> {
	System.out.println(product);
});
// select * from Product where created<= now() and salesman.name='Petter'
```
**Example 3:**

``` java
        Selector.from(products).groupBy("location", String.class).setTransformer(new View<Product>() {
			protected void setAttributes(Tuple tuple, Group<Product> group) {
				tuple.set("maxPrice", group.max("price", Float.class));
				tuple.set("minPrice", group.min("price", Float.class));
				tuple.set("avgFreight", group.avg("freight"));
				tuple.set("sumSales", group.sum("sales"));
			}
		}).list().forEach(tuple -> {
			System.out.println(tuple);
		});
// Equivalent to: select location,max(price) as maxPrice, min(price) as minPrice,avg(freight) as avgFreight,sum(sales) as sumSales from Product group by location
```
**Example 4:**

``` java
Selector.from(products).groupBy("location", String.class)
                       .groupBy("style",Product.Style.class).having(group -> {
	                         return group.avg("freight").compareTo(BigDecimal.valueOf(55)) > 0;
                        }).setTransformer(new View<Product>() {
                            protected void setAttributes(Tuple tuple, Group<Product> group) {
                                    tuple.set("maxPrice", group.max("price", Float.class));
                                    tuple.set("minPrice", group.min("price", Float.class));
                                    tuple.set("avgFreight", group.avg("freight"));
                                    tuple.set("sumSales", group.sum("sales"));
                                }
                        }).list().forEach(tuple -> {
                            System.out.println(tuple);
                        });
// Equivalent to: select location,style,max(price) as maxPrice, min(price) as minPrice,avg(freight) as avgFreight,sum(sales) as sumSales from Product group by location,style having avg(freight) > 55
```
**Example 5:**

``` java
Sorter<Product> sorter = Orders.descending("price", BigDecimal.class);		Selector.from(products).orderBy(sorter).list(100).forEach(product -> {
	System.out.println("Name: " + product.getName() + ", Price: " + product.getPrice() + ", Freight: " +                        product.getFreight());
});
// Equivalent to: select name,price from Product order by price desc limit 100
```

Git repository：https://github.com/paganini2008/devtools.git

