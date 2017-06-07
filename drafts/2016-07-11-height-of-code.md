## Height of Code

Have you ever felt a sense of satisfaction holding something you built with your own hands? The joy of beholding the physical manifestation of your labor? As a software engineer, sometimes I feel like I'm missing out in my digital toil. Bakers can see (and taste) delicious breads and cakes at the end of their days. When carpenters are finished with a project, the result is a house they can physically walk through or live in. Even other varieties of engineering produce towers, bridges, rockets, and automobiles. 

<img src="https://upload.wikimedia.org/wikipedia/commons/e/e8/Bound_computer_printout.agr.jpg" width="30%" style="float: left; margin-right: 10px;">

This tangible, physical result of labor and the satisfaction that comes with it is something that I very much miss in software engineering. At the end of my workday I'm left with nothing more than some shifted bits on a magnetic disk. Sure, I can run my code and see the result on a screen. But for many projects, even months of work and thousands of lines of code might only result in the output of a single number. Contrast this with a project like a skyscraper, where the amount of effort invested is directly reflected by the enormity of the result [1].

Authors also work in digital-land, but they get to enjoy the physical heft of their labor in the form of a published novel. So why don't software engineers do the same? Actually, early software engineers did routinely print their code in a ["listing"](https://en.wikipedia.org/wiki/Listing_(computer)) for the purposes of debugging, sharing, and storing code. [[[Talk about moon code here?]]] Over time, improvements in computer monitors, storage space, and communication capabilities obviated the need to print out listings. This got me thinking: what if we still did this today? How many pages would a modern software project need? How tall would that stack of pages be? Are we talking a magazine, or "War And Peace"? Or something much larger...

### A Million Lines of Code

To answer these questions, first we need some data on the size of modern software projects. Luckily for me, an intrepid group has already gone to the trouble of scraping the web for this [data](http://www.informationisbeautiful.net/visualizations/million-lines-of-code/), cataloging projects ranging from a thousand to a billion lines of code [2]. Now let's try to figure out how tall some of these code bases would be. We'll work in orders of magnitude, starting at a single line of code.

### 1 LOC (.02 mm - 2 cm )

On the order of one line of code (LOC), we have a single sheet of paper. Let's got through the calculation we'll use to support the rest of this exercise. A ream of *500 sheets of paper* is *2 inches thick*. The number of lines per page is a free variable, depending on font size, line spacing, margins, etc. Let's just assume *55 LOC per sheet* is reasonable; that's about a full page of text with standard margins at 10 pt font. We'll also assume the page is one-sided. 

With these constants, we can calculate that you can fit 27500 LOC per ream of paper, or *1 LOC per 1/165000 feet*. Now we have a useful conversion between lines of code and feet [X]!

### 10,000 LOC (2 cm - 20 cm)

A small app or website might be around 10 KLOC, or just under an inch tall. You could print out your hobby iOS app using less than a ream of paper. That doesn't seem so bad.

### 100,000 LOC (20 cm - 2 m)

At 100 KLOC we get to our first major application: the first version of Photoshop, reeleased in 1990 was 120 KLOC, almost a foot tall (0.22m). Or take the space shuttle; with a codebase of 400 KLOC, its listing would be 2.42 feet tall (0.74 m).

<img src="https://upload.wikimedia.org/wikipedia/commons/2/2e/Margaret_Hamilton.gif" width="40%" style="float: right; margin-right: 10px;">

With a little more code, we've reached our first point of comparison. Consider this famous photograph of Apollo software engineer Margaret Hamilton standing next to a stack of simulation printouts. Assuming she's roughly 5 feet tall (estimating from the the height of the wall socket), Then this stack of papers represents 825,000 lines of code.

### 1 Million LOC (2 m - 20 m)

At 1 Million LOC (MLOC), the listing would be as tall as a 6 foot human. It used to be that software on the order of one million lines was impressive; in the original Jurassic Park, Dennis Nedry famously states that he debugged the 2 Million lines of code needed to run the entire park. It turns out that 2 MLOC also runs the Hubble Space Telescope. Its listing would reach to the top of a 12 foot (3.69 m) ceiling.

### 10 Million LOC (20 m - 200 m)

### 100 Million LOC (200 m - 2000 m)

### 1 Billion LOC (2000 m - ... )

Finally, we get to the mother of all codebases. Google is famous for storing all of its code in one, monolithic repository weighing in at over 2 Billion LOC. This is more code than every other project on the million lines of code list *combined*. Think about that for a second. Google's codebase is larger than all of Debian 5.0 + packages; OS X 10.4; Windows 3.1, XP, Vista, 7, and NT 3.5 though 4.0; Unix 1.0; Linux 2.6 and 3.1; Visual Studio 2012; Office 2001, 2006 (for Mac), 2013, and Open Office; the Boeing 787, the Mars curiosity rover, the space shuttle, the F-35, and the Large Hadron Collider; and all of Facebook. Combined! 

So how tall is the listing of the Google codebase? 2 Billion lines of code would reach over 12,000 feet. If you were to climb to the top of this listing and jump off, it would probably look a lot like my favorite scene from Godzilla (2014, set in San Francisco) [X].:

<iframe width="560" height="315" src="https://www.youtube.com/embed/lmZJiBZtahk?rel=0&amp;controls=1&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>



## What's this mean?

So what can we learn from this? I presented this article as a fun exercise, because that’s how I originally approached the idea. But by the end, I realized (and I hope you’ve had(and maybe you’ve just had this realization as well) that I have no ability to comprehend the enormity of modern software projects.  For a long time now, I've believed that the practice of software engineering as it exists today is becoming untennable. Even at a million lines of code, software seems manageable. I mean, I've at least read a million lines of text in my life; the entirety of anything I've read might reach 3 stories talk, if that. But could it reach to the height of a tower of a mountain? Never!  

By the end of my research, I became very concerned that it takes more code to run Google than it does to run every piece of software I've ever used in my entire life combined. And with all that code, what are we getting out of it. Do we need all of that code?   What kind of software could we expect at 3 billion lines of code? 
 
Our thesis in working on Eve is that fewer lines of code leads to lower overall complex it, all else being equal. We've implemented a basic spreadsheet application in only 50 lines of code. Whereas something comparable might be 1000 lines or more in another language.  
 
 
 While it pales in comparison to Excel, 




[1] Obviously its hard to *understand* all the effort put into building skyscraper just by looking at it. But its sheer physical enormity automatically gives a sense that a massive amount of work was needed to build such a thing. If a program just produces a single number as output, you really can't say anything about how much work was put into the it.

[2] Some of the data are dubious; 500 Million lines of code for Healthcare.gov has been debunked thoroughly: {{{CITE}}}. We'll try to only use the well documented examples.

[X] A more accurate conversion would be discontinuous, but we're dealing with numbers so large that neglecting the step error is dwarfed by the rounding errors.

[X] On the way down (1:45), you can even see Windows 7 (Godzilla) battling it out with Windows Vista (MUTO) At about 50 million LOC, the listing of either Windows versions would be about as high as these 300 foot tall monsters.