---
title: 'Elasticsearch - Incompatible type'
date: 2018-01-07 16:43:00 +0100
categories: programming
---

# What is Elasticsearch?

[Elasticsearch](https://www.elastic.co/products/elasticsearch) is a search and analitycs engine. It allows you to store data and search through it with ease, but it is not a database engine per se. It doesn't guaranties [ACID](https://en.wikipedia.org/wiki/ACID). Elasticsearch stores data in the index. It is special construction that fastens search queries, but it is vulnerable for corruption. It's not a problem since all your data should be stored in proper database engine like [MySQL](https://www.mysql.com/) or [MongoDB](https://www.mongodb.com/), so index can be rebuilt from it.

# Incompatible type

Elasticsearch is RESTful and uses JSON to transfer data. It uses [dynamic field mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/dynamic-field-mapping.html) to distinguish types of fields. Let's check an example of JSON, which causes an incompatible type error.

```json
{
    "positions": [
        {
            "id": 1,
            "name": "Apple",
            "quantity": 2,
            "unit": "pcs"
        },
        {
            "id": 2,
            "name": "Water",
            "quantity": 1.25,
            "unit": "litre"
        }
    ]
}
```
The problem is with field quantity. If Elasticsearch recognizes field as Long and then the same field has fraction then error occures, cause you can't store Double as Long. This problem also happens when Double is recongized first and then came Long.

There is a quite simple solution for this problem. We need to [define types](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html#_explicit_mappings) on our own. To demonstrate it, I will assemble small project using [Spring Data Elasticsearch](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/).

# Project

Foremost, I go to [Spring Initializr](https://start.spring.io/). I selected: Elasticsearch, DevTools and [Lombok](https://projectlombok.org/features/all). 
All we really need is Elasticsearch, but DevTools and Lombok does our work easier.
To bootstrap our project, we need Elasticsearch engine running. Luckily, it is already build-in feature in our project. First we need to add following annotation to main class. 

```java
@EnableElasticsearchRepositories(basePackages = "com.github.wpanas.elasticsearchdemo.repository")
```

The last thing we are missing is `ElasticsearchTemplate` class. Below is a bean that creates its local instance.

```java
@Bean
public ElasticsearchOperations elasticsearchTemplate() {
    return new ElasticsearchTemplate(nodeBuilder().local(true).node().client());
}
```

This configuration launches a local Elasticsearch instance everytime we run our project that stores indexes in the folder `data` in the project root. Now let's create documents that we will use to demonstrate the previously described problem. First we make documents that causes an error.

```java
@Data
@Document(indexName = "invoice")
public class Invoice {
    @Id
    private String id;
    private String number;
    private LocalDate saleDate;
    private Set<InvoicePosition> invoicePositions;
}
```

```java
@Data
@Document(indexName = "invoice-positions")
public class InvoicePosition {
    @Id
    private String id;
    private String name;
    private BigDecimal price;
    private BigDecimal quantity;
    private String unit;
}
```

We can solve an invalid type recognition by usage of the annotation `@Field`. Below I present you fixed documents.

```java
@Data
@Document(indexName = "proper-invoice")
public class ProperInvoice {
    @Id
    @Field(type = FieldType.String)
    private String id;
    @Field(type = FieldType.String)
    private String number;
    @Field(type = FieldType.Object)
    private LocalDate saleDate;
    @Field(type = FieldType.Nested)
    private Set<ProperInvoicePosition> invoicePositions;
}
```

```java
@Data
@Document(indexName = "proper-invoice-positions")
public class ProperInvoicePosition {
    @Id
    @Field(type = FieldType.String)
    private String id;
    @Field(type = FieldType.String)
    private String name;
    @Field(type = FieldType.Double)
    private BigDecimal price;
    @Field(type = FieldType.Double)
    private BigDecimal quantity;
    @Field(type = FieldType.String)
    private String unit;
}
```
The most important changes are:
-  the annotation `@Field(type = FieldType.Nested)` by property `invoicePostions` in class `PorperInvoice` 
- annotations `@Field(type = FieldType.Double)` in class `ProperInvoicePositions`. 

These changes are self-explanatory, but don't forget to used them both. Without `FieldType.Nested` your changes in `ProperInvoicePostion` class will not matter when you will save `ProperInvoice`.

`@Field(type = FieldType.String)` is not necessary in this example, but I included it to show, how to use this annotation. 

You might notice that I used `@Field(type = FieldType.Object)` with property `saleDate` that is type `LocalDate`. Of course, I could use `FieldType.Date`, but it doesn't works out of the box and for an explanation for this problem, I will make an another article.

We defined types by ourself, but Elasticsearch doesn't know about it. To create indexes with our types we need to do one more action.

```java
elasticsearchTemplate.createIndex(ProperInvoice.class);
elasticsearchTemplate.createIndex(ProperInvoicePosition.class);
```

# Summary

Elasticsearch is a great tool to accelerate search queries in your application. It uses the dynamic type recognition, and it might be a problem with numbers. To avoid this problem, we can define types ourself. This can be achieved by a usage of annotation `@Field` and creating index before first call to Elasticsearch.

The project that I created for this article is available on [GitHub](https://github.com/wpanas/code-snippets/tree/master/elasticsearch-demo).
Check it out and launch tests to find out how it works.
