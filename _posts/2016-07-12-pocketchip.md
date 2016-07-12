---
layout: post
title: "Eve on a PocketCHIP"
author: "Corey Montella"
tags: []
---

In our [latest Dev Diary](http://incidentalcomplexity.com/2016/06/30/apr/), we showed off the latest Eve REPL. Well, not too long after, [Mark R. Hacker](https://twitter.com/markrhacker1) managed to get Eve and the REPL running on his Pocket Chip.

<blockquote class="twitter-tweet" data-lang="en">
  <p lang="en" dir="ltr">
    <a href="https://twitter.com/nextthingco">@nextthingco</a>
    <a href="https://twitter.com/hashtag/PocketCHIP?src=hash">#PocketCHIP</a>
    <a href="https://twitter.com/ibdknox">@ibdknox</a>
    <a href="https://twitter.com/with_eve">@with_eve</a>
    <br>
    Eve running on my PocketChip. :-)
    <a href="https://t.co/snLogzeRbS">pic.twitter.com/snLogzeRbS</a>
  </p>
  &mdash; Mark Robert Hacker (@MarkRHacker1)
  <a href="https://twitter.com/MarkRHacker1/status/751147433997004800">July 7, 2016</a>
</blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8">
</script>

Check it out!

![EvePocketChip]({{ site.url }}/images/pocketchip1.jpg)

###### (image courtesy Mark R. Hacker)

I wasn't very familiar with this device before now, but apparently it's an ultra-low-cost computer akin to the Raspberry Pi. The CHIP sports a 1GHz SOC, and 512 MB of RAM, so it's not exactly an anemic machine. Still, it's the weakest machine to ever run Eve, and to my knowledge, this represents the first time that Eve has been compiled to ARM. Mark reports that since the CHIP runs Debian, all he had to do was follow the standard build instructions to compile natively to ARM on the device. However, he noted that starting Eve actually took 10 minutes, so you need some patience. Despite this, Mark reports that the REPL runs fine in the Iceweasel browser.

![EvePocketChip]({{ site.url }}/images/pocketchip3.jpg)

###### (image courtesy Mark R. Hacker)

Mark is currently working on getting this version of Eve working on his PocketCHIP, so we'll check back with him later for an update on his progress.