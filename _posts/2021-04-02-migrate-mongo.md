---
title: "How to migrate MongoDB?"
---

MongoDB is advertised as ["schemaless"] and it's true. Until
you need to make a breaking change to documents stored 
in your collection.

In that situation you realize that using schemaless database 
doesn't free from migrating data when it's structure changes.

How to do it in most efficient fashion? Let's say you have
tens of millions of documents in your collection. At that point this question changes to: *how to do it without technical break?*

I will proceed with a short story that adds context
to a purpose of migrating data. If you don't need it, just go to
section: *How to migrate documents in MongoDB collection?*.

# When do you need to migrate documents?

Let's say that you are a developer in an e-commerce company.
Your application stores bank accounts of merchants. One day
your product owner comes and asks you to allow merchants to add
multiple accounts for different currencies. Up until now your
documents in your MongoDB collection have looked like this:

```json
{
    "_id": "1b85af01-0ede-496a-927d-bb27a8e2bded",
    "bankAccount": "US77602308983748679732663455"
}
```

Based on experience you figured out that most of the merchants
will have from one to three different accounts. Some of the merchants
will have up to ten accounts. That data will fit perfectly into one
document & most of the time it will be fetched together.

[The attribute pattern] is the best fit for purpose of this change.
You created a new document schema.

```json
{
    "_id": "1b85af01-0ede-496a-927d-bb27a8e2bded",
    "bankAccounts": [
        {
            "currency": "USD",
            "number": "US77602308983748679732663455"
        },
        {
            "currency": "EUR",
            "number": "DE05842479609094250496642135"
        }
    ]
}
```

# How to migrate documents in MongoDB collection?

To migrate old documents to new documents, you need to do following
operation:

```kotlin
fun migrate(oldCustomer: OldCustomer): NewCustomer =
    NewCustomer(
        id=oldCustomer.id,
        bankAccounts=setOf(
            BankAccount(DEFAULT_CURRENCY, oldCustomer.bankAccount)
        )
    )
```

It is pretty simple, but doing it on tens or hundreds of millions 
of documents might be tricky. You might think of following problems to solve:

- How to migrate a collection without a technical break?
- How to keep track on not migrated documents?
- How to handle both document schemas until everything is migrated?
- How to migrate documents for the most active merchants first?

For all of these questions, the answer is the Schema Versioning pattern.

First step is to add a field `schemaVersion` when you store new documents for new merchants.
Old documents don't need to be changed yet. No `schemaVersion` value 
is what we need in them.

```json
{
    "_id": "1b85af01-0ede-496a-927d-bb27a8e2bded",
    "bankAccounts": [
        {
            "currency": "USD",
            "number": "US77602308983748679732663455"
        },
        {
            "currency": "EUR",
            "number": "DE05842479609094250496642135"
        }
    ],
    "schemaVersion": 2 // the constant value for all new documents
}
```

Then you can migrate old documents in two ways. Perfectly, you
can do both at the same time.

# How to migrate documents on the fly?

Most of the applications fetch documents, modify them and then save
changes. If your application acts like this, then you can migrate
documents while fetching them. Just add a deserializer that
according to `schemaVersion` value does proper migration.

You can find more detailed example in this [GitHub repository]. I used with [Kotlin], [Spring Boot] & [Testcontainers].

# How to migrate documents in the background?

Start with adding an index to field `schemaVersion`. If you use MongoDB older than 4.2, remember
to add the index in the [background]! 

Then iterate through documents in collection using the new index.
This way background operation will be efficient & you will be
sure that all documents are migrated.

["schemaless"]: https://www.mongodb.com/unstructured-data/schemaless
[GitHub repository]: https://github.com/wpanas/migrate-mongo
[The attribute pattern]: https://www.mongodb.com/blog/post/building-with-patterns-the-attribute-pattern
[background]: https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#options-for-all-index-types
[GitHub repository]: https://example.com
[Kotlin]: https://kotlinlang.org/
[Spring Boot]: https://spring.io/projects/spring-boot
[Testcontainers]: https://www.testcontainers.org/