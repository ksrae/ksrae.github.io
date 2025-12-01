---
title: "Differenct between update and replace"
date: 2019-07-29 12:26:00 +0900
comments: true
categories: mongodb
tags: [update, replace]
---

[한국어(Korean) Page](https://velog.io/@ksrae/MongoDB%EC%9D%98-update%EC%99%80-replace%EC%9D%98-%EC%B0%A8%EC%9D%B4)
<br/>

Let's compare `update` and `replace` operations in MongoDB. This explanation is based on a review of resources.

### Update Operations

```jsx
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)

db.collection.updateOne(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)

db.collection.updateMany(
   <filter>,
   <update>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```

The `update` functionality in MongoDB offers three distinct methods: `update`, `updateOne`, and `updateMany`. These methods primarily serve to modify existing data within a collection. Notably, the `update` method includes a `multi` option, which determines whether to update multiple documents or just one. The `updateOne` and `updateMany` methods implicitly handle single and multiple document updates, respectively. Therefore, the `update` method, when combined with the `multi` option, can effectively replicate the behavior of both `updateOne` and `updateMany`.

`update` operations function similarly to standard database update commands. They modify specific fields within a document. The `upsert` option allows for the insertion of a new document if no matching document is found based on the filter criteria.

Consider the following JSON document:

```json
{
  "id": 12345,
  "name": "test",
  "age": 1
}
```

To update the `age` field to `10`, you would use the `$set` operator:

```jsx
db.collection.update( {id: 12345}, {$set: {age: 10}}, {multi: false})
```

This command sets the `age` field of the document with `id: 12345` to `10`, resulting in the following document:

```json
{
  "id": 12345,
  "name": "test",
  "age": 10
}
```

Within the filter, operators like `$gte` (greater than or equal to) and `$lte` (less than or equal to) can be utilized, especially when combined with `{multi: true}` for updating multiple documents. Additionally, the `$unset` operator can be used to remove a field from a document.

### Replace Operations

Starting from version 3.2, MongoDB introduced the `replaceOne` method.

```jsx
db.collection.replaceOne(
   <filter>,
   <replacement>,
   {
     upsert: <boolean>,
     writeConcern: <document>,
     collation: <document>
   }
)
```

Unlike the `update` operation, `replaceOne` is the *only* available method for replacing documents. This means you cannot replace multiple documents with a single command. This distinguishes it significantly from the `update` operation.

Another key difference lies in the scope of the operation. While `update` modifies specific fields, `replaceOne` replaces the *entire* document with a new document.

Consider the following JSON document:

```json
{
  "id": 12345,
  "name": "test",
  "age": 1
}
```

To replace the document's data, setting `age` to `10`, you would use:

```jsx
db.collection.replaceOne( {id: 12345}, {age: 10});
```

This command replaces the entire document with `id: 12345` with a new document containing only the `age` field set to `10`, resulting in:

```json
{
  "age": 10
}
```

With the introduction of `replaceOne` in version 3.2, the distinction between `update` and `replace` became more defined. Consequently, to modify specific fields within a document, the `update` operation should be used. To replace an entire document, the `replaceOne` operation should be employed.