---
layout: default
---


<!-- #humblebrag -->

## Open Source Projects

### Border Patrol

[Border Patrol](https://github.com/lookout/ngx_borderpatrol) is an nginx
module to perform authentication and session management at the border of your
network.

BorderPatrol makes the assumption that you have some set of services that
require authentication and a service that hands out tokens to clients to access
that service. You may not want those tokens to be sent across the internet,
even over SSL, for a variety of reasons. To this end, BorderPatrol maintains a
lookup table of session-id to auth token in memcached.


### FactoryJS

[FactoryJS](https://github.com/lookout/factoryjs) is a Factory tool for
standardizing object definition and retrieval in various systems. To use this
in your code you would describe the Factory of objects you are going to be
creating, this is usually a domain of object definitions that have reasonable
similarity.


### Active Push

[ActivePush](https://github.com/lookout/activepush) is a web push service built
around ActiveMQ (or any STOMP broker).

ActivePush subscribes to a STOMP broker and relays messages with a specific
`push_id` header to subscribed Socket.io clients. The message bodies are opaque
to ActivePush, so the service is useful in a variety of applications.



---

## Ruby Gems

* rack-graphite
* stapfen
* openbanana
* testengineer
* budurl-gem
* commit2jira


---

## Forks

* ActionBarSherlock
* lookout-statsd


---

You can find more code open sourced by Lookout [on our GitHub
page](https://github.com/lookout).
