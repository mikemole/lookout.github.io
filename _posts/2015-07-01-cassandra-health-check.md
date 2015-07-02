---
layout: post
title: "Cassandra Health Checks: can A see B and C?"
tags:
- cassandra
- opensource
- monitoring
---

We recently ran into a problem during a rolling restart where one node was
up, but couldn't see the other nodes. While a restart solved the problem
pretty easily, it took us a lot of time to figure out what was going wrong,
especially since all of our current monitoring focused primarily on the
health of a single node.

What we were missing was that a node is only really "healthy" if it can
answer some queries itself as well as talk to every other node in the cluster.
This isn't easy for Cassandra since nodes can come and go.

What we found was that creating a keyspace with a replication factor equal
to the number of nodes in the cluster, then doing a SELECT on that with CL_ALL
will guarantee that every node is reachable from the node performing the
query. So, we wrote some code to do just that.

Our new open source
[Cassandra Health Check](https://github.com/lookout/cassandra-health-check)
automatically creates and/or recreates a keyspace named 'healthcheck'
that has a replication factor equal to the number of nodes in the cluster,
then inserts a row and queries for that row.
If the selected coordinator reports that all of the replicas are consistent,
you get an exit code of 0 (success).

While that's what happens most of the time, sometimes the coordinator can't
get consensus from all of the nodes in the cluster. In that case, the
code downgrades the consistency to make sure the data is available somewhere.
We treat that as a warning, and return an exit code of 1.

If something more catastrophic goes wrong, and you can't get any data at all
from this node, then you get an exit code of 2.

This exit code is used by our consul service so we can get better cluster-wide
health information from the consul gui interface, as well as alerting.

Special care is taken to make sure that the coordination of the query happens only from localhost (or the node specified on the command line).

Just want a standalone binary? Grab the jar from [bintray](https://bintray.com/lookout/systems/cassandra-health-check#files), then just run it with

    java -jar cassandra-health-check-1.0.0-all.jar
\---  
\- [Ron Kuris](https://github.com/rkuris)
