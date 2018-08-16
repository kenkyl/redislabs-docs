---
Title: Developing with Sets in a CRDB
description: $description
weight: $weight
alwaysopen: false
---
Redis Sets are an unordered collection of Strings. It is possible to
add, remove, and test for the existence of members with Redis commands.
A Redis set maintains a unique collection of elements. Sets can be great
for maintaining a list of events (click streams), users (in a group
conversation), products (in recommendation lists), engagements (likes,
shares) and so on.

Sets in CRDBs behave the same and maintain additional metadata to
achieve an "OR-Set" behavior to handle concurrent conflicting
writes. With the OR-Set behavior, writes across multiple CRDB Instances
are typically unioned except in cases of conflicts. Conflicting Instance
writes can happen when a CRDB Instance deletes an element while the
other adds the same element. In this case and observed remove rule is
followed. That is, remove can only remove instances it has already seen
and in all other cases element add wins.

Here is an example of an "add wins" case:

**Time**

**CRDB Instance1**

**CRDB Instance2**

t1

[SADD key1 "a"]{style="font-family: courier;"}

t2

[SADD key1 "b"]{style="font-family: courier;"}

t3

[SMEMBERS key1\
"a"]{style="font-family: courier;"}

[SMEMBERS key1\
"b"]{style="font-family: courier;"}

t4

[--- Sync ---]{style="font-family: courier;"}

t3

[SMEMBERS key1\
"a"\
"b"]{style="font-family: courier;"}

[SMEMBERS key1\
"a"\
"b"]{style="font-family: courier;"}

Here is an example of an "observed remove" case.

**Time**

**CRDB Instance1**

**CRDB Instance2**

t1

[SMEMBERS key1\
"a"\
"b"]{style="font-family: courier;"}

[SMEMBERS key1\
"a"\
"b"]{style="font-family: courier;"}

t2

[SREM key1 "a"]{style="font-family: courier;"}

[SADD key1 "c"]{style="font-family: courier;"}

t3

[SREM key1 "c"]{style="font-family: courier;"}

t4

[--- Sync ---]{style="font-family: courier;"}

t3

[SMEMBERS key1\
"c"\
"b"]{style="font-family: courier;"}

[SMEMBERS key1\
"c"\
"b"]{style="font-family: courier;"}

 