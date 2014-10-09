---
layout: post
title: "Light Table and Apple's Swift"
author: Rob Attorri
tags: []
---

In case you missed it in all the craziness of WWDC last week, Apple launched not only a new language, Swift, but also a new set of tools in XCode called "Playgrounds". According to [Chris Lattner][sabre], the director of tools at Apple, Light Table heavily influenced their work. Heck, even the name sounds [familiar][playground]... I can't think of higher praise than seeing Light Table begin to influence the broader industry.

Lattner wants to make programming more interactive, approachable, and fun. Good. It's about damn time that we do.

The more people that come around to this way of thinking the better. But starting to work together would be best. It's no accident that so many professionals in other fields aren't just averse to programming, they actively do not want to learn how.

### Pulling down the walls.

A little over 2 years ago, Chris showed me the original prototype for Light Table, suffixing his demo with, "This is big." When faced with something big, I think everyone sees different possibilities in the expanse of potential. Although our initial application to Y Combinator was for an idea pertaining to medical software, the prototype that would become Light Table was too compelling to ignore. Why? Recall what Marc Andreessen wrote: Software is eating the world.

As a scientist who worked on stem cells, not IDEs, that rings quite true. Patient records and experimental protocols lived on the hospital intranet, Excel handled most of the number crunching (sometimes Matlab, R, or SPSS got involved), and email was indispensable for collaboration. An immense amount of frustration came out of the subpar tools that governed my work and the inability to swiftly remedy my own situation. I felt, as I suspect many other scientists might, that the barrier to entry for writing software is too high.

Light Table was the first step towards lowering that barrier and scratching my own itch, not because it could magically make me a Clojure programmer overnight, but because it embraced a better design philosophy, one that might finally allow me to start harnessing the power of my own computer.

### So what does better design look like?

And where should we be taking our inspiration from? I can't give you a definitive answer for those questions, but I can share with you what's impacted us the most. It's impossible not to talk about [Bret Victor's ideas][bret], and after spending so much time in Excel in my past life, I fancy his talk on [Drawing Dynamic Visualizations][dynamic] the most.

My other personal favorite inspiration comes from [HyperCard][hypercard], something so dramatically ahead of its time that it was able to make programmers out of normal users without them even knowing it. Cyan used HyperCard to write Myst, which is probably my favorite game series ever made. With only very slight modifications and different marketing, it might have become the modern internet. Its failure belied all the things it got right that made it whimsical, fun, and quietly powerful, all without alienating less technical users. This is why I love tools like Bret's visualizations or HyperCard, that are more than just technically impressive - they're desirable. They draw you in whether you call yourself a developer or not and give you the desire to create. Don't design software so users begrudgingly learn to use it because it's the only means to the end they need to reach, design it so they look at it and wonder, "Wouldn't it be cool if I could..."

### "I can do that."

That's the reaction we wanted people to have when we first started with Light Table. We're humbled that a few guys with wild ideas can inspire the team at Apple who worked on Swift, and we're even more proud that we've managed to attract a fantastic open source community who contributes so much to this vision.

The past couple years have been quite the learning experience, and our thinking has evolved since those early versions of the playground. We've come to understand that the future revolves around making computers simple, easy, and desirable to control. And in order for us to get there, programming has to change quite a bit. The work that remains ahead of us is daunting, but we can't wait until everyone can go from "wouldn't it be cool if..." to "I can do that."

-Rob

A list of some of our favorite influences:

* [Bret Victor's Drawing Dynamic Visualizations][dynamic]
* [Bret Victor's Inventing on Principle][principle]
* [HyperCard][hypercard]
* [Eskil Steenberg's LOVE Tools][love]
* [Gary Bernhardt's Wat][wat]

[sabre]: http://nondot.org/sabre/
[playground]: /2012/06/24/its-playtime/
[bret]: http://worrydream.com/
[dynamic]: http://vimeo.com/66085662
[principle]: http://vimeo.com/36579366
[hypercard]: http://arstechnica.com/apple/2012/05/25-years-of-hypercard-the-missing-link-to-the-web/
[love]: https://www.youtube.com/watch?v=DPIA2g8T6Hw
[wat]: https://www.destroyallsoftware.com/talks/wat
