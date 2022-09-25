---
title: MongoDB Optimization Part 1
date: 2022-09-25 09:54:00 +0300 
categories: [MongoDB, Optimization]
tags: [mongo, nosql, optimization, mongodb, mongodb-optimization, series-1]
image:
  path: /assets/2022-09/mongo.png
  width: 1000
  height: 400
  alt: MongoDB Logo
--- 

As many before me, I was introduced to database systems in university, with a relational DBMS. I learned about normalization, transactions, ACID ... the list goes on. So when I first heard of this mystical thing called a document database, which didn't use SQL, which very publicly denounced normalization, I was a bit skeptical. How could a document database be better than a relational database? After all, relational databases have been around for a very long time, and have been used to build some of the most complex systems in the world. I thought it was just a fad, and that it would die out soon. But, oh boy, was I wrong. After working with mongo for a while, I have come to appreciate its simplicity and flexibility. In this post, I will try to recap a bit on database systems and start to delve into optimization tips and tricks for mongo.

## Relational vs document databases

Relational databases store data in tables, where each row represents an entity, and each column represents an attribute of the entity. Each table is defined by a schema, which specifies the name of the table, and the name, type, and constraints of the columns. Basically, you can visualize relational databases as a spreadsheet, where each row is a record, and each column is a field.

Document databases on the other hand, store data in documents, which are similar to JSON objects. Each document represents an entity, and each field represents an attribute of the entity. The documents are stored in collections, which are similar to tables. Collections, however, do not have a schema. This means that the documents in a collection do not need to have the same fields, and the fields do not need to have the same type. In the case of document databases, you can imagine each document is a json file. and a collection is a folder.

Mongo is a document database, plain and simple. To the best of my knowledge, it is by far the most popular document database out there. It is also the database I have the most experience with, so I will be focusing on mongo in this post. However, the concepts I will be talking about are applicable to other document databases as well.

## Benchmarking

Before we can get started to improve performaces, we need to have a baseline to compare against. To do this, first we need to have a dataset to work with.
I have went along and created two collections,

1. Person

| Field     | Type     |
| --------- | -------- |
| \_id      | ObjectId |
| name      | String   |
| age       | Number   |
| birthdate | Date     |
| hobbies   | Array    |
| job       | ObjectId |

2. Job

| Field       | Type     |
| ----------- | -------- |
| \_id        | ObjectId |
| name        | String   |
| salary      | Number   |
| description | String   |

I have seeded the database with around 100,000 (133,553 to be exact) documents in the person collection, and 20,000 documents in the job collection.

Now we can create a simple wrapper to the mongo client to simplify the benchmarking process, and also to include custom export of the perfomance to create graphs based on them.

Lets start with planning a simple benchmarking wrapper.

What it needs to do is:

1. Accept a query we'd like to benchmark
2. Get the execution plan for the query
3. Run the actual query and get the execution time
4. Return the benchmark results

To make the code look a bit cleaner and nearly similar to regular mongo queries, we can create a proxy object that will wrap the mongo client and return the benchmark results when we call one of the operation methods. Let's start by defining a const that will hold the name of operations we want to benchmark.

```js
const _BENCHMARK_METHODS = [
  "aggregate",
  "count",
  "distinct",
  "find",
  "findOne",
  "findOneAndDelete",
  "findOneAndReplace",
  "findOneAndUpdate",
  "insertMany",
  "insertOne",
  "replaceOne",
  "updateMany",
  "updateOne",
];
```

Now we can create the proxy object. We will use the Proxy object to intercept calls to the mongo client and return the benchmark results when we call one of the operation methods.

```js
const benchmarkProxy = {
  get: (target, property) => {
    // if the property is one of the benchmark methods
    if (_BENCHMARK_METHODS.includes(property)) {
      // run the benchmark here
      return functionThatRunsTheBenchmark;
    }
    // return the original mongo client method
    return target[property];
  },
};
```

That's a good skeleton for the proxy object. Now we can start filling in the details.

First, we need to implement `functionThatRunsTheBenchmark` . This function will accept the query we want to benchmark, and return the benchmark results.

```js
const benchmarkProxy = {
  get: (target, property) => {
    // if the property is one of the benchmark methods
    if (_BENCHMARK_METHODS.includes(property)) {
      return async (query) => {
        // run the benchmark here and return the results
        // we have access to
        // - target: the mongo client
        // - property: the name of the method we are calling (eg. find)
        // - query: the query we are passing to the method (eg. { name: "John" })
      };
    }
    // return the original mongo client method
    return target[property];
  },
};
```

Using the above skeleton, we can start implementing the benchmarking function. We can start by getting the execution plan for the query.

```js
const benchmarkProxy = {
  get: (target, property) => {
    // if the property is one of the benchmark methods
    if (_BENCHMARK_METHODS.includes(property)) {
      // run the benchmark here
      return async (query) => {


        // get the execution plan
        const executionPlan = await target[property](query).explain(
          "executionStats"
        );
        return {
          executionPlan,
        };

    }
    // return the original mongo client method
    return target[property];
  },
};
```

Now we can run the actual query and get the execution time.

```js
const benchmarkProxy = {
  get: (target, property) => {
    // if the property is one of the benchmark methods
    if (_BENCHMARK_METHODS.includes(property)) {
      // run the benchmark here
      return async (query) => {
        // get the execution plan
        const executionPlan = await target[property](query).explain(
          "executionStats"
        );

        const start = Date.now();
        await target[property](query);
        const end = Date.now();
        const executionTime = end - start;

        return {
          executionPlan,
          executionTime,
        };
    }
    // return the original mongo client method
    return target[property];
  },
};
```

Our benchmarking function is now looking good enough. We can now create a factory function that we can use to inject the mongo collection we want to benchmark. This will allow us to create multiple benchmarking functions for different collections.

```js
export const benchmarkFactory = (collection) => {
  return new Proxy(collection, benchmarkProxy);
};
```

As a final touch, lets allow the benchmarking function to accept a configuration object that will allow us to customize the benchmarking process. For example, we can allow the user to specify the number of times to run the query. We can expand on this later to allow the user to specify other options.

```js
export const benchmarkFactory = (collection, config = {}) => {
  const { iterations = 1 } = config;
  collection._config = {
    iterations,
  };
  return new Proxy(collection, benchmarkProxy);
};
```

And modify the benchmarking function to use the configuration.

```js
const benchmarkProxy = {
  get: (target, property) => {
    // if the property is one of the benchmark methods
    if (_BENCHMARK_METHODS.includes(property)) {
      // run the benchmark here
      return async (query) => {
        // get the execution plan
        const explainerClient = await target.explain("executionStats");
        const executionPlan = await explainerClient[property](query);

        // run the query and get the execution time
        const elapsedTimes = [];
        for (let i = 0; i < target._config.iterations || 1; i++) {
          const start = Date.now();
          await target[property](query);
          const end = Date.now();
          elapsedTimes.push(end - start);
        }

        // we can now get a better estimate of the execution time
        const slowestTime = Math.max(...elapsedTimes);
        const fastestTime = Math.min(...elapsedTimes);
        const averageTime = elapsedTimes.reduce((a, b) => a + b) / elapsedTimes.length;

        return {
          executionPlan,
          executionTime,
        };
    }
    // return the original mongo client method
    return target[property];
  },
};
```

Amazing! We now have a simple benchmarking function that we can use to benchmark our queries. We can now use this function to benchmark our queries and get the execution plan and execution time.

```js
const { MongoClient } = require("mongodb");
const { benchmarkFactory } = require("./benchmark");

const mongoUrl = "mongodb://localhost:27017";

const client = new MongoClient(mongoUrl);
await client.connect();

const db = client.db("test");
const jobsCollection = db.collection("jobs");

const benchmarkedJobsCollection = benchmarkFactory(jobsCollection, {
  iterations: 10,
});

const result = await benchmarkedJobsCollection.find({});

console.log(results);
```

The above test yields the output

````json
{
  "executionPlan": {
    "queryPlanner": {
      "plannerVersion": 1,
      "namespace": "test.jobs",
      "indexFilterSet": false,
      "parsedQuery": {},
      "winningPlan": {
        "stage": "COLLSCAN",
        "direction": "forward"
      },
      "rejectedPlans": []
    },
    "executionStats": {
      "executionSuccess": true,
      "nReturned": 20060,
      "executionTimeMillis": 5,
      "totalKeysExamined": 0,
      "totalDocsExamined": 20060,
      "executionStages": {
        "stage": "COLLSCAN",
        "nReturned": 20060,
        "executionTimeMillisEstimate": 2,
        "works": 20062,
        "advanced": 20060,
        "needTime": 1,
        "needYield": 0,
        "saveState": 20,
        "restoreState": 20,
        "isEOF": 1,
        "direction": "forward",
        "docsExamined": 20060
      }
    },
    "serverInfo": {
      "host": "joeking-pee-cee",
      "port": 27017,
      "version": "4.4.5",
      "gitVersion": "ff5cb77101b052fa02da43b8538093486cf9b3f7"
    },
    "ok": 1,
    "$clusterTime": {
      "clusterTime": {
        "$timestamp": "7147275551382700033"
      },
      "signature": {
        "hash": "AAAAAAAAAAAAAAAAAAAAAAAAAAA=",
        "keyId": 0
      }
    },
    "operationTime": {
      "$timestamp": "7147275551382700033"
    }
  },
  "executionTime": {
    "slowestTime": 118,
    "fastestTime": 67,
    "averageTime": 76.8
  }
}
```

For now, focus only on the `executionTime` object (at the bottom of the output). This is the execution time of the query. We can see that the slowest time is 118ms, the fastest time is 67ms and the average time is 76.8ms. We can use this information to generalize that about 20000 documents are being returned in 70-80ms. Ofcause, this is just a rough estimate and there are a lot of factors at play like the processing power of the machine, the size of the documents, etc. But this is a good start.

I know, I know that is an overwhelming amount of information, plus a lot of work for a simple query. But trust me, it will pay off later in this series. We will use this information to optimize our queries.

This turned out to be a long post. I will try to keep the next post shorter. In the next post, we will look at some of the common bottlenecks we can encounter when querying MongoDB and how we can avoid them.

If you have any questions or comments, please leave them in the comments section below. If you found this post useful, please share it with your friends and colleagues. If you want to be notified when I publish a new post, follow me on LinkedIn where I post updates about my blog posts.

#### Thanks for reading!