---
layout: post
title: Introducing Border Patrol, service authentication at the border
tags:
- borderpatrol
- ruby
- rails
- nginx
---


As Lookout has grown over the years our server infrastructure has continued to
evolve. Migrating from one application service to many presents a number of
design and implementation challenges. This post touches on one of those challenges:
cross-service session composition and the tool we developed to solve it,
[Border Patrol](https://github.com/lookout/ngx_borderpatrol).


## Growing pains

Historically, Lookout had used both Ruby and [Rails](https://rubyonrails.org);
the original [Lookout Mobile Security](https://play.google.com/store/apps/details?id=com.lookout) application back-end was built as a single
Ruby on Rails application. It was a massive, sprawling application that did everything: served the
JavaScript-based Lookout front-end applications; handled billing; housed
asynchronous worker jobs; managed the various databases; held all metadata for
backed up devices; was the interface for all devices communicating with Lookout;
and on. and on and on.

As we set out building the [Mobile Threat Network](https://www.lookout.com/mobile-threat-network)
and began to make the transition from a monolithic Rails application
to a service-oriented architecture, we quickly determined that the monorail
application couldn't own authentication for all services, too. We needed to
fundamentally change the way we handled authentication.


## Crawl before you walk

Since Lookout only ever had one service, we'd never had to think about
service-to-service authentication. That changed as we dipped our big toe into the
ocean of [SOA](http://martinfowler.com/articles/microservices.html) and realized
we would need to be a new foundation service that we named
**[Keymaster]**(http://i1.ytimg.com/vi/N9L7UUp0FxY/hqdefault.jpg).

**Keymaster** hands out short-lived authentication tokens to services and
devices, allowing them to make authenticated calls to other services. It's like
[Kerberos](http://en.wikipedia.org/wiki/Kerberos_(protocol) for RESTful API calls.

**Keymaster** is a whole other blog post, but there's one important point to
cover here: **Keymaster** tokens are issued by a specific service for another
specific service. E.g.: The _LocationService_ gets a token to talk to the
_PushService_ to initiate a device locate.

This is great for back-end services communicating with other back-end services.
But when you're dealing with Javascript front-end applications that might need
to speak to or pull in data from multiple services, **Keymaster** tokens broke down.
To make this work we'd have to do one of the following:

  1. The monorail would have to proxy requests to other services or
  1. We'd need to implement **Keymaster** token signing and encryption in JavaScript
  1. Have the JavaScript applications use the same APIs that devices did to
     request tokens from **Keymaster**.

None of these were attractive options. The first meant adding more code to the
sprawl. The second meant implementing/relying on JavaScript cryptographic
libraries. _O, that way madness lies; let me shun that_.

The third option meant potentially complex token management code shared across
multiple web front-ends. Doable with a solid library, but really more complexity
than we wanted to push onto our front-end. This was certainly the least offensive
of the three options, but then I attended a talk that gave me another idea.


## Dangerously good ideas

At [Ricon West 2012](http://ricon.io/archive/2012/west.html) I saw a talk by
[Dana Contreras](http://twitter.com/danadanger) of Twitter on how they
decomposed their monorail into services ([Rebuilding a Bird in
Flight](http://vimeo.com/55503728) (video)). In that talk she briefly
mentions how Twitter has pushed their authentication to their proxy layer. This
idea resonated with me and felt like the correct missing solution to our token
management problem.

Lookout was gearing up to launch [Lookout for Business
(L4B)](https://www.lookout.com/mobile-security-for-business), which had an
entirely new stack separate from the monorail. Since the monorail still owned
functionality like device Lock/Locate/Wipe, the L4B stack would need to make
calls into the monorail to trigger those actions.

This seemed like the perfect use case to build a system like the one Dana had
mentioned at Ricon. Lookout already had a service that generated authentication
tokens for devices. We had a variety of other services using those tokens
for authentication via an HTTP Header on RESTful API calls. All we needed was
a service to sit between the web browser and back-end services that would manage
the service-specific tokens internally and speak HTTP session cookies to the
Javascript running in the browser.


## Meet Border Patrol

**Border Patrol** is an [nginx](http://nginx.org/) module implemented in
[Lua](http://www.lua.org) that performs authentication at the edge of the
network. **Border Patrol** is basically a big session store whose values are the
**Keymaster** tokens for the upstream services a browser wants to speak to.

Here's an example:

  1. A client requests a protected resource from a service.
  1. **Border Patrol** determines there is not a valid session for this
     request, and simply let's the request pass through
  1. The upstream service redirects the user to it's own hosted login page.
  1. The user fills out their credentials and submits the form. This POSTs to
     **Border Patrol** who validates the credentials via **Keymaster** and returns
     one or more service tokens
  1. **Border Patrol** creates a session id and returns it to the client via an
     HTTP cookie: this session id informs **Border Patrol** how to retrieve the
     service tokens
  1. Subsequent requests from the browser present their session id via the cookie,
     and **Border Patrol** injects the appropriate service token into the request
     headers

<center>
<img src="/images/post-images/intro-to-borderpatrol/bp-flow.png" alt="Flow of
requests in Border Patrol"/><br/><strong>Series of requests made in Border
Patrol</strong>
</center>

Currently, **Border Patrol** relies on [memcached](http://memcached.org) for its
session store.  Additionally, we lean on **Keymaster** to do the actual user
authentication and token generation. However, any auth-token system could be
used. Which leads me to...

## Future Direction

The current Lua implementation is messy since the nginx Lua modules don't allow
for creation of directives. because of that, we've been forced to implement
[subrequest](http://www.evanmiller.org/nginx-modules-guide-advanced.html#subrequests)
spaghetti inside of the nginx module itself. One thought here is to move to native-c
nginx extensions which would allow us to add or extend existing nginx directives,
making configuration simpler.

But given the direction we're taking **Border Patrol**, we might need to do more
than that. We're already in the process of moving ownership of login and account
management to a Rails service called **Checkpoint**. This lets us build services
that never have to care about login or password management.

Furthermore, with the complexity of nginx, the fact that we don't actually use
most of it, the features we want to add over the year, and the performance
requirements inherent in fronting of every request made into lookout.com, we're
currently thinking about dropping nginx as the engine and moving to a JVM-based
platform. This would allow us to build rate limiting, load shedding, session
storage, and request routing components as services, if we so desired, relying
heavily on evented IO.

Lookout is pleased to announce that we're open-sourcing **Border Patrol**. If
you'd like to know more, further details can be found in the project's [public
GitHub repository](https://github.com/lookout/ngx_borderpatrol).

## Credit where credit is due

**Border Patrol** was conceived of by me, started by [R. Tyler Croy](https://github.com/rcroy),
and has been worked on by both of us and a variety of people at Lookout,
all of whom should be given credit for this blog post, where the service is today,
and where it's going. Those people are Dirk Koehler, Nathan Smith,
[William Kimeria](https://github.com/wkimeria), and Christopher Chong.

*- [Rob Wygand](https://github.com/rwygand)*
