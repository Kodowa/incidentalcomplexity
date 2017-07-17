---
layout: post
title: "Dev Diary (December 2016 - January 2017)"
author: "Corey Montella"
tags: []
---
_(Eve is a new programming language, and this is our development blog. If you’re new to Eve, [start here](http://play.witheve.com))_

After a long journey during the year, we closed out 2016 on a high note with the release of Eve Alpha v0.2.3, and some very exciting community engagement. We're looking to keep the momentum going in 2017, so let's take a look at what we've been doing the past couple months.
December was really a time of reflection, as we had to consider what to do with the feedback we received following the v0.2 release. Ultimately, it was pretty clear (particularly in [this thread](https://groups.google.com/forum/#!topic/eve-talk/vR-4y2kJv4Q)) that general feedback aligned along three directions:

1. Database explorer/visualizer
2. Extensibility
3. Real-world examples

Ideally, we'd like to build a database explorer for Eve in Eve itself. Unfortunately, the v0.2.x version of Eve lacks some features that would make this possible (like the ability to introspect the compiler, treating code as records). This drove us to embark on an extensive refactor (happening on the [refactor-runtime brach](https://github.com/witheve/Eve/tree/refactor-runtime)), which will accomplish a couple of goals simultaneously:

- Making the runtime incremental. The v0.2 runtime wastes a lot of time doing unnecessary computation. An incremental Eve runtime will be smarter about how it uses resources, computing values only when they need to be updated. This kind of optimization is already showing an order-of-magnitude improvement in performance, making Eve more practical for more resource-intensive programs.

- Making the runtime extensible, meaning new ways to interface with Eve (see the Javascript DSL below, for example), and the ability to get data into and out of Eve. The Javascript DSL in particular means that you’ll be able to integrate Eve into an existing project seamlessly. From there, you can use as little or as much Eve as you want.

- Making the runtime more approachable. We're going to be re-structuring the directory layout, as well as extensively documenting each component. Furthermore, we'll be building the runtime through pull requests, so we have a complete history of its development.

#### DSL
As part of the refactor, we've built a Javascript DSL to write Eve code. The following block is 
Javascript, but it describes an Eve block:

```javascript
let prog = new Program("test");
 prog.block("simple block", ({find, record, lib}) => {
    let a = find("person");
    let b = find("person");
    a.age > b.age;
    let result = a.age + b.age;
    return [
     record({age1: a.age, age2: b.age, result})
    ]
 });
```

If you're familiar with the Eve syntax, this is equivalent to:

```eve
search
  a = [#person]
  b = [#person]
  a.age > b.age
  result = a.age + b.age

commit
 [age1: a.age, age2: b.age, result]
```

You can embed these blocks in any Javascript code, allowing you to seamlessly incorporate Eve into existing projects. Documentation is forthcoming, but in the meantime you can see more examples of the DSL in practice in the [tests folder](https://github.com/witheve/Eve/tree/refactor-runtime/test) of the refactor branch.

#### Eve Examples

While the runtime is being refactored, we will be releasing some larger-scale examples of Eve programs. We'll start with some example mini-programs, each of which illustrate a component of a larger, more complete app. These smaller examples will be hosted on the [witheve/eve-examples](https://github.com/witheve/eve-examples) repository.

Development of the first large-scale example is going on in the [witheve/mamarob](https://github.com/witheve/mamarob) repository. This example is a food truck management app, which allows a user to manage a food truck menu, take orders, accept payments, and integrate with social media. This app should cover everything you need to know in order to build your own complete Eve application. It's not quite finished yet, but we'll let you know when it's complete! 

After that we have a couple more projects in mind, including a robot car and an operations management example. We’ll keep you updated as work on those commences.

### Community

#### Eve Around the World

We are very happy to see meetups being organized by members of the Eve community. In January we saw two meetups, one in Pittsburgh, PA and another in Copenhagen, Denmark.

**Pittsburgh, January 17**

Ruben Niculcea organized a [meetup](https://www.meetup.com/Pittsburgh-Code-Supply/events/235492181/) in Pittsburgh Pennsylvania, hosted by Pittsburgh Code & Supply Co. The presentation was recorded, and you can view it below: 

<iframe width="560" height="315" src="https://www.youtube.com/embed/GRSFMLc_AfM" frameborder="0" allowfullscreen></iframe>

Or, if you prefer you can view Ruben's slides directly [here](https://docs.google.com/presentation/d/1honhRiF6TVWz9kGh90WG3hKWJmgRe9MhyRGoYrOFgFo/edit?usp=sharing)

You can read some follow-up discussion, as well as view the results of the survey mentioned in the video [here](https://groups.google.com/forum/#!topic/eve-talk/oBIq-KgAjSQ). 

**Copenhagen, January 25**

The Eve meetup group in Copenhagen, founded by Zubair Quraishi, held its [second meetup](https://www.meetup.com/evecph/events/237131433/) in January. There was a [followup discussion](https://groups.google.com/forum/#!topic/eve-talk/zAXkukWgEXM) on the mailing list as well.

If you'd like to host an Eve meetup in your area, let us know how we can help. We're very encouraged to see this kind of spontaneous engagement, so thanks again to Ruben and Zubair for making these meetups a success!

#### Eve Around the Web

[Liron Shapira](https://twitter.com/liron), finished his six-part series on Eve with a post titled ["Eve will be perfect for realtime apps"](https://hackernoon.com/why-eve-will-be-perfect-for-realtime-apps-92b965b80ad#.sl1fmo2hv
).

Finally, William Taysom wrapped up his [_Puzzles and Paradoxes_ series](https://groups.google.com/forum/#!searchin/eve-talk/%22Puzzles$20$26$20Paradoxes%22%7Csort:date) on the mailing list. In total, there are 24 puzzles, which highlight some of the surprising or interesting corners of the Eve language. 
