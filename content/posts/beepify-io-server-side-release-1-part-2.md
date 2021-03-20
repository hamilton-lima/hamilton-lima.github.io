---
title: 'beepify.io, server side release 1 - part 2'
date: Mon, 17 Oct 2016 19:58:24 +0000
draft: false
tags: ['beepify.io', 'database', 'Ebean', 'infiniteloop', 'jsonignore', 'manytoone', 'Model', 'onetomany', 'playframework', 'selfhackaton']
---

The second part of the server side release 1 will describe some details about the implementation it self. please [see the first part](https://hamiltonlima.com/blog/beepify-io-server-side-release-1-part1/) for configuration details.

### Quick tips to a playframework application

*   Of course there are tons of nice features that can be used, but for the sake of simplicity here are the concepts that I consider the most important ones for this implementation:
*   Edit the file **conf/routes** to match the services urls and the implementation of it, as simple as: **GET /mock controllers.MockDataController.add()**, that will publish the url /mock at your application, and when this address is call the method add() from the class MockDataController will be called, and that class is implemented at the folder app/controllers and extends Controller
*   Create your model classes, as [POJO](https://pt.wikipedia.org/wiki/Plain_Old_Java_Objects) files, make life simple all the attributes can be public, you just need to add the @Entity notation, extend Model class and add @Id notation as well, that will generate the necessary table in the database to store your objects
*   Use [Ebean](https://ebean-orm.github.io/) to read data from the database, most of the time is very simple and straightforward, for example an simple find by barcode on the Products is written like this:
```
     new Model.Finder(Product.class).where().eq("barcode", barcode).findUnique();
```

### Declaring relationship between classes

This is the tables in use for this application ![beepify-database](/images/2016/10/beepify-database.png) Note that we have some relationships: from Product to Prices, from Prices to Product and from Prices to Store, let´s see how this represented in the Java code. For reference here are the complete java classes :

*   [Price.java](/images/2016/10/Price.java_.txt)
*   [Product.java](/images/2016/10/Product.java_.txt)
*   [Store.java](/images/2016/10/Store.java_.txt)

At the Product class the relationship with the list of Price is represented by the annotation @OneToMany that indicates the attribute "product" as the way to map Price to one Product
```
   @OneToMany(mappedBy="product")
     public List<Price> prices;
```
At the Price class we have counterpart of the relationship represented with the annotation 
@ManyToOne

```
@ManyToOne
 @JsonIgnore
 public Product product;
```

> Note that the attribute "product" also has an annotation @JsonIgnore, that will avoid an infinite recursion when trying to generate an JSON to represent a product. without this instruction the JSON parser would try to read the prices list of the product that is in the price, that would result in a infinite loop (been there done that)

The infinite loop error :) ![2016-10-16_2006](/images/2016/10/2016-10-16_2006.png) The Controllers On this implementation we have the following urls/controllers GET /products controllers.ProductController.getAll() GET /product/:barcode controllers.ProductController.findByBarcode(barcode: String) GET /mock controllers.MockDataController.add() /products as the name implies read all the existing products, this of course is only usable for tests, imagine a service returning 100k products... /product/:barcode is an url that accepts one parameter and that parameter will be passed to the method findByBarcode( ), so its possible to make calls like: /product/00000 where 00000 is the barcode of the product to be search. /mock is a service to create the test data used in this implementation. Here are the full source code for each controller, the itself is very simple and straight forward, for example a way to store each Model to the database is calling the method: save(), simple isn´t it ? ;)

*   [ProductController.java](/images/2016/10/ProductController.java_.txt)
*   [MockDataController.java](/images/2016/10/MockDataController.java_.txt)