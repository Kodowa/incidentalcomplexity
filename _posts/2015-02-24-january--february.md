---
layout: post
title: "January / February: GUIs, time, joins and aggregates"
tags: []
---

Better late than never.

### People

Joshua Cole and Corey Montella joined us in the new year, bringing us up to a merry band of five.

### Editor

The goal for Eve is to have both textual and graphical code representations that stay in sync, so users can switch back and forth as they prefer. We've shown textual prototypes before - this is our first graphical prototype.

<iframe width="600" height="450" src="https://www.youtube.com/embed/EfClpyk0jhQ" frameborder="0" allowfullscreen></iframe>

The core visual metaphor is cards. Each card represents a single chunk of code. Cards for specialised domains like UI have their own specialised representations and editors. We intend to have pages of cards for organising large projects - so a server project might have a network page that contains code cards for some of the important views as well as UI cards with graphs of network activity.

The table editor is functionally complete - you can write any Eve program using it - but the UI for adding aggregates was designed for an older version of the language and can behave weirdly.

The UI editor is still very experimental and has only been used to build simple forms. We still have to add support for dynamic contents (eg lists), styling and layout.

### Aggregates

Our current implementation of aggregates is very restricted. The aggregate functions can only return a single result and are limited to aggregating over the results of index lookups (eg 'count the number of events which have id="foo" and label="click"').

We can fix these problems by adding first-class sets. We can restrict the usage of these sets so they never have to be fully materialised in the runtime - they only exist to make the semantics clear. They allows us to write complex aggregates using the same primitives we use for joins. They can also express negation, existentials and temporal queries like 'find all consecutive events which are more than 5 minutes apart'.

We are currently working through the details in an isolated implementation and intend to include it in the next iteration of the language. In the next post we will lay out the exact semantics and hopefully show some working examples.

### Time

We experimented with first-class time intervals as described in [Time and Relational Algebra](http://www.amazon.com/Time-Relational-Theory-Second-Management/dp/0128006315/ref=sr_1_1?ie=UTF8&qid=1424816894&sr=8-1&keywords=date+temporal+relational). Unfortunately this caused huge performance problems, partly due to the lack in incremental evaluation in the current runtime and partly because of the lack of custom value types in javascript.

We also experimented with various different ways of representing time and change in Eve. Within the same program we can express timeless aggregates (like 'x is the number of unique click events') or more imperative updates (like 'given a click at time t, the new x is the previous x plus 1') without ever needing destructive change. It won't be clear which patterns work best until the runtime is able to support much bigger programs.

### Joins

We implemented the [Tetris algorithm](http://arxiv.org/abs/1404.0703). Along the way we came up with some nice tricks for packing bitwise tries efficiently (rather than one node per bit) and also found some optimisations for Tetris that avoid most of the overhead in the recursive search stage.

Using dyadic gaps, as the paper suggests, is prohibitively expensive. The algorithm ends up generating thousands of gaps to join a dozen rows. The next experiment is to use arbitrarily sized gaps and treat resolution as a memoization problem inside the gap index rather than building it into the join algorithm itself.

We *may* also have a way to make Tetris work for arbitrary types (eg strings) as opposed to the fixed size bitstrings used in the paper.

Tetris is still a good candidate for our join algorithm if we can work out these issues. Otherwise, some of the algorithms described in previous posts have produced reasonable results and are fine fallbacks.

### Control

While running in the browser is a requirement for Eve, it's always been clear that using javascript directly was not a long-term option. So many of our implementation problems come down to lack of control over data layout. For Eve we need to implement:

New types (like intervals) - but there is a space overhead of 24 extra bytes per object
Polymorphic comparisons - but dispatching on typeof is slow
Cache-friendly indexes - but it's hard to store multiple js objects sequentially in memory
Radix tries - but converting strings to bytes is slow

We also want to be able to distribute native code for mobile devices and use real threads on servers. Lastly, there is some benefit to using reference-counting for the indexes so that we can avoid copying nodes when we know we have sole access.

We ruled out C++ and D on aesthetic grounds - we have a preference for small, simple languages that we can understand completely.  Rust wins points for safety and abstraction but the toolchain is not nearly as mature and there are issues that currently prevent compiling with Emscripten. C gives us less support in the language but is much more future-proof at the moment.

### Roadmap

Josh is continuing to work on improving the new editor. Corey is working out the details of the new aggregate implementations. Chris and Jamie are away at [Hacker School](http://hackerschool.com/) where they will experiment with porting chunks of Eve to both C and Rust.
