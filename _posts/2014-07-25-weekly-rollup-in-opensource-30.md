---
layout: post
title: "Open Source Rollup for Week #30"
tags:
- weekly-oss
- opensource
---

The last full week in July has been quite a busy one here at
[Lookout](http://www.lookout.com/about/careers). Despite a number of engineers
on, or preparing for, vacations there was still quite a bit of open source
hacking going on according to [GitHub](https://github.com/lookout). Most of the
development this week was focused on some projects regular readers may be
familiar with: [Oraculum](https://github.com/lookout/oraculum) and [Border
Patrol](/2014/06/introducing-borderpatrol). In addition to those two projects
we had minor changes going into [pipefish](https://github.com/lookout/pipefish)
(a small utility that writes the results of a SQL statement to HDFS) and
[FactoryJS](https://github.com/lookout/factoryjs) (a JavaScript tool for
standardizing object definition and retrieval in various systems).


For the Oraculum project, this week saw some minor bug fixes and contributions
which resulted in a [0.0.2
release](https://github.com/lookout/oraculum/tree/0.0.2). The full diff set can
be [viewed here on
GitHub](https://github.com/lookout/oraculum/compare/0.0.1...0.0.2)


Meanwhile [ngx_borderpatrol](https://github.com/lookout/ngx_borderpatrol)
there's been continued work on merging LuaJIT into the codebase. [One
commit](https://github.com/lookout/ngx_borderpatrol/commit/04abb68d68dd967e764bb38e4e8ececb84dab9c0)
was merged but promptly reverted after finding some issues with it after the
merge, whoops!


### New repositories


While not necessarily completely new, we pulled
[Hermann](https://github.com/lookout/Hermann), a Ruby gem wrapping the
librdkafka library as a C extension. under the Lookout organization. The
Hermann gem wraps [librdkafka](https://github.com/edenhill/librdkafka) to
provide MRI-based Ruby projects producer and consumer access to
[Kafka](http://kafka.apache.org). This differs from
[poseidon](https://github.com/bpot/poseidon), another Ruby gem that connects to
Kafka, in that Hermann doesn't implement or do any heavy-lifting to talk to Kafka but
rather relies on the C library.




The raw numbers:

 * 11 git pushes
 * 11 pull requests created
 * 2 repositories forked
 * 2 comments on pull requests
 * 4 commit comments
 * 6 issues created






### Contributors


  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/42b61b891d0988c200a6cf301fa59212?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/egeste" target="_blank">egeste</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/f11c75ac1acb714cf2d3c3f00a2014ff?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/emilong" target="_blank">emilong</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/65e652ffdfcf956e8dc1bff5dfd669e9?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/ismith" target="_blank">ismith</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/9766b19e046a81a650562b630c8125c9?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/phrinx" target="_blank">phrinx</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/26f775897f32fd3dd4f1cebf3893abd5?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/rothrock" target="_blank">rothrock</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/d565139dbbafc06e7daf4826ca0f0228?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/rtyler" target="_blank">rtyler</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/ad07702900af3578fe320bc5bb0a7842?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/rwygand" target="_blank">rwygand</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/c83ddabc44c7cd98f78db7a9dbdbf672?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/webspinner" target="_blank">webspinner</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/3a38900a6cdc59829aa2c7acc0a1b5e0?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/wkimeria" target="_blank">wkimeria</a>
  </strong>
  </div>

<br clear="all"/>
