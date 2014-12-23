---
layout: post
title: "JRuby, sponsored in part by Lookout"
tags:
- jruby
- opensource
---


In [my](https://github.com/rtyler) time at Lookout we've had to grow in just
about every way imaginable. New products, new applications, new people, new
teams; growth in every dimension. In order to help engineering continue to grow
and be successful within our service-oriented infrastructure, we created a team
named "Core Systems" late this year. Among the team's responsibilities are
building tools, support engineering and maintaining systems foundational to a
number of our products such as [Kafka](http://kafka.apache.org),
[Storm](http://storm.apache.org), [Cassandra](http://cassandra.apache.org) and
so on. The systems we'll cover in later blog posts, but for this post we'll
focus on our support of the JRuby toolchain.

<a href="http://jruby.org" target="_blank"><img src="/images/jruby_large.png" alt="JRuby Jay!" width="200" align="left"
hspace="10" vspace="10"/></a>

Since the beginning Lookout has primarily been a Ruby-based engineering team.
As our deployment and tooling requirements have evolved, we've become more and
more invested in the JVM, which means [JRuby](http://jruby.org) and some of the
associated tooling (e.g. [jbundler](https://github.com/mkristian/jbundler))
are critical to the efficiency of our day-to-day work.

To help us support the JRuby toolchain internally we've welcomed [Christian
Meier](https://github.com/mkristian), a very talented JRuby hacker, to the
Lookout engineering team.

The benefits have almost been instantaneous, with a great number of changes
contributed by Christian on Lookout's behalf making their way into: [JRuby
1.7.17](http://jruby.org/2014/12/09/jruby-1-7-17.html), [JRuby
1.7.18](http://jruby.org/2014/12/22/jruby-1-7-18.html), [jbundler 0.7.0](https://github.com/mkristian/jbundler/tree/0.7.0) and [jruby-openssl
0.9.6](https://github.com/jruby/jruby-openssl/tree/v0.9.6).

This post doesn't aim to cover each commit and bug fix specifically, but will
provide a general overview of the areas of focus for our bug fixes and
improvements.


## JRuby

In JRuby [1.7.14](http://jruby.org/2014/06/24/jruby-1-7-13.html) and
[1.7.14](http://jruby.org/2014/08/27/jruby-1-7-14.html) the
[LoadService](http://jruby.org/apidocs/org/jruby/runtime/load/LoadService.html)
was refactored quite a bit to support operations such as: `File.exists?`,
`Dir['*']`, and `require 'somefile`. Despite best intentions, a number of
regressions did sprout up after the releases went public.


### File operations without "native" support

One series of regressions occurred when the native support from
[FFI](https://github.com/ffi/ffi#ruby-ffi-httpswikigithubcomffiffi-) would not
be properly loaded, and the fallback to Java failed. The cases where the FFI
libraries cannot load aren't necessarily bugs since JRuby supports explicitly
disabling loading of "native code" at runtime.

The failure of the Java-based fallback code would impact operations such as:
`jruby -S gem install my.gem` if `~/.gem/jruby/1.9` was not a directory.
Basically operations that would require:

 * `File.exists?`
 * `File.file?`
 * `File.directory?`
 * `File.executable?`

And so on. The JRuby test suite *does* contain many tests for these calls, but
it was only being run **with** native code enabled.

Fixing this class of bugs required not only fixing the tests to run with and
without native code enabled, but also required fixing a number of underlying
bugs in Java code.

### Testing on various JRuby "installations"

Previously much of JRuby's testing focus was on the JRuby which you might
install directly on your system. Because of the versatility and embeddability
of JRuby however, there are a number of different forms of "installations"
which look different than what you might get from [RVM](http://rvm.io):

* `org.jruby:jruby:pom. artifact
* `org.jruby:jruby:pom:noasm` artifact which is the above with asm classes
  repacked to org.jruby.org.asm packages
* `jruby-jars.gem` which is jruby.jar (from the install bundle) and
  org.jruby:jruby-stdlib:jar
* `org.jruby:jruby-complete:jar` (which we use in our [frankenwar and
  frankenjar](/2014/08/frankenwar-the-frankenjar-revisited) applications)


The biggest difference between those methods and the regular JRuby installation
(via rbenv, rvm, etc) is that the JRuby stdlib is inside a jar. Some features
like the "default gems" bundled with JRuby (e.g. `jruby-openssl`) do not work
when you set `jruby.home` to be `classpath:/META-INF/jruby.home`. This means
users could not pick the correct version of the `jruby-openssl` or even get a
nasty `MethodNotException` if they attempted to.

JRuby gives some (unnecessary) preference for the
`Thread.currentThread().getContextClassLoader()` and assumes that JRuby is
loaded via this classloader.  When running inside of a Frankenwar, or a Storm
topology this is not always going to be the case.

By adding some tests to exercise these alternative forms of JRuby
installations, a number of bugs were identified and corrected here as well.


### URI-like paths

As mentioned in the previous section, JRuby supports a number of different
forms of installations and can be easily embedded. A side-effect of this
functionality is that a "file path" in JRuby can be a number of different things:

* `classpath:my/path/to/a/file/on/the/classpath`
* `uri:bundle:0.17://my/path/inside/an/osgi/bundle`
* `uri:classloader://my/path/to/classloader/of/jruby`
* `file://my/path`
* `file://my.jar!/some/path/inside/this/jar`
* `my.jar!/some/path/inside/this/jar`
* `jar:file://my.jar!/some/path/inside/this/jar`

These URI-like paths can pop up very quckly when using constructs such as
`File.dirname(__FILE__)` from a file within the stdlib, JRuby kernel or even
code packed inside of a `.jar` file.

Sometimes these URI-like paths would need to be translated into a
`java.io.InputStream` like with `X509Store#add_cert`. The `LoadService`
refactoring already had some support for handling these use-cases but it was
error prone. The older mechanisms for doing this had already been dropped in
the next major development branch of JRuby (also known as JRuby 9000) but the
1.7.x branch needed some fixes too.

## jruby-openssl

### Concurrency problems

The `X509Store` had some synchronization issues discovered by some of our
applications running in production. While not an egregious bug, fixing it was a
good exercise for Lookout of getting production-level issues reported, triaged
and merging fixes into an open source project.

In this case the mutex for an X509Store instance was used by several classes
and needed to be fixed and synchronized properly.


### Certificate loading from URI-like paths

Related to the URI-like paths changes above, jruby-openssl also needed some updates to ensure that it could load certificates from within `.jar` files like `jruby-stdlib.jar`. Previously attempting to load these bundled certificates would always fail due to the aforementioned URI-like paths, but starting with JRuby 1.7.17 this behavior has been finally corrected.


### Misc. fixes

jruby-openssl did see a lot of refactoring around `Digest` and `Certificates`
prior to the 0.9.6 release. Ruby code was moved into Java, in addition to some
cases where code was using Bouncycastle directly instead of the Java Crypto
Extensions. After that regression there were some hairy test regressions which
needed to be fixed against both JRuby 9000 and the JRuby 1.7.x branch of
development in order to get the jruby-openssl 0.9.6 gem released.

----


Overall we're thrilled to be sponsoring JRuby development and helping "raise
all the boats" by getting bugs we're finding into the open source ecosystem.

I'm certainly looking forward to what we can do in 2015 as an [engineering
organization](https://github.com/lookout), building great products with great
technology, with regular open source contributions of course!


If you're interested in helping to build a great security company and enjoy
challenges, [join Lookout](https://www.lookout.com/about/careers). We've got a
number of positions open, not only on Core Systems but across the board in
engineering from Android, to iOS, Security, Operations, and 'Platform and
Infrastructure.'


- [R. Tyler Croy](https://github.com/rtyler)
