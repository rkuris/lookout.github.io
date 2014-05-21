---
layout: post
title: Introducing Border Patrol, service authentication at the border
tags:
- borderpatrol
- ruby
- rails
- nginx
---


As Lookout has grown over the years, our server infrastructure has continued to
evolve. Migrating from one application to many presents a number of design and
implemtnation challenges, this post touches on one of those challenges:
cross-service session composition, and the tool we developed to solve it:
[Border Patrol](https://github.com/lookout/ngx_borderpatrol).


## Growing pains

Historically, Lookout had been users of both Ruby and
[Rails](https://rubyonrails.org); the original product back-end was built as a
single Ruby on Rails application. It was a massive, sprawling application that
did everything: served the JavaScript-based Lookout front-end applications;
handled billing; housed asynchronous worker jobs; managed the various
databases; held all metadata for backed up devices; was the interface for all
devices communicating with Lookout; and on. and on and on.


The big monorails approach works for only so long and when we started to develop
the [Mobile Threat Network](https://www.lookout.com/mobile-threat-network), the
days of a single Rails application became numbered.


As we began to make the transition from a monolithic Rails application
to a service-oriented architecture, we quickly determined that the monorails
application couldn't own authentication for all services, in addition to its
100 other responsibilities. We would need to fundamentally change the way we
handled authentication.


## Crawl before you walk

Since we only ever had one service, we never had to think about
service-to-service authentication. To that end we built a foundational service
called **Keymaster**.

Keymaster hands out short-lived authentication tokens to services and
devices, allowing them to make authenticated calls to other services. Think of
it like kerberos for RESTful API calls.


**Keymaster** is a whole other blog post, but there's one important point to
cover here: **Keymaster** tokens are one-to-one. They're issued by a specific
service for another specific service. This is great for back-end services
communicating with other back-end services. Where we found it specifically
break down for us was in developing those pesky JavaScript web applications.
We wanted to build an intelligent JavaScript web application that
could pull in content from any number of back-end services, but to do this with
just the Monorail and Keymaster would mean either that:


  1. The monorail would have to proxy requests to other services or
  1. We'd need to implement Keymaster token signing and encryption in JavaScript
  1. Have the JavaScript applications use the same APIs that devices did to
     request tokens from Keymaster.


None of these were attractive options. The first meant adding more code to the
sprawl. The second meant implementing/relying on JavaScript cryptographic
libraries. _O, that way madness lies; let me shun that_.

The third option meant potentially complex token management code shared across
multiple web front-ends.

This was the least most attractive of the three, but then I attended a talk
that gave me another idea.


## Dangerously good ideas

At [Ricon West 2012](http://ricon.io/archive/2012/west.html) I saw a talk by
[Dana Contreras](http://twitter.com/danadanger) of Twitter on how they
decomposed their monorail into services ([Rebuilding a Bird in
Flight](http://vimeo.com/55503728) (video)). In that talk she briefly
mentions how Twitter has pushed their authentication to their proxy layer. This
seemed like the way to go.


Lookout was gearing up to launch [Lookout for Business
(L4B)](https://www.lookout.com/mobile-security-for-business), which had an
entirely new stack separate from the monorail. Since the monorail still owned
functionality like device Lock/Locate/Wipe, the L4B stack would need to make
calls into the monorail.

Lookout already had a service that generates authentication tokens for devices.
We also had a variety of other services using those tokens for authentication
via an HTTP Header on RESTful API calls.

We were launching a new service with a new back-end that needed to somehow
integrate with old infrastructure.

This was the perfect use case to build a system like the one Dana had mentioned
at Ricon.


## Meet Border Patrol


**Border Patrol** is an [nginx](http://nginx.org/) module implemented in
[Lua](http://www.lua.org) that performs authentication at the edge of the
network. Border Patrol is basically a big session store whose values are the
**Keymaster** tokens for the upstream services an end-user wants to speak to.

Here's an example:

  1. A client requests a protected resource from a service.
  1. **Border Patrol** determines there is not a valid session for this
     request, and simply let's the request pass through
  1. The upstream service redirects the user to it's own hosted login page.
  1. The user fills out their credentials and submits the form. This POSTs to
     **Border Patrol** who validates the credentials via **Keymaster**
  1. Assuming the credentials were valid, **Border Patrol** creates a new
     session, inserts the **Keymaster** tokens acquired into the session, and
     injects the appropriate auth token into the service request. Now, whenever
     **Border Patrol** sees that session, it will inject the service-appropriate
     token.


<center>
<img src="/images/post-images/intro-to-borderpatrol/bp-flow.png" alt="Flow of
requests in Border Patrol"/><br/><strong>Series of requests made in Border
Patrol</strong>
</center>


Currently, Border Patrol relies on [memcached](http://memcached.org) for its
session store.  Additionally, we lean on **Keymaster** to do the actual user
authentication and token generation. However, any auth-token system could be
used. Which leads me to..


## Future Direction

The current Lua implementation is messy since the nginx Lua modules don't allow
for creation of directives. because of that, we've been forced to implement
[subrequest](http://www.evanmiller.org/nginx-modules-guide-advanced.html#subrequests)
spaghetti inside of the nginx module itself. It would be cleaner to re-implement
in C using the native nginx directive APIs so that we could simplify
authorization configuration to look something like:


    location / {
      auth_with_border_patrol
    }


Secondly, we're slowly splitting the logical components of Border Patrol out into
micro-services of their own. The first of these is "**Checkpoint**", which
centralizes login and account management functions (password change, account
recovery).

This will enable other front-end services to simply be **Keymaster** aware,
reducing code duplication in various products at Lookout as well as simplifying
the login flow inside Border Patrol.


Other things we're planning to add to Border Patrol are:

  1. Moving from Memcached as session store to a Session micro-service
  1. Adding a rate-limiting micro-service so that we can further remove shared
     code from various services
  1. Adding a routing engine, so that we can better deal with service rollouts
     and API version changes.


Lookout is pleased to announce that we're open-sourcing **Border Patrol**. If
you'd like to know more, further details can be found in the project's [public
GitHub repository](https://github.com/lookout/ngx_borderpatrol).


*- [Rob Wygand](https://github.com/rwygand)*
