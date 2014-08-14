---
layout: post
title: "Introducing Oraculum"
tags:
- chaplin
- backbone
- backgrid
- oraculum
- factoryjs
- javascript
- enterprise
- opensource
---

### The Enterprise Javascript MVC Framework.

Oraculum is a [javascript MVC framework](http://todomvc.com/architecture-examples/oraculum/) and a collection of `mixin`s for [Backbone](http://backbonejs.org/) `Model`s, `Collection`s and `View`s written for [FactoryJS](https://github.com/lookout/factoryjs/). It inherits all of its application structure, many behaviors, and is generally inspired by [Chaplin](http://chaplinjs.org/).

Why another JavaScript MV* framework?
-------------------------------------

### Playing with Prototypes: A Cautionary Tale.

<a href="https://www.lookout.com/enterprise-mobile-security" style="float:right;padding:1em;border:1px solid #ccc;margin:0.5em 1em;border-radius:0.5em;" target="_blank">
  <img src="/images/post-images/introducing-oraculum/Lookout%20App%20Intel%20Console.png" style="width:200px;" alt="Lookout App Intel Console"/>
  <br/>
  Lookout App Intel Console
</a>

Oraculum emerged to support the development of our [App Intel Console](https://www.lookout.com/enterprise-mobile-security), a dynamic [single page application](http://en.wikipedia.org/wiki/Single-page_application) that allows complex querying of our malware/security knowledge. We knew that the App Intel Console would one day serve as the codebase for our internal research/analysis toolchain, and would need to be embedded into other products/services. With this in mind we decided that we needed to adopt a library that supported workflows and processes for cross-product, cross-team code portability.

Initially evaluating several frameworks, we were all pretty smitten with Backbone and Chaplin. We really liked Chaplin's introduction of a proper `Controller`, as well as the many improvements to `Backbone`'s `Views`, `Models`, `Collection` and `Router` it provided. What's more, Chaplin introduced the `CollectionView`, which beautifully solved the problem of rendering `Collection`s in an elegant and performant way. The tide only started to turn once we got into the nitty gritty of building an application of great complexity. Some of the implicit behaviors Chaplin provided suddenly weren't what we wanted or needed. Modifying Chaplin's underlying object prototypes to change its behaviors was a tedious and error-prone task, often leading to very tightly coupled behaviors and strange side effects. Foregoing this option and using CoffeeScript's faux classes/extension provided some relief, but eventually led to deep inheritance, and, once again, tight coupling/strange side effects. Opting for a `mixin`-based strategy helped to some extent, but the fact that we were still bound to overriding methods on a prototype continued to lead to the same outcome.

<blockquote style="text-align:center;">
  <h2>Frankly, the entire paradigm had to change.</h2>
</blockquote>

### Enter FactoryJS

Around the time we thought we had most of this stuff reasonably figured out, [Gabriel Hernandez](https://github.com/webspinner) joined Lookout supporting our Research & Response team as a Senior Software Engineer. He brought with him a passion for the concepts of [factories](http://en.wikipedia.org/wiki/Factory_method_pattern), [flyweights](http://en.wikipedia.org/wiki/Flyweight_pattern) and [mixins](http://en.wikipedia.org/wiki/Mixin), which quickly caught my ear as a brilliant solution to the structural challenges we were facing. After a few asks, the initial version of FactoryJS was born. We started wrapping Backbone and Chaplin in our fancy new factory container, and abstracted our behaviors out into `mixin`s. Immediately our codebase became more modular, more stable, and easier to test. Code sharing increased dramatically between the App Intel Console product and our internal research tools.

Simply using Backbone and Chaplin wrapped in the factory container took us a long way towards a functional product, but as the App Intel Console continued to increase in complexity, we once again ran into problems related to tight coupling. Chaplin's underlying implicit behaviors were still forcing us to stub out methods we didn't want, modify method implementations, and prohibited the use of things we had taken for granted in vanilla Backbone (like the `initialize()` method). Eventually we decided that while we loved the additional behaviors Chaplin provided, we couldn't continue to use them as they were written. We also needed to abstract these behaviors out into `mixin`s. Thus Oraculum was born.

So how does this thing work?
----------------------------

Oraculum is actually a library with three separate types of components and two utility functions. [The official documentation](http://hackers.lookout.com/oraculum) goes into great detail about all of these components, but from a high-level perspective, these components are:

  1. A set of application/MVC components.
  1. A set of `mixin`s/`definition`s for rendering tabular data.
  1. A library of `Abstract`, `Model`, `Collection`, and `View` behaviors.
  1. A set of utility functions that allow instance methods to emit events.

### Application/MVC components

<a href="/images/post-images/introducing-oraculum/Oraculum%20Application%20Components.jpg" style="float:right;padding:1em;border:1px solid #ccc;margin:0.5em 1em;border-radius:0.5em;">
  <img src="/images/post-images/introducing-oraculum/Oraculum%20Application%20Components.jpg" style="width:200px;" alt="Oraculum Application Structure"/>
  <br/>
  Oraculum Application Structure
</a>

If you're already familiar with Chaplin, Oraculum's application components will look familiar to you. Aside from the implementation details in the underlying classes, and the named `definition` resolution, structuring an application in Oraculum almost exactly the same as Chaplin 1.1.x. However, just like in Chaplin, these components are completely optional and aren't required to use Oraculum.

The Oraculum application components are essentially nothing more than Chaplin 1.1.x's `Application`, `Composer`, `Composition`, `Controller`, `Router`, `Route`, and `Dispatcher`. These objects have been ported directly from Chaplin and wrapped in the FactoryJS container, having several of their implicit behaviors replaced by `mixin`s that perform the same (or similar) functions. As a result, these `mixin`s are available to any `definition` that wants to use them. They include `mixin`s for publishing/subscribing to a global event bus, making objects disposable in a memory-safe way, freezing objects after construction, and several others.

### Rendering tabular data

The tabular interface provided by Oraculum was introduced to solve the incredibly common problem of rendering `Collection`s in a tabular format with configurable columns, sortable attributes, etc. These components are actually an excellent candidate to be broken out into a separate Oraculum plugin in a separate repository, but we decided to keep them in core since it is such a ubiquitous use case. To this end, Oraculum provides several `mixin`s and `definition`s that allow a `Collection` to be rendered in a tabular fashion with a simple interface comparable to [Backgrid](http://backgridjs.com/), but more configuration-oriented.

### Behavior library

Oraculum comes with a large library of `mixin`s that aim to solve some of the most common use case problems when building a Backbone application. For example, if you need to bind an attribute of a `Model` to a particular element in a `View`, you may choose to use `DOMPropertyBinding.ViewMixin`, which has both a configuration and data-attribute based interface for one-way model -> element binding. If you want a `Model` to automatically invokes `fetch()` after it's been constructed, you could use `AutoFetch.ModelMixin`, etc. At the time this blog was written, here is the exhaustive list of available mixins:

`$ find src -type f | grep mixins`

  * `src/mixins/callback-provider.coffee`
  * `src/mixins/disposable.coffee`
  * `src/mixins/evented-method.coffee`
  * `src/mixins/evented.coffee`
  * `src/mixins/freezable.coffee`
  * `src/mixins/listener.coffee`
  * `src/mixins/middleware-method.coffee`
  * `src/mixins/pub-sub.coffee`
  * `src/models/mixins/auto-fetch.coffee`
  * `src/models/mixins/disposable.coffee`
  * `src/models/mixins/dispose-destroyed.coffee`
  * `src/models/mixins/dispose-removed.coffee`
  * `src/models/mixins/last-fetch.coffee`
  * `src/models/mixins/pageable-interface.coffee`
  * `src/models/mixins/remove-disposed.coffee`
  * `src/models/mixins/sort-by-attribute-direction-interface.coffee`
  * `src/models/mixins/sort-by-attribute-direction.coffee`
  * `src/models/mixins/sort-by-multi-attribute-direction-interface.coffee`
  * `src/models/mixins/sort-by-multi-attribute-direction.coffee`
  * `src/models/mixins/sortable-column.coffee`
  * `src/models/mixins/sync-machine.coffee`
  * `src/models/mixins/xhr-cache.coffee`
  * `src/models/mixins/xhr-debounce.coffee`
  * `src/views/mixins/attach.coffee`
  * `src/views/mixins/auto-render.coffee`
  * `src/views/mixins/cell.coffee`
  * `src/views/mixins/column-list.coffee`
  * `src/views/mixins/dom-cache.coffee`
  * `src/views/mixins/dom-property-binding.coffee`
  * `src/views/mixins/html-templating.coffee`
  * `src/views/mixins/layout.coffee`
  * `src/views/mixins/list.coffee`
  * `src/views/mixins/region-attach.coffee`
  * `src/views/mixins/region-publisher.coffee`
  * `src/views/mixins/region-subscriber.coffee`
  * `src/views/mixins/remove-disposed.coffee`
  * `src/views/mixins/static-classes.coffee`
  * `src/views/mixins/subview.coffee`
  * `src/views/mixins/templating-interface.coffee`
  * `src/views/mixins/underscore-templating.coffee`

### Aspect-oriented programming

Instead of relying on faux classical inheritance for modifying an object's prototype, Oraculum provides two utilities: `makeEventedMethod`, and `makeMiddlewareMethod`. These utilities create a wrapped version of an instance's target method that emit `:before` and `:after` events on a target event emitter. Oraculum also provides this functionality as a `mixin` via `EventedMethod.Mixin` and `MiddlewareMethod.Mixin`, which allow methods to be wrapped immediately after an object is constructed.

`makeEventedMethod` and `makeMiddlewareMethod` are how Oraculum provides shallow composition over deep inheritance, and they form the heart of Oraculum's [aspect-oriented programming strategy](http://en.wikipedia.org/wiki/Aspect-oriented_programming) for logic decoupling.

Conclusions
-----------

By [favoring composition over inheritance](http://en.wikipedia.org/wiki/Composition_over_inheritance), and combining the concepts of [factories](http://en.wikipedia.org/wiki/Factory_method_pattern), [flyweights](http://en.wikipedia.org/wiki/Flyweight_pattern), [mixins](http://en.wikipedia.org/wiki/Mixin), and [aspect-oriented programming](http://en.wikipedia.org/wiki/Aspect-oriented_programming) in a [dependency injection container](http://en.wikipedia.org/wiki/Dependency_injection), our code has become more stable, testable, dynamic, and portable, allowing insanely fast development of intensely complex browser-based applications.

Want to know more?
------------------

  * [Follow @OraculumJS on Twitter](https://twitter.com/OraculumJS)
  * [Check out the homepage](http://hackers.lookout.com/oraculum)
  * [Read the official documentation](http://hackers.lookout.com/oraculum/docs/README.md)
  * [Fork us on github](https://github.com/lookout/oraculum)
  * Or even better yet, [Come work with us!](https://www.lookout.com/about/careers)

<a href="https://github.com/egeste" target="_blank">
  <img align="absmiddle" style="border-radius:50%;" src="http://www.gravatar.com/avatar/42b61b891d0988c200a6cf301fa59212?s=48"/>
  - Steve (Egesté) Regester
</a>
