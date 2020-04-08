---
title: 'Review: Intro to ElasticSearch - Pluralsight Courses'
date: '2020-04-08T13:45:00Z'
---
[Searching and Analyzing Data with ES: Getting Started](https://app.pluralsight.com/library/courses/elasticsearch-analyzing-data/table-of-contents)

[Designing Schema for ElasticSearch](https://app.pluralsight.com/library/courses/elasticsearch-designing-schema/table-of-contents)

## Overview

These two courses by [Janani Ravi](https://app.pluralsight.com/profile/author/janani-ravi) provide an introduction to Elasticsearch:
- what is Elasticsearch
- when is Elasticsearch appropriate for your application/platform
- how to interact with Elasticsearch

The course also covers the history of search as well as the search algorithms used by ES to make it performant.

Overall, I found the course to be a great introduction to ES as a techonology and the demos were helpful, concise and informative in understanding how to configure and interact with an ES cluster.

## Key Takeaways 
- ES is a schemaless, document based data store that is optimized for search.
- the "Index" is the key component of Elasticsearch and is analougous to a collection in MongoDB and conceptually similar to a table in a relational database.
- ES indices are distributed amongst various nodes in a cluster. This is to say an ES index is "sharded" across multiple nodes.
- ES nodes are replicated to provide high availability.
- you can interact with an ES cluster via a REST API. Querying an ES cluster can be done with search parameters in the url or in the request body.  IMO the latter is preferable as the request body is in JSON format and is easy to read.
- by default, ES infers the type of an index field based on the incoming JSON object.  This is generally not desireable as you'll want more control over these fields and therefore can specify upfront which fields of which type make up the index. This is similar to defining a database table in a RDBMS.
- ES shines when it comes to text search.  By default, a string field will be indexed both by "full text search" strategy and "keyword" strategy.  The former strategy splits up the text, tokenizes and normalizes said text for easy text search. The latter is an exacty match on a provided search string.
- querying of an ES index seems very robust, including pattern matching, boolean logic, subdocument search, etc.
- while ES documents can have pointers to other documents and have parent-child relationships, my impression based on these tutorials was that this is really not what ES should be used for.  It's main purpose is for performant search queries and joining of documents and normalized data significantly hampers performance.
- the author introduced a nifty UI tool for ES cluster monitoring called [ElasticSearch Head](https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm?hl=en-US)

