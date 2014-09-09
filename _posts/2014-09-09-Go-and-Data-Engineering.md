--- 
layout: post 
title: "Capturing Change Data with Mudskipper"
author: rothrock 
tags: 
- MySQL
- C 
- Go
--- 

Engineering at [Lookout](https://github.com/lookout) diiferentiates itself with
its embrace of a heterogeneous coding environment.  Although Ruby usually
carries the day for server-side components, we write in Java, Objective-C,
Bash, R, C, Scala, Python, and Go.

One such project built in Go we're open sourcing today:
[Mudskipper](https://github.com/lookout/mudskipper), a utility for extracting
change-data events from MySQL binlogs. Currently at Lookout, Mudskipper
captures about 20 million events per day from 7 MySQL tables.

When I set out to build Mudskipper, I had two goals in mind:

  1. Find a way to capture change-data for select tables in our MySQL databases.
  2. Explore novel features in Go like [channels](https://golang.org/doc/effective_go.html#channels) and [goroutines](https://golang.org/doc/effective_go.html#goroutines).

What Is Change-Data?
--------------------

Change-data is metadata stored in tabular format that embodies all the changes to a database table during a window of time.
It differs from a log mainly in that it is stored in a row-column format meant for querying by a database system.
For example, a banking application might have an account_balance_audit table that keeps track of all the changes to the account_balance table.
Historically, the _audit table would be populated by triggers on the table it is shadowing.
This puts extra workload on the database and introduces more complexity in the application.

Mudskipper Takes A Different Approach
-------------------------------------
With row-based binary logging enabled, Mudskipper can scan the binlog stream and selectively extract change events.
Mudskipper decouples change-data capture from the event or app that caused the change.
Decoupling lets us spread the data capture effort across multiple, independent processes and possibly across many CPUs.
The banking application no longer needs custom logic and data definitions for each table.
Auditing the application is further isolated from the app itself.

Why Go?
-------
First, it''s fun to be on the avant-garde of new technology.

Second, My background is in C, and Go feels like a well-planned, 21st century version of C.

Third, I have never much cottoned to object-oriented programming concepts.
There are no classes in C. No classes, means no inheritance.
Object construction is simply calling new() on a defined type. No constructors, factories, or destructor functions.

Instead, Go implements OO concepts with a very light touch that I really appreciate.

  - Modules live in a hierarchical namespace.
  - Function attached to structs implements data hiding.
  - Interfaces focus the developer on communication between objects.

Finally, goroutines and channels offered a simple approach to distributing the workload over more CPU cycles.
Goroutines and channels also encourage a coder to think more about loose coupling and tight cohesion.

In Mudskipper, the binlog scanning and extraction is separated from the effort of writing the output.
Moreover, the scanning process is implemented as a dynamic pool of goroutines.
When lots of binlogs show up quickly, the code easily brings on more scanners.
They spin down when the workload slacks.

Mudskipper is very much a work in progress. I invite you to check out the code, offer comments, and send pull requests.

*- [Joseph (Beau) Rothrock](https://github.com/rothrock)*
