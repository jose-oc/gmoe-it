---
title: "Types of Indexes"
date: 2021-11-10T06:44:28+01:00
draft: true
---


Indexes try to minimize the number of pages loaded from disk to solve a query. The more pages the slower, so they can provide a huge improve on performance.

## B-Tree index

It builds a tree as flat as possible with links to the item id and page to find the data.

## Hash index

Excellent for == matching.
Only one field can be used.
This type of index requires very little memory.


## GIN index: Generalised Inverse index

Very efficient for: 

* full text search
* use with tsvector type (useful for jsonb and array fields)

## BRIN index: block range index

It stores the min and max values of a field per page. Useful for fields that only increase or decrease.
This type of index requires extremely little memory.



# Analyzer

When you perform a `explain analyze {sql_query_here}` you get information about how Postgres is going to resolve the query.

A `Seq scan` means that postgres has to go over the whole tabla to get the data that you're asking for.
An `Index only scan` is super fast as it only needs to use the index to get the information requested by the query, so it doesn't have to go to the actual data, to the actual table. 

When the analyzer shows a `Bitmap index scan` means it's using the index to find out which pages it has to read and which records within those pages, so it's quick.
After that it usually does a `Bitmap Heap scan` to fetch the actual data from those pages.

# Cost

The cost of indexes is not only the disk space you need to store them but the RAM they consume, the time to keep them up to date, the time the planner takes to decide how to resolve a query...


----

Interesting video to show how indexes work: [PostgreSQL Indexing : How, why, and when.](https://www.youtube.com/watch?v=clrtT_4WBAw)