---
title: MongoDB Optimization Part 2 - Common Bottlenecks
date: 2022-10-15 10:15:15 +0300
categories: [MongoDB, Optimization]
tags: [mongo, nosql, optimization, mongodb, mongodb-optimization, series-1]
image:
path: /assets/2022-09/mongo.png
width: 1000
height: 400
alt: MongoDB Logo
---

<!-- // 1. Person

// | Field | Type |
// | --------- | -------- |
// | \_id | ObjectId |
// | name | String |
// | age | Number |
// | birthdate | Date |
// | hobbies | Array |
// | job | ObjectId |

// 2. Job

// | Field | Type |
// | ----------- | -------- |
// | \_id | ObjectId |
// | name | String |
// | salary | Number |
// | description | String |

// I have seeded the database with around 100,000 (133,553 to be exact) documents in the person collection, and 20,000 documents in the job collection. -->

In the previous post, we looked at how we can benchmark our queries to get an idea of how fast they are. We also looked at how we can get the execution plan of our queries. In this post, we will look at some of the common bottlenecks we can encounter when querying MongoDB and how we can avoid them.

In any data storage system, no matter what type it is, we almost always have two types of operations, reads and writes. This is because we can essentially boil down any complex operation to a read or a write in respect to the data store. Sure we may transform the data in some way, chain multiple operations together, etc. But at the end of the day, we are either reading or writing data.

To make things easier on this post

## Reads

Let's first look into common read operations along with their bottlenecks and how we can avoid them.

### `find`

The `find` operation is the most common operation we will use when querying MongoDB. It is a function that takes a query object, to tell MongoDB what criteria to use when filtering the documents, and returns a cursor.

```js
// const benchmarkedJobsCollection = benchmarkFactory(jobsCollection, {
//     iterations: 10,
//   });
// This is the benchmarked collection we created in the previous post

const result = await benchmarkedJobsCollection.find({});
```

The above code snippet is a simple `find` operation. It returns all the documents in the collection. It is a very simple operation, and since we have not specified any criteria, it will return all the documents in the collection.

To make things easier to digest, let's over simplify everything and try to understand how the database handles queries. Imagine the database as a clerk sitting in the dark and undergound records room, minding his own business. Suddenly, you rush in and ask him to find all the documents in the records room that you want. But mind you, the clerk is not telepathic, so he has no idea what you want. You have to tell the clerk which documents you want by describing some intrinsic properties of the documents. For example, you can tell the clerk that you want all the documents that are printed on a `red` paper. The clerk will take this description, go to the first document, and check if it is printed on a `red` paper. If it is, he will run back and give it to you. If it is not, he will go to the next document and repeat the process. This is how we would expect a `find` operation to work.

Our query object is, for the most part, the description we give to the clerk. Mongo's query object a declarative way of describing the documents we want. It is a JSON object that describes the criteria we want to use to filter the documents. For example, if we want to find all the documents that are printed on a `red` paper, we can use the following query object.

```js
const query = {
  color: "red",
};
```

The descriptions we give the clerk may not always be definitive. For example, we may want to find all the documents that are printed on a `red`, `blue` or `green` paper. Or we may want all documents which are printed on a paper that is smaller than the size of the clerk's hand. We could even ask the clerk to return some number of documents he feels like returning. These are all examples of non-definitive descriptions. Mongo gives us various ways to describe our criteria in a non-definitive way. Some of the most common ones are:

- `$gt` - greater than
- `$gte` - greater than or equal to
- `$lt` - less than
- `$lte` - less than or equal to
- `$in` - in
- `$nin` - not in
- `$ne` - not equal to
- `$exists` - exists
- `$regex` - regular expression

It even gives us a way to combine multiple criteria together using the `$and` and `$or` operators. For example, if we want to find all the documents that are printed on a `red` paper and are smaller than the size of the clerk's hand, we can use the following query object.

```js
const query = {
  $and: [
    // combine multiple criteria together
    { color: "red" }, // the `color` of the paper is `red`
    { size: { $lt: "$$size of clerk's hand" } }, // the `size` of the paper is `less than` the `size of the clerk's hand`
  ],
};

// FYI: The `$$` is a special character that tells Mongo to use the value of the variable that is passed to the query object.
```

Now let's imagine the records room is a very large room. This means every time we ask the clerk to find a document, he has to go through a lot of documents before he finds all the ones
we want. This goes without saying that the more documents we have, the longer it will take the clerk to find the documents we want. Let us see how this affects our `find` operation.

```js
const personsCollection = db.collection("persons");
const benchmarkedPersonsCollection = benchmarkFactory(personsCollection, {
  iterations: 10,
});

const result_one = await benchmarkedPersonsCollection.find({});
const result_two = await benchmarkedPersonsCollection.find({
  age: { $gt: 30 },
});
```

The persons collection has 133,553 documents. Let's see how this trivial query of finding all the documents where the `age` is greater than `30` performs against the query of finding all the documents.

```
Result One:
Slowest Time: 919
Fastest Time: 789
Average Time: 866.8

Result Two:
Slowest Time: 684
Fastest Time: 557
Average Time: 627
```

As we can see, the query that finds all the documents is slower than the query that finds all the documents where the `age` is greater than `30`. We can predict this by thinking about how the clerk would not find the documents but how he hands them off to us. The clerk has to go through every single document either way. But when we ask him to find all the documents where the `age` is greater than `30`, he can skip over a lot of documents and carry less documents to us. A general rule of thumb is that the more documents we have, the slower the query will be. This is because the clerk has to go through more documents before he finds the ones we want.

Let's complicate things a bit more for the clerk. Let's give him a book, and ask him to tear and give us all the pages that have the word `hat` on them. This is very similar to going through the entire record room to find a document which matches our description. Little do we know the book manufacterer saw this coming (damn you book manufacterer), and has added an appendix at the back of the book. The appendix contains a list of words and the pages they are on. The clerk finishes in no time and gives us the pages we want. This EXACT same thing happens in nearly all databases. They have a data structure called an index that is just like the appendix in the book. The index contains the values of fields and the documents they are on. This means the clerk doesn't need to check every single document to find the ones we want. He can just check the index and directly go to the documents we want. This is why indexes are so important. They make our queries faster.

Let's add an index to the `age` field and see how it affects our query.

```js
const personsCollection = db.collection("persons");
const benchmarkedPersonsCollection = benchmarkFactory(personsCollection, {
  iterations: 10,
});

await personsCollection.createIndex({ age: 1 });

const result_one = await benchmarkedPersonsCollection.find({});
const result_two = await benchmarkedPersonsCollection.find({
  age: { $gt: 30 },
});
```

```
Result One:
```
