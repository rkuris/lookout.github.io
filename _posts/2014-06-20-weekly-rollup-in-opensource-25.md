---
layout: post
title: "Open Source Rollup for Week #25"
tags:
- weekly-oss
- opensource
---

Post-hackathon, this week has been largely "back to normal" as far as Lookout's
open source hacking is concerned. As always you can find our repositories in our [GitHub
organization](https://github.com/lookout).

Without further ado, here's a roll-up of what's been hackin'!

---


This week we've open sourced 1 new repository:
[lookout/borderpatrol](https://github.com/lookout/borderpatrol). Not to be
confused with [ngx\_borderpatrol](https://github.com/lookout/ngx_borderpatrol)
which we recently [wrote about](/2014/06/introducing-borderpatrol/). More
explanation from Lookout hacker [Rob Wygand](https://github.com/rwygand):

> *Lookout's "Hackathon V" birthed a Finagle-based reimagining of Border Patrol.
> This version is completely non-functional, unwarranted, and
> experimental, but does represent a future direction the Lookout engineering
> team is hoping to embrace.*


Meanwhile, [ngx\_borderpatrol](https://github.com/lookout/ngx_borderpatrol) is
in the process of having [statsd
support](https://github.com/lookout/ngx_borderpatrol/pull/11) added.

---

A 1.0.0 [release](https://github.com/lookout/factoryjs/releases) of [Factory
JS](https://github.com/lookout/factoryjs) was created. From contributor [Steve
Regester](https://github.com/egeste):

> *This release was a major due to the fact that it changed the directory
> structure and is a breaking change.*

A 1.*1*.0 release is in the works that will switch away from
[RequireJS](http://www.requirejs.org/) towards
[Browserify](http://browserify.org/). We'll definitely be posting a separate
blog post when that release is fully baked.

---

We've opened up a [pull
request](https://github.com/localshred/protobuf/pull/199) from our
[fork](https://github.com/lookout/protobuf) of the
[protobuf](https://rubygems.org/gem/protobuf) with support for HTTP as an RPC
transport. Protobuf HTTP support was built on top of the
[Faraday](https://rubygems.org/gems/faraday) gem, which provides fantastic
support for pluggable HTTP/REST API calls (we highly recommend it).


This weeks raw numbers:

 * 11 issues created
 * 13 git pushes
 * 2 comments on pull requests
 * 1 pull request created
 * 1 repository open sourced



### Contributors


  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/d33b243451d463d6df3982a493a1f1f7?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/docgravel" target="_blank">docgravel</a>
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
  <img align="absmiddle" src="http://www.gravatar.com/avatar/b8b1657e3d9725114383b2763d367a3a?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/tlrobinson" target="_blank">tlrobinson</a>
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
