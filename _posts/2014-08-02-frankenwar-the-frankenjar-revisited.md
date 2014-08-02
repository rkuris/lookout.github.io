---
layout: post
title: "Frankenwar: the Frankenjar revisited"
tags:
- sinatra
- ruby
- jruby
- rake
- java
- releng
---

Last year I wrote about Lookout's use of the
[frankenjar](/2013/08/deploying-the-frankenjar/), a self-contained archive of
an entire [JRuby-based](http://jruby.org) artifact. The benefits of the
frankenjar were many:

 * An entirely self-contained Ruby application without reliance on external
   gem or other dependencies.
 * A simple to re-use, test and deploy executable file.
 * A single trackable artifact to follow through the build and release system.
 * By using Rake as a built-in task runner, a single artifact can be used for
   many different, but related, functions. Such as running database migrations
   for the web application contained within the file.

<img src="/images/jruby_large.png" alt="JRuby Jay!" width="200" align="right"/>

The frankenjar approach does have its downsides however. For running the web
server, we were invoking a Rake task which would start up an embedded
[Puma](http://puma.io) instance. This means relying on Puma for managing
request workloads itself, configuring it to use the appropriate number of
request handling threads. It also means that if the `.jar` is running, you have
a web server, if the `.jar` isn't you have *nothing.*

At a more fundamental level, using Puma or any Ruby-based
([Rack](http://rack.github.io/)) webserver, also means that there **will** be
additional performance overhead. A Rack-based server means lots of string
handling in Ruby (not cheap) but more importantly it means that the application
will be using **Ruby sockets**.

### Ruby is not the issue, dude.

The problem with Ruby sockets is not something specific to Ruby at all, it's
just that the `Socket` interface in Ruby is a binding on top of the traditional
[BSD Socket](http://en.wikipedia.org/wiki/Berkeley_sockets) interface. This
traditional interface provides *blocking I/O* primitives such as `select(2)` or
`poll(2)` which make building highly-scalable (read: concurrent) network
servers difficult.

The JVM provides a much higher performance Socket interface with its "[New
I/O](https://en.wikipedia.org/wiki/New_I/O)." Mostly referred to as "NIO", it
abstracts some underlying evented I/O primitives provided by modern operating
systems through `kqueue(2)` (Mac OS X/BSD) or `epoll(7)` (Linux). You can use
NIO from Ruby with the [nio4r](https://github.com/celluloid/nio4r) gem but I've
not yet seen a Rack server outside of [Reel](https://github.com/celluloid/reel).


## Servlets

A number of the short-comings of the frankenjar approach can be easily
mitigated by using a Java Servlet Container such as Tomcat or Jetty. For the
unfamiliar, Java has a specification similar to Ruby's Rack for defining a
server interface, the [servlet
specification](https://en.wikipedia.org/wiki/Java\_Servlet#History). Many
servers implement this specification often times adding additional enhancements
around the servlet API itself. This allows you to take a web archive (a.k.a. "a
.war") and drop it into any number of different servlet containers.

Servlet containers like [Tomcat](http://tomcat.apache.org/) usually use NIO by
default or at least allow it to be configured. They also typically provide some
amount of workload management for your applications. Tomcat for example can be
configured to keep a threadpool for request handlers active with a minimum, maximum and
minimum spare threads in it. Thereby allowing the server to dynamically, within
bounds, scale up and down to handle varying request workloads.


## You got chocolate in my peanut butter

Fortunately the same tool we use for creating the frankenjar,
[warbler](https://github.com/jruby/warbler), can create a franken**war**. The
process is largely the same as the
[frankenjar](/2013/08/deploying-the-frankenjar/), with some minor differences.


Like the frankenjar, the majority of the interesting configuration is in the
**config/warble.rb** file:

    # Disable Rake-environment-task framework detection by uncommenting/setting to false
    Warbler.framework_detection = false

    # Warbler web application assembly configuration file
    Warbler::Config.new do |config|
      # Turn the `runnable` feature on to make sure that we can have an
      # executable war file
      config.features = %w(runnable)
      # Include the app/ directory in the .war
      config.dirs = ['app']
      # Ensure that we have our Rakefile available too
      config.includes += FileList['Rakefile']
      config.jar_name = 'hellowarld'
      config.override_gem_home = false
    end

In the frankenjar we relied on a Rake-based runner to be the first file in the
project's `bin/` directory. Warbler would then use this executable to drive the
executable `.jar` file. Another Lookout engineer, [Marc
Chung](https://github.com/mchung), discovered some undocumented functionality
in Warbler's [war bootstrap
code](https://github.com/jruby/warbler/blob/master/ext/WarMain.java) which
allows for executing binaries from within the `.war` file with the `-S`
argument, e.g.: `java -jar my.war -S rake`.

This hidden functionality is really what makes the frankenwar feasible.

----

*Note:* if you intend to use Rake in this fashion, make sure it's in the
"default" group in your Gemfile so Warbler will include the gem in the package.

----

Of course, since this is intended to be a web application, we must define a
`config.ru` file which will help indicate to Warbler that it should build a
`.war` file:

    # Add our current working directory to the LOAD_PATH
    $LOAD_PATH.unshift(File.expand_path(File.dirname(__FILE__)))

    require 'rubygems'
    require 'app/hellowarld'

    map '/' do
      run Sinatra::Application
    end

With the `config/warble.rb` and `config.ru` files in place we can build and
execute our [hellowarld](https://github.com/rtyler/hellowarld) application:

    % bundle exec warble war
    rm -f hellowarld.war
    Creating hellowarld.war
    % java -jar hellowarld.war -S rake hello
    Hello there
    %

If we were to then drop our `hellowarld.war` into a servlet container, we'll
get "Hello Warld" when we access the `/hellowarld` URI.

(**Note:** a fully functional example of this can be found in [this
hellowarld](https://github.com/rtyler/hellowarld) application)


----

A Warbler-generated frankenwar in hand, many of the original needs for the
franken*jar* are met:

 * An entirely self-contained Ruby application without reliance on external
   gem or other dependencies.
 * A simple to re-use, test and deploy executable file.
 * A single trackable artifact to follow through the build and release system.
 * A versatile application which can run application-related tasks such as
   database migrations.


All with the added performance and support that Java servlet containers can
offer:

 * Hot-deployment of a `.war` application. In many cases this is just dropping
   a `.war` file in a special directory on the application.
 * Higher concurrency through the server's use of NIO.
 * Lower per-request overhead since much of the request parsing occurs in Java
   within the server.

Previously I had proclaimed that the frankenjar was "the easiest Ruby app
deployment ever." I'm ready to rescind that statement.

Not having learned my lesson about making wild proclaimations, I can now
confidently say that:

The **frankenwar** is the easiest, and highest performance, Ruby app deployment
ever.



*- [R. Tyler Croy](https://github.com/rtyler)*
