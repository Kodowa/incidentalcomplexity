---
layout: post
title: "Dev Diary (September 2016)"
author: "Corey Montella"
tags: []
---

_(Eve is a new programming language, and this is our development blog. If you’re new to Eve, [start here](https://github.com/witheve/Eve))_

When we last left off in [August][0.1], we were iterating on the just-released community preview. In September, we improved the editor, worked on documentation, made some adjustments to the language, and also put Eve up on the Web.

[0.1]: http://incidentalcomplexity.com/2016/08/31/august/

### Platform

#### Editor

A lot of work has gone into imagining the editing experience for Eve. Here is [Tic-Tac-Toe][2.0] opened in the editor:

![Eve editor]({{ site.url }}/images/editor.png)

On the left we display the table of contents, in the center is the rendered program, and on the right is an area for comments. Let's look at each one in turn:

[2.0]: https://github.com/witheve/Eve/blob/master/examples/tic-tac-toe.eve

##### Table of Contents

The table of contents (TOC) is one of the more interesting features of the editor. The TOC is generated using the headers in the document, which allows you to bookmark key sections of the program. Clicking on an entry in the TOC moves the corresponding section into focus, so this is a nice way to navigate through a larger program.

But we can go one step beyond this. One criticism of [literate programs][2.1] is that an ordering that makes narrative sense doesn't necessarily make sense while editing a program. For example, in Tic-Tac-Toe, we talk about winning the game in two entirely different sections. That's okay - instead of jumping around between sections, we can display only the sections we care about:

![Eve editor elision]({{ site.url }}/images/markdown-elision.gif)

Now the editing experience is customized to the task at hand. This opens up some interesting possibilities, like being able to save and share these views, or generating customized views based on a query. For example, during a code review you might want to see all blocks modified by Steve in the last week. Something like that is further down the line, but it's nonetheless an interesting possibility.

[2.1]: https://witheve.github.io/docs/handbook/literate-programming/

##### Document Editing

We've improved the editing experience with in-browser WYSIWYG support. One of the slight drawbacks of being [CommonMark compatible][2.2] is having to type code fences for every block of code. Not only is this sometimes tedious, but backticks and tildes are not even on some international keyboards. Fear not: if you write code in our editor (and we hope you will, but you don't have to) you can create a code block with a nifty shortcut. Furthermore, since we render Markdown in the editor, the code fences don't clutter your code.

![Eve editor]({{ site.url }}/images/editor2.png)

[2.2]: http://incidentalcomplexity.com/2016/08/31/august/#eve-and-markdown

##### Comments

Finally, the editor has an area for comments. These aren't code comments (although maybe they could be, we just don’t know how well that would work yet). Rather, you can think of this area as a place for collaborative comments from team members, similar to Google Docs. This area can also contain warnings and errors from the Eve compiler:

![Eve editor comments]({{ site.url }}/images/comments.jpg)

Here, you can see a compiler error displayed next to the block that generates it. It's not hooked up here, but we've shown previously how errors like this can be [fixed automatically][2.3] with the click of a button. We'll be incorporating more of those ideas in the next month.

[2.3]: http://incidentalcomplexity.com/2016/08/03/july/#error-handling

#### Eve on the Web

Another big development this month is a version of [Eve written in TypeScript][3.0]. This has the notable advantage over the C/Lua version of being able to work on every platform, including Windows. Also, in this version we've started to add some debugging tools. Here's an example of a performance report, which displays performance statistics for each block:

![perf]({{ site.url }}/images/perf.png)

We've also added some [testing facilities][3.1] for language-level sanity checks:

![testing]({{ site.url }}/images/tests.png)

We'll have a lot more to talk about on this front in a couple of weeks, so stay tuned.

[3.0]: https://github.com/witheve/Eve/tree/ts-merge
[3.1]: https://github.com/witheve/Eve/tree/ts-merge/test

#### Word Choice Adjustments

In August, we tentatively settled on the term "context" in place of "bag" to describe a collection of facts. After testing "context" for a couple weeks, we came to the conclusion that it just wasn't the right word; it didn't adequately describe the underlying concept, and thus it was misleading to some people. Instead, we've decided on "database", which is a much more obvious and standard choice. We've found less confusion in using this word, so it seems like a win so far. Let us know what you think over at the [syntax RFC][4.0].

We also decided to rename `match` to `search`. Match was supposed to evoke the idea of pattern matching against data. This wasn't clear to some people, and we've found that search is more intuitive.

```
bag -> context -> database
match -> search
```

[4.0]: https://github.com/witheve/rfcs/issues/4

#### Changing the meaning of `@`

We've been using `@` as a shortcut for the name attribute on records. For example, instead of writing `[name: "Corey"]` you could write `[@Corey]`. However, we’ve decided to remove this shortcut for several reasons. First, after writing more applications in Eve, we've found that the name attribute isn't actually commonly used, so a shortcut doesn't save that many characters. Furthermore, the tag shortcut ( `#` ) can be used instead with the same effect. Finally, the semantics of `@` in Eve are different from `@` in other contexts; whereas `@` resolves to a unique username on Twitter and Github, `@` in Eve could match against multiple records. After all, name is just an attribute like any other.

Therefore, we're removing that sugar, and instead we'll be using `@` exclusively to reference databases. If you recall, databases are containers of facts. You can perform actions, like searching or binding, on one or more databases. With the removal of the name shortcut, we can now treat databases as first-class citizens of the Eve language:

```
search
  db = if [#foo] then @db1
       else @db2

bind db
  [#bar]
```

Here, we're assigning a database to a variable based on a condition. If the condition is satisfied, `[#bar]` is bound to `@db1`. If the condition is not satisfied, `[#bar]` is bound to `@db2`.

### Community

We've been a little quiet on the blog, but community efforts are still abound! Most of the community-focused work in September went toward building out our documentation and guides. 

#### Handbook

![Eve Docs]({{ site.url }}/images/docs.png)

We've put a lot of work into [witheve/docs][6.0]. The handbook now has a lot more content, and can be [browsed on the web][6.1]. In October, we'll be making improvements with a focus on styling and completeness.

[6.0]: https://github.com/witheve/docs
[6.1]: https://witheve.github.io/docs

#### Quick Start Guide

The quick start guide is coming along nicely. It's gone through a few iterations of user feedback, and we'd like to hear from a wider audience. The guide starts by showing you "Hello World!" in Eve, and from there slowly introduces language concepts until you have the tools to write a simple web app. We hope you'll [give it a read][7.0] and let us know how it can be improved.

[7.0]: https://witheve.github.io/docs/guides/quickstart/

That's all for September. But as I said earlier, we have some exciting stuff in the pipeline for October. As always, we hope you give Eve a try and tell us what you think on the [mailing list][8.0]!

[8.0]: https://groups.google.com/forum/#!forum/eve-talk 