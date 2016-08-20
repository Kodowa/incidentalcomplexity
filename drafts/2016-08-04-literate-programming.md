```
---
layout: post
title: "Literate Programming and Eve"
author: "Corey Montella"
tags: []
---
```



### What is Literate Programming?


Literate programming is best explained by its creator Donald Knuth. In his [influential paper][2], he explains: 

> The practitioner of literate programming can be regarded as an essayist, whose main concern is with exposition and excellence of style. Such an author ...  strives for a program that is comprehensible because its concepts have been introduced in an order that is best for human understanding, using a mixture of formal and informal methods that reinforce each other.

This description fits with the ethos of Eve - that programming is primarily meant to communicate with other humans, not the computer. You'll notice the above Eve program is actually written in two languages: Markdown, used to format the prose; and Eve, which is delineated by standard Markdown code blocks. Only the content within a block is compiled, while everything else is disregarded as a comment.

### Literate Programming and Eve

We've been toying with literate programming in various forms for a while now. The clearest example was the EveMarkdown experiment from January. This incarnation of Eve to the form of a rich document editor that could be used to create reactive documents (documents that are backed by data and update as the underlying data changes).  


We haven't written many Eve programs so far, but of the larger ones our average LOC per block is 8 with a standard deviation of 3.8 lines.


We can actually do a step better than literate programming as presented by Knuth. In his 

### Why Practice Literate Programming?

Writing code this way has several properties that result in higher quality programs:

- Literate programming forces you to consider a human audience. While this is usually the first step in writing any document, programming treats the machine as the primary audience. For an Eve program, the audience might be your collaborators, your boss, or even your future self when revisiting the program in a year. By considering the audience of your program source, you create an anchor from which the narrative of your program flows, leading to a more coherent program.

- The human brain is [wired][3] to engage with and remember stories. Think back to a book you read (or maybe a show you watched) last year. You probably remember in great detail all of the characters and their personalities, the pivotal moments of the plot, the descriptions of the various settings, etc. But how much can you remember of a piece of code you haven't looked at for year? Literate programming adds another dimension to your code that will help you keep more of your program in working memory.

- "if you program in a team that does code reviews, the team can see your approach to the solution and the reasons. They can critique your work at a more profound level. If the code review happens before accepting the change commit, the quality of the code is higher." {{{REPHRASE THIS}}}

- "Third, code lives. Sourceforge is a gravesite of hundreds of thousands of programs that have died because the authors are no longer maintaining the code. New users are confronted with a source tree of tiny files which they are unable to understand and therefore unable to modify and maintain." {{{REPHRASE THIS}}}

- Since Eve code blocks can be arranged in any order, literate programming encourages the programmer to arrange them in an way that makes narrative sense. Code can have a beginning, middle, and end just like a short story. Or like an epic novel, code can have many interwoven storylines. Either way, the structure of the code should follow an order imposed by a narrative, not one imposed by the compiler.

- It's often said that you don't really understand something until you explain it to someone else. Literate programming can reveal edge cases, incorrect assumptions, gaps in understanding the problem domain, and shaky implementation details before any code is even written.

Literate programming is a first-class design concept in Eve. We will be writing all of our programs in this this manner, and will encourage others to do the same for the reasons above. That said, there is nothing in the syntax that specifically requires literate programming; you can write your program as a series of code blocks without any prose, and it will be perfectly valid.

### Criticisms of Literate Programming

The idea behind literate programming has been around for decades, and yet we haven't seen widespread adoption of this practice. Given the outlined benefits of literate programming, why is it that more people aren't writing code this way?

- Current tools are bad

- Languages are bad

- documentation gets out of sync with code, so why bother?

- code has multiple audiences

- It's hard to maintain literate programs


[1]: https://en.wikipedia.org/wiki/Literate_programming
[2]: http://www.literateprogramming.com/knuthweb.pdf
[3]: https://blog.bufferapp.com/science-of-storytelling-why-telling-a-story-is-the-most-powerful-way-to-activate-our-brains
[4]: http://www.perl.com/pub/tchrist/litprog.html