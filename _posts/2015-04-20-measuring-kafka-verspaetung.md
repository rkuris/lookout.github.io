---
layout: post
title: "Measuring Kafka consumer delay with Verspätung"
tags:
- kafka
- opensource
- monitoring
---


Sometime in mid-2014, [we](https://www.lookout.com/about/careers) started to
roll out [Apache Kafka](http://kafka.apache.org) as a replacement to our older
[ActiveMQ](http://activemq.apache.org) messaging infrastructure. While we were
evaluating and deploying it, we became enamoured with its robustness and clean
architectural design. Many of the problems with our previous infrastructure
would no longer exist in the brave new Kafka-based world.

*Note: if you are unfamiliar with Kafka's design, our very own [Brandon
Burton](https://github.com/solarce) wrote up a good [intro to Kafka](http://sysadvent.blogspot.com/2014/12/day-4-introduction-to-kafka.html) for last year's SysAdvent.*

One key principle with Kafka's design, is that consumers store an **offset**
into the commit log for a topic/partition. This means that for a single topic
(e.g. `device_events`) I can easily have hundreds, if not thousands, of
consumers with no additional storage requirements on the Kafka cluster.

A drawback of this approach is that understanding how "behind" a single
consumer group might be behind the "latest" for a Kafka topic can be difficult
to discover. Most consumers will use [Zookeeper](http://zookeeper.apache.org)
for storing their offsets, while Kafka topics store their "latest offset"
inside of Kafka itself.


Unable to find a suitable solution to track and report on this information, I
embarked on writing a new daemon to help: **[Verspätung](https://github.com/lookout/verspaetung#readme)** (the German word for delay or lateness)


## Meet Verspätung

<img src="/images/verspaetung.png" align="right" width="200"/>

[Verspätung](https://github.com/lookout/verspaetung#readme) is a
[Groovy](http://groovy-lang.org) based daemon which implements a few key
behaviors:

 * Subscribes to Zookeeper subtrees for updates
 * Periodically polls Kafka brokers using the Kafka meta-data protocol
 * Exposes offset deltas to be consumed by metrics systems (e.g.
   [Datadog](http://datadoghq.com).

### Watching Zookeeper

Argubly one of the most important features of Verspätung is its ability to
"watch" an entire Zookeeper subtree for updates. This means that Verspätung
doesn't need an ahead-of-time knowledge of what consumers might be coming or
going. When nodes are added to the Zookeeper subtree, Verspätung will receive
an event and register a metric for that consumer.

Implementing this turned out to be far easier than I had anticipated thanks to
the wonderful  [Curator](http://curator.apache.org) library, and its
[TreeCache](http://curator.apache.org/curator-recipes/tree-cache.html) recipe.

The implementation for this logic can be found in the "TreeWatcher" files [in the
source
tree](https://github.com/lookout/verspaetung/tree/master/src/main/groovy/com/github/lookout/verspaetung/zk).


This information only gives us a part of the puzzle, the currently committed
offset for a consumer group, we still need to get the latest from Kafka.


### Polling Kafka

The best way to get data from Kafka, is to speak Kafka! Therefore Verspätung
also includes a dependency on the Kafka client library.

After studing the code behind Kafka's
[GetOffsetShell](https://cwiki.apache.org/confluence/display/KAFKA/System+Tools#SystemTools-GetOffsetShell)
I implemented Verspätung's
[KafkaPoller](https://github.com/lookout/verspaetung/blob/master/src/main/groovy/com/github/lookout/verspaetung/KafkaPoller.groovy)
whose sole purpose in life is to check in with Kafka and get updated metadata
every now and again.

I wasn't able to find any means of getting an evented stream of this
information, so the `KafkaPoller` will ask Kafka every second (currently
hard-coded) for the latest offsets on all topics and partitions.

Regardless of whether we have active consumers for these topics or not, all
this data is recorded internally to Verspätung.

With the second piece of the puzzle in place, we finally need to start
_reporting_ the deltas!


### Publishing Metrics


Verspätung uses another fantastic library,
[dropwizard-metrics](https://dropwizard.github.io/metrics/3.1.0/), for
publishing metrics on Kafka consumer delays.

When a new consumer is discovered in Zookeeper, Verspätung will register a new
[gauge](https://dropwizard.github.io/metrics/3.1.0/manual/core/#gauges) to its
internal `MetricsRegistry`. The gauge in term will consult our internal [data
structures](https://github.com/lookout/verspaetung/blob/master/src/main/groovy/com/github/lookout/verspaetung/metrics/ConsumerGauge.groovy)
for computing the delta only when asked by the
[reporter](https://dropwizard.github.io/metrics/3.1.0/manual/core/#reporters).

Due to this design, the retrieval of offsets from
both Kafka and Zookeeper is completely decoupled from the recording of metrics.
This allows for a very granular time-period for reporting metrics or a very
coarse one depending on your needs.

<center><img src="/images/post-images/verspaetung/grat-delay.png"/><em>Delay for
a production consumer over 24 hours</em></center>

Currently by default, Verspätung will use our [forked metrics-datadog
reporter](https://github.com/lookout/metrics-datadog) which allows for
dynamic per-Gauge tags.

Verspätung then reports a few different tags to Datadog, namely
"consumer-group', "partition" and "topic" which allows us to cut and slice the
metrics to fit our needs in the Datadog interface.



## Los geht's!


There's still some [outstanding
issues](https://github.com/lookout/verspaetung/issues)  with Verspätung but
we've been running
[v0.2.0](https://bintray.com/lookout/systems/verspaetung/0.2.0/view) in
pre-production and production environments without issues for over a month now.



We hope you find the tool useful!



- [R Tyler Croy](https://github.com/rtyler)
