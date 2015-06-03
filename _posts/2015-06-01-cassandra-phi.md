---
layout: post
title: "Phi Fi Foe Fun, I smell the blood of a dead Cassandra node"
tags:
- cassandra
- opensource
- monitoring
---

One thing Cassandra does fairly well is direct traffic to nodes that are less busy or have
better throughput. It will even mark nodes as "down" if they aren't reporting gossip as
often as they should.

The latter part is done by the FailureDetector.  The FailureDetector is stateless, whereas
the gossiper has a set of states you might already be familiar with. The gossiper uses the
FailureDetector as a way of knowing that a node just isn't reporting in often enough.

There are two primary methods on the IFailureDetector. The first one is report, which is
what the gossiper calls every time a message is received from that node. A message is supposed
to arrive every second from the Gossiper. The failure detector will record the time between messages
for the last 1000 messages, which should be about 16 minutes, 40 seconds of history. It will not
record the time if the last message was sent more than 2 seconds ago.

With this information at hand, the gossiper does a status check every second as well (usually).
It then calculates this magical "phi" value (pronounced "fee") which is the time since the last message 
divided by the average time of the collected samples. It then divides by a constant, PHI_FACTOR,
which is about 0.434. If that is above phi_convict_threshold, then the failure detector considers
the node "down".

The average collected samples must average less than 2, and rarely falls below 1, since items above 2
are discarded and we only send messages every second. A message delayed just over 2 seconds might be
right behind another that was sent less than a second ago, and we'll only record the second sample,
which can cause the number to drop lower than 1. I doubt this was the intended behavior, but it's how
it works and can be seen in testing. For most networks, even with some jitter, you should see an
average value of about 1.

The upshot is that your phi value says how long it's been since the last sample. If that's (by default) about
18.4 seconds, your node is down. That means in the default configuration, a node is
considered down if you don't get a gossip ping in about 18.4 seconds, if you usually get one every
second. The 18.4 second value came from the default phi_convict_threshold of 8, divided by PHI_FACTOR of
0.434.

There's one more thing to consider, though, and that is that if the FailureDetector decides that a
node should be "convicted", it calls a callback, but the callback can choose to do nothing.
That happens in two places in the code. If you're streaming, or running a repair,
the node is still up unless your phi value is 100 times the convict threshold, or 800 with a
default phi_convict_factor. That math is done without any adjustment by PHI_FACTOR, and the 100 is not tunable.
This means that, by default, you have about 13 minutes, 20 seconds before a node is considered down.

This, of course, is adjustable. The documentation says to increase phi_convict_factor when you're on
AWS to 12. If you set it this high, you instead have about 1200 seconds where gossip doesn't need to
arrive, or about 20 minutes, for repairs or streaming.

### Monitoring Phi

Unfortunately, there are no good ways of monitoring phi, unless you exceed it, in which case the node
is marked down. I wanted some way to know if we're close, so I added some code in
[CASSANDRA-9526](https://issues.apache.org/jira/browse/CASSANDRA-9526) to expose it via JMX, and
added some code in the latest version of the [Cassandra Statsd Agent](https://github.com/lookout/cassandra-statsd-agent)
to send that out to our DataDog system for monitoring.
---
- [Ron Kuris](https://github.com/rkuris)
