Interact with data in Redis
How to interact with data in Redis, including queries, triggered functions, transactions, and pub/sub

Redis 운영에 필요한 다양한 측면들을 상세하게 정리했습니다.

Redis Query Engine
Programmability
Transactions
Publish/subscribe
Search and query with the Redis Query Engine
The Redis query engine lets you retrieve data by content rather than by key. You can index the fields of hash and JSON objects according to their type and then perform sophisticated queries on those fields. For example, you can use queries to find:

matches in text fields
numeric values that fall within a specified range
Geospatial coordinates that fall within a specified area
Vector matches against word embeddings calculated from your text data
Programmability
Redis has an interface for the Lua programming language that lets you store and execute scripts on the server. Use scripts to ensure that different clients always update data using the same logic. You can also reduce network traffic by reimplementing a sequence of related client-side commands as a single server script.

Transactions
A client will often execute a sequence of commands to make a set of related changes to data objects. However, another client could also modify the same data object with similar commands in between. This situation can create corrupt or inconsistent data.

Use a transaction to group several commands from a client together as a single unit. The commands in the transaction are guaranteed to execute in sequence without interruptions from other clients' commands.

You can also use the WATCH command to check for changes to the keys used in a transaction just before it executes. If the data you are watching changes while you construct the transaction then execution safely aborts. Use this feature for efficient optimistic concurrency control in the common case where data is usually accessed only by one client at a time.

Publish/subscribe
Redis has a publish/subscribe (Pub/sub) feature that implements the well-known design pattern of the same name. You can publish messages from a particular client connection to a channel maintained by the server. Other connections that have subscribed to the channel will receive the messages in the order you sent them. Use pub/sub to share small amounts of data among clients easily and efficiently.