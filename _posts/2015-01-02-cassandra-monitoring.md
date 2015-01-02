---
layout: post
title: "Reliably monitoring Cassandra with Statsd"
tags:
- cassandra
- opensource
- monitoring
---


I'm a newbie at Lookout, but already loving working here. One of the cool projects
I was asked to help on was to merge the few Cassandra instances we have at Lookout
into one. One of my
[freshly adopted rules](https://twitter.com/rkuris/status/543149098996469760) is
that "if it moves, graph it". So, I started researching monitoring Cassandra.

I found that the stuff that's out there is pretty scattered. Both
[nodetool](http://www.datastax.com/documentation/cassandra/2.1/cassandra/tools/toolsNodetool_r.html)
and [DataStax OpsCenter](http://www.datastax.com/what-we-offer/products-services/datastax-opscenter)
seem to rely on JMX to gather their statistics. This is really a "poll operation" which means
that you are going to have another service you'll have to stand up if you want to look at trends.

The first thing I looked at was integrating with [New Relic](http://newrelic.com). It
appeared to have a semi-supported [cassandra plugin](https://github.com/threelegs/newrelic-plugins) that
had [some issues](https://github.com/threelegs/newrelic-plugins/issues), some that were easy to fix
but there would be a lot more work to do to make it secure enough for us.
The first problem was that [JMX didn't use any credentials](https://github.com/threelegs/newrelic-plugins/issues/20).
This is fixable; it's not hard to add credentials to JMX. 
In general though, any ingress to a critical service, like our cassandra server would need authentication,
authorization, a security audit, and periodic vulnerability reviews.

The other problem with this approach is that the poller machine itself might fail,
which means that all nodes might stop reporting. We might think our entire cluster
has failed, and wake someone up, just because the reporting node crashes. The real
issue here is that there's no redundancy in this solution.

Lookout also uses [graphite](http://graphite.wikidot.com) and [statsd](https://github.com/etsy/statsd)
for gathering and displaying information, so I headed off to check that out instead, especially since
Cassandra has [native support for graphite](http://www.datastax.com/dev/blog/pluggable-metrics-reporting-in-cassandra-2-0-2).
This would have worked for us, but we would still have some authorization issues. UDP outbound-only
reporting of statsd is really what we wanted.

Despite Cassandra claiming that it actually has
[pluggable metrics](http://www.datastax.com/dev/blog/pluggable-metrics-reporting-in-cassandra-2-0-2),
what that really means is you can plug in any metric reporter you want, as long as it's
[console, csv, ganglia or graphite](https://github.com/addthis/metrics-reporter-config).
If you want Cassandra to report directly to statsd via the codehale metrics library using
[another open source project](https://github.com/organicveggie/metrics-statsd), you're
pretty much out of luck, unless...

An awesome idea for plugging in some extra code into an existing executable is to use a
[java agent](http://docs.oracle.com/javase/7/docs/api/java/lang/instrument/package-summary.html).
This approach was taken by another open source project to get
[statds support in cassandra](https://github.com/StartTheShift/UnderSiege).
This works great -- each client independently sends a UDP packet with the
statistics to the server. We don't have to open any ports up except for an inbound UDP
to statsd. Since it's connectionless UDP, if the server is down, we just lose a packet,
and we'll just send another in 10 seconds.

There were a few bugs and enhancements we needed to make this work with our
infrastructure, so we forked and created a
[github repo for this](https://github.com/lookout/cassandra-statsd-agent/tree/master).

### Putting this all together

If you want statsd for cassandra, it's super easy now. First, grab these two jars:
 
    curl -L http://dl.bintray.com/lookout/systems/com/github/lookout/metrics/agent/1.0/agent-1.0.jar -o agent-1.0.jar
    curl -L https://oss.sonatype.org/content/groups/public/com/bealetech/metrics-statsd/2.3.0/metrics-statsd-2.3.0.jar -o metrics-statsd-2.3.0.jar

Put these jars in cassandra's lib directory.

Change cassandra startup to add this agent. This can be done in
a stock install by adding the following to /etc/default/cassandra:

 `export JVM_OPTS="-javaagent:/usr/share/cassandra/lib/agent-1.0.jar=localhost"`

Note the '=localhost' at the end. If your statds server is somewhere else, you
should change this.

There are some additional installation details available
[in the README](https://github.com/lookout/cassandra-statsd-agent/blob/master/README.md),
including options to change the port number or reporting interval.

---

With this easily-cheffable plugin, we're off to add additional monitoring
to our cassandra clusters. What a great way to start 2015!

- [Ron Kuris](https://github.com/rkuris)
