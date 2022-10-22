- What must be included in the blog post?

  - A very brief recap of part 1
  - Read optimizied or write optimized?
    - Focus more on write optimized and schema structure in later posts
  - How reads typically work?
    - More data returned more time taken
  - Common read problems
  - Ways to improve read speed
    - Indexes
      - $hint
    - Aggregation
      - $Inc to change numeric values
      - Match as early as possible
      - Project only what you need
      - Limit as early as possible
      - Restructure to $match before $facet

- What flow will the blog post have?
  - Establish an analogy
    - The database engine is a clerk
    - The database itself is a library
    - A document is a book
    - A database page is a shelf
    - A collection is the genre of books
    - The clerk always photocopies the book before giving it to you
    - A photocopy is like a network request
    - An index is like an appendix at the back of the book
    - A query is like asking the clerk to find a book, based on a description

## On the last ...

In the previous post, we looked at how we can benchmark our queries to get an idea of how fast they are. We also looked at how we can get the execution plan of our queries. In this post, we will look at some of the common bottlenecks we can encounter when querying MongoDB and how we can avoid them.

## I will read 500 words, and I will write 500 more

In any data storage system, no matter what type it is, we almost always have two types of operations, reads and writes. This is because we can essentially boil down any complex operation to a read or a write in respect to the data store. Sure we may transform the data in some way, chain multiple operations together, etc. But at the end of the day, we are either reading or writing data.

If the only operations we have are reads and writes, then the reason our queries are not performing well must because of issues in reads or writes. As we will see later in this series, the more we try to read optimize our database, the more likely we will sacrifice our write performance, or vice versa. The need for every use case is different, and we must always be aware of the tradeoffs we are making. In this post, we will focus on read optimization.

To make matters simpler, and since I'm a bit of a fan of analogies, I will try to abstract the entire database engine and database system into a clerk at a library. I know, it's a bit of a stretch, but sue me ... I'm pretty sure it won't hold up in court, but it will (probably) help us understand the concepts better. We will also assume you are an absolute madlad armed only with the most rediculous requests, while the humble clerk is trying to help you out as much as possible.

## Clerk Kent

Imagine you went to library (yes, imagine because we both know you don't go to libraries anymore). This library you went to is a bit quirky, you can't go past the counter, and your only point of accessing the library is through the clerk. The library also holds books that only have a single copy, so the clerk has to photocopy the entire book and hand it over to you.

In this analogy, the clerk is the database engine, the library is the database, the book is the document, and the photocopy is the database request. We can get on with the rest of the post now.

P.S This post won't focus much (if not at all) on code, but rather on the concepts. I will try to explain the concepts as best as I can. I will hopefully have the benchmarked code for the examples in the next post.

## How reads typically work?

When we go to the clerk, the first thing we do is (after greetings ofcourse) tell him the description of the book we want. But how do we do that actually? If we know exactly which book we want, we can tell the clerk the ISBN (International Standard Book Number). But I'm not sure we go around holding a list of ISBN numbers, they're not very human readable, and communicate no useful information about the book to be used else where.

What we probably do however be, asking the clerk to find a book based on a description. For example, we might say `I want a book by Agatha Tolkien, written in 2069, and is about a gentleman hobbit detective`. This way we can let the clerk know what book we want by using a description of the book (in a declarative way).

A beginner clerk would go to the shelf and start looking for the book. He would go through every book in the library, one by one and checks if the book matches the description. If it does, he pile it in a cart or something (the mechanics of this won't matter until much later). If it doesn't, he moves on to the next book. When the clerk finishes checking the entire library, he would then go to the photocopy machine and photocopy the books in the cart. He would then hand over the photocopies to you.

This is how a naive implementation of a read operation would work. The database engine would take the query, and go through every document in the collection, one by one, and return the documents that match the query. This is a very inefficient way of doing things, and I'm sure you can see why.

The first takeaway we can have from this is that the more data we return, the more time it takes to return the data. This is because the clerk has to go through more books, and photocopy more books. This is also why we should always try to limit the amount of data we return, and only return the data we need.

## Common read problems

Now that we have a basic understanding of how reads work in MongoDB, we can look at some of the common problems we can encounter when reading data from MongoDB. The first problem we can encounter is the naive implementation of a read operation. The naive implementation is not a problem in itself, but it is a symptom of a bigger problem, which is that the clerk is not using any of the tools at his disposal.

### Indexes: What do they know? Do they know things? Let's find out!

The tools the clerk has at his disposal are the indexes. Indexes are commonly found at the back of books, and they are used to help you find occurences of a word in a book. But the concept of indexes cannot be limited to books, it can be applied to any data structure. In our library, we can have one book that is not an actual book, but a list of where books of some characteristics are located. This list is called an index.

For example, if we created an index that holds the location of all the books written by the same author, it may look like this:

| Author          | Location                       |
| --------------- | ------------------------------ |
| Agatha Tolkien  | Room 4, Shelf 2, Row 3, Book 1 |
| Agatha Tolkien  | Room 9, Shelf 1, Row 1, Book 2 |
| Elliot Alderson | Room 1, Shelf 3, Row 3, Book 7 |
| Borat Horseman  | Room 2, Shelf 1, Row 1, Book 8 |

etc...

Now if we give the same request `I want a book by Agatha Tolkien, written in 2069, and is about a gentleman hobbit detective`, the clerk checks the index first. He sees that there are two books by Agatha Tolkien, so he now only has to go to those two locations, and check if the books match the description. If they do, he photocopies them, and hands them over to you. If they don't, he moves on to the next book.

Just by adding this little index, the clerk has reduced the amount of time it takes to find the books by a lot. This is because he doesn't have to go through every book in the library, he only has to go through the books based on the index. This is the same for MongoDB, if we create an index, the database engine can use the index to find the documents that match the query, and return them. This is a much more efficient way of doing things, and we should always try to create indexes for our queries.

### Indexes: The bad, and the ugly

This in itself creates new problems though. The first problem is that the clerk has to maintain the index. If a book is added to, removed from, or moved in the library, the clerk has to update the index.

Also, we shouldn't forget the index itself is a book, so it takes up space in the library. If we create too may indexes on too many fields, say for example, we create an index for books that start with 'a', 'b', 'c', etc. Then we would have to maintain a lot of indexes, and the library would be filled with indexes.

### Not all indexes are created equal

Some indexes are more useful than others. For example, while creating an index for books based on the author makes sense, creating an index for books based on the first letter of the author's name + the second letter of the book's name probably won't be very useful (unless you have a VERY VERY unusual usecase).

Bottom line is indexes are awesome, we should always try to create indexes for our queries to improve performance, but we not just make indexes for the sake of making indexes. We should also try to make the most useful indexes for our queries. One way to achieve this is to not use indexes at first, and when we notice queries are slow, we can then create indexes for what is common in most of the queries.

### Mo indexes, mo problems

Another thing before we leave the realm of indexes is compound indexes. Compound indexes are indexes that are made up of multiple fields. For example, we can create an index for books based on the author and the year the book was written. This way, we can find books based on the author, or the year the book was written, or both. This is useful for queries that most commonly appear together. This saves us from having to go through two index books to find the books we want.

The problem with compound indexes is that they take up more space than single field indexes. This is because they have to store the values of multiple fields. This is why we should only create compound indexes for queries that most commonly appear together.

### Excuse me, clerk. I know more than you.

Final thing to note that won't, most of the time, be a problem but might want to keep in mind for that extra 1% of the time is that the clerk probably has an index selection process. This means that the clerk will choose the most useful index for each query. This is useful if you have a lot of indexes, and the clerk has to decide which index to use for the query. This itself takes time (a small amount of time, but still time). If you are a regular user of the library, and know the lay of the land, you can probably tell the clerk which index to use for each query. This will save the clerk time, and you time. This is the same for MongoDB, if you know the most useful index for each query, you can tell the database engine which index to use for each query through the `$hint` operator.

## On the next...

This is as good a place to stop as any. I was hoping to cover more topics, especially on aggregation pipelines, but I think this is already getting too long. I hope you enjoyed this post, and if you did, please like, share, comment, and subscribe. I'll be posting more articles on MongoDB, and other topics, so stay tuned!
