---
layout: post
title: "Open Source Rollup for Week #23"
tags:
- weekly-oss
- opensource
---

Over the past week we've been doing quite a lot of open source work within our
[GitHub organization](https://github.com/lookout). Here's a roll-up of what's
been happening!


This week we've open sourced 2 repositories:

 * [lookout/ngx_borderpatrol](https://github.com/lookout/ngx_borderpatrol) - *An nginx module to perform authentication and session management at the
   border of your network.*
 * [lookout/elementary-rpc](https://github.com/lookout/elementary-rpc) - *Elementary
   RPC is a simple, async client library for interacting with Protobuf RPC
   servers*


We released a new version of the
[lookout-jruby](https://github.com/lookout/lookout-jruby) gem (*collection of hacks to make building/running JRuby apps easier*) which
removes some path munging code thanks to revent improvements in
[JRuby](https://jruby.org) 1.7.12.


We've been refining HTTP transport layer support in [our
fork](https://github.com/lookout/protobuf) of the [protobuf ruby
gem](https://github.com/localshred/protobuf). More details in [this pull
request](https://github.com/lookout/protobuf/pull/1).


Finally, our [ngx_borderpatrol](https://github.com/lookout/ngx_borderpatrol)
module is now being tested by [Travis CI](https://travis-ci.org) meaning all
pull requests for the Nginx module will be properly tested before they're
merged.



### Raw Data

The raw numbers:

 * 21 git pushes
 * 10 issues created
 * 13 pull requests created
 * 13 comments on pull requests
 * 2 repositories open sourced
 * 1 commit comment
 * 4 repositories forked

#### Contributors

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/254af7d265d4e131e8c4e35374ceae81?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/abhiyerra" target="_blank">abhiyerra</a>
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
  <img align="absmiddle" src="http://www.gravatar.com/avatar/a061fe6bd3e9018e6837fba4e1d09206?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/reedloden" target="_blank">reedloden</a>
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
  <img align="absmiddle" src="http://www.gravatar.com/avatar/4201b8c4bc17ca5a1fb382a32df35650?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/svrana" target="_blank">svrana</a>
  </strong>
  </div>

  <div style="float: left; margin: 10px;">
  <img align="absmiddle" src="http://www.gravatar.com/avatar/b8b1657e3d9725114383b2763d367a3a?s=48"/>
  <br/>
  <strong>
  <a href="https://github.com/tlrobinson" target="_blank">tlrobinson</a>
  </strong>
  </div>

<br clear="all"/>
