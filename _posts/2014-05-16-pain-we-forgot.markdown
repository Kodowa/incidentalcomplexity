---
title: Pain we forgot
author: Jamie Brandon
layout: post
---

Much of the pain in programming is taken for granted. After years of repetition it fades into the background and is forgotten. The first step in making programming easier is to be concious of what makes it hard. So let's put ourselves in the shoes of a smart but inexperienced end user trying to build, test and maintain a simple application.

Anon the intern is charged with managing lunch orders and quickly realises that their job could be done by a computer: Every day at 1000, send each employee an email with a link to a form where they can choose what they want for lunch. At 1200, gather up the replies and email the list to the catering company. At the end of every month, tally up what each person owes and send the list to accounting. While this a simple program, it covers all the basics: data input, validation, error handling, calculation, presentation, communication, reactivity, scheduling, deployment etc. There are probably dedicated apps that cover this particular example but we are more concerned with how an end-user would solve this kind of problem in general and the difficulties they will encounter along the way.

A lot of the problems we will encounter seem unavoidable - they are forced on us by outside constraints. Most of these constraints though are the product not of deliberate choices but of historical accident. We still program like it's 1960 because there are powerful path dependencies that incentivise pretending your space age computing machine is actually an 80 character tty. We are trapped in a local maximum.

One might also argue that these tools are simple enough once you learn to use them. I would only point out that, emperically, that bar is too high. Despite the clear benefits, the vast majority of the world has chosen to remain illiterate. Even tools for which there is a clear need (eg version control) have largely failed to make a dent. Clearly there is a need for a less hostile programming environment.

It is tempting to believe that this is the best we can do and that programming is naturally this complex. But as we work through our lunch app, consider how little of the work we have to do actually relates to the problem of specifying the application.

---

### Getting started

First we need to get a development environment running. Let's try clojure:

``` bash
lein new lunch_app
cd lunch_app
mkdir resources
touch resources/index.html
LightTable resources/index.html # insert script tag for web repl
firefox resources/index.html
LightTable clj/lunch_app/core.clj # server side, fire up compiler, connect to repl
LightTable cljs/lunch_app/core.cljs # client side, connect to web repl
```

At this point you have already lost 99% of the population and we haven't even touched on css or templates yet. Worse, none of this was discoverable. I happen to already know how to setup a simple client-server web app so all these steps seem obvious to me. But Anon the intern needs to be able to open up Programming&trade; and click 'New Web Form'. Intellij does a reasonable job on this front - you can start a new web project, compile and open the result in a browser in a few button clicks. But in most environments you need a tutorial just to start a new project.

### Finding help

So Anon is now staring at a blinking cursor on a blank editor page. What next? How does one go about making a web form, or send a email? For common tasks google will probably find you entire code samples or at the very least some javadocs. The samples will be missing lots of implicit information such as how to install the necessary libraries and how to deal with missing dependencies and version conflicts. Transcribing and modifying the examples may lead to bugs that suck up time. It's not terrible, mostly thanks to sites like stackoverflow, but it's still a lot of unnecessary distractions from the task at hand.

I want to just type 'email' and see a list of functions and libraries relating to email. If I select a function from autocomplete, its dependencies should be automatically added to the project without any fuss. Missing dependencies or version conflicts should be presented alongside suggestions for resolution (click here to choose version A). [Bing Code Search](http://blogs.msdn.com/b/visualstudio/archive/2014/02/17/introducing-bing-code-search-for-c.aspx) takes this idea even further and autocompletes code for common tasks.

### Writing code

Even for experts, programming is an exploratory process. We experiment with libraries, run through examples and iteratively build up features. One of the most painful lessons beginners have to learn is just how often everyone is wrong about everything. Tightening the feedback loop between writing code and seeing the results reduces the damage caused by wrong assumptions, lightens the cognitive load of tracking what should be happening and helps build accurate mental models of the system. The latter is especially important for beginners who often suffer from miscomprehensions about even the basic semantics of the language. Unfortunately, the most you are likely get is automatically refreshing your browser. Maybe a REPL if you are lucky.

Imagine a spreadsheet where every time you change something you must open a terminal, run the compiler and scan through the cell / value pairs in the printout to see the effects of your change. We wouldn't put up with UX that appalling in any other tool but somehow that is still the state of the art for programming tools. I suspect a lot of the blame lies in our failure to find a model for GUI tools that is as flexible and composable as plain text. I see a lot of potential in Paul Chiusano's ideas for [killing the application](http://pchiusano.blogspot.com/2013/05/the-future-of-software-end-of-apps-and.html) and in Eskil Steenberg's [Verse](https://www.youtube.com/watch?feature=player_detailpage&v=f90R2taD1WQ#t=1837).

Light Table at least gives you [inline eval](https://www.youtube.com/watch?v=gtXpOD6jFls), [watches](https://www.youtube.com/watch?v=d8-b6QEN-rk) and the [instarepl](https://www.youtube.com/watch?v=YY6B9EHbH24). This type of interaction is taken further by ideas like [Debug Mode is the Only Mode](http://gbracha.blogspot.com/2012/11/debug-mode-is-only-mode.html) and [Example Centric Programming](http://www.subtext-lang.org/OOPSLA04.pdf)). Instead of having a separate editor, compiler, repl, debugger etc you develop everything by editing code in the debugger. It is a similar idea to JIT compilers - the IDE has more information available at runtime then at compile time so it can make better decisions and provide better feedback (eg by generating example inputs and outputs as you write a function).

Plain text is also very limiting. Language is very good for conveying meaning but not so great for displaying data. Being able to quickly throw up graphs and diagrams (like in [rhizome](https://github.com/ztellman/rhizome), [automat](https://github.com/ztellman/automat) or [lamina](https://github.com/ztellman/lamina/wiki/Channels)) is incredibly useful. Light Table's [inline graphs](http://www.chris-granger.com/images/040/ipygraphs.png) are a start but we haven't otherwise made much use of visualisation. First person to implement inline graphviz gets a cookie.

### Running code

Surprisingly, one of the most common difficulties we have heard from beginners is just running code. Even if we were to hand Anon the entire lunch\_app source code they would likely still struggle to actually use it. They have to install dependencies, compile code, start servers and open ports. At each step the errors are difficult to diagnose and time-consuming to fix. The tools that are intended to fix this are often even worse themselves (every time I write a blog post in octopress I find rvm has somehow broken again). IDEs like Intellij and Visual Studio do a reasonable job of standardising the build process and capturing dependencies so that it is usually possible to import a project and just hit run, but that only goes as far as development. For deployment we have tools like Docker which make deployment highly repeatable but don't help much with capturing the process in the first place. None of these really help Anon the intern deploy lunch\_app.

The lunch app is going to need scheduling too, and error logging and monitoring. Anon needs to be alerted if the emails don't go out or if there are no reponses. Setting up even the simplest logging, monitoring and restarting is a hassle even for professional programmers.

Wolfram's Language [workflow](https://www.wolfram.com/universal-deployment-system/) is pretty close to ideal. You work in a notebook where code runs and updates instantly with no manual compile step. Deploying to a cloud server is just a single function call which automatically collects code and dependencies and returns a url where your program is now running. No need to think about files or libraries, no project files, no build artefacts, no messing about setting up servers and opening ports.

From there it doesn't take much imagination to add easy task scheduling and an error logger that emails Anon when something goes wrong. None of this requires giving up control either. You could just as easily replace 'cloud server' with 'departmental server' or 'little black box that came with our internet'. The important point is that there are sensible defaults for deployment and that it is 'batteries included' in the language or IDE.

### What?

The simplest question we could ask about our application is 'what is the current state'. Bizarrely, very few programming environments give you any help on this front. Many programmers get by with nothing but print statements. If you are lucky you may have a debugger or watches, but you still end up looking at your application through a keyhole. You have to actively insert instrumentation by hand to view the state of each tiny part of the application. If you want to modify that state you have to mentally work backwards and construct the correct piece of code to find and change the variable that you are looking at. It may not even have a name that is accessible from the repl (eg a variable in an anonymous closure). Viewing and modifying the state of the application should be a fundamental interaction and yet it's made unreasonably difficult by our languages and tools.

Compare this to a tool like Excel or [Django Admin](http://www.youtube.com/watch?v=kvFDV1oM-ZA) where *all* the data is laid out for easy browsing without any active effort from the user and can be *directly* modified just by clicking and typing. The tooling itself isn't difficult but it requires rethinking the way we manage state in programming languages. All mainstream languages, regardless of paradigm, encourage [anonymous local state](http://scattered-thoughts.net/blog/2014/02/17/local-state-is-harmful/) which can't be easily observed and modified.

Once we have managed state, whether using a relational model like [Bloom](http://bloom-lang.net/) or more traditional functional data-structures like [Opis](http://infoscience.epfl.ch/record/136776), we can easily record history too. Tools like [time travelling debuggers](http://chrononsystems.com/products/chronon-time-travelling-debugger) that require huge engineering effort in traditional languages become trivial when you can cheaply record or reconstruct the past. Reproducing bugs is easier when you can just snapshot your history and mail it to the developer. Bloom and Opis are also both able to determine dataflow topologies from source code so when stepping into an unfamiliar project you can quickly get a visual overview of how the various components communicate (examples are buried [here](http://db.cs.berkeley.edu/papers/cidr11-bloom.pdf) and [here](http://infoscience.epfl.ch/record/136776/files/DagandETAL09Opis.pdf)).

### Why?

Traditional debuggers focus entirely on the *what* - walking through a narrow slice of state on step at a time. But when debugging the question one usually starts with is *why*? Why are the lunch options in the wrong order? Why didn't the lunch email go out? Why is everyones bill for the month zero? These questions typically involve reasoning backwards from effect to cause whereas debuggers walk you forward from cause to effect. The result is that debugging consists mostly not of finding the problem but manually walking backwards along the chain of causes by setting up isolated test cases and repeatedly rerunning them under the debugger.

[Theseus](http://blog.brackets.io/2013/08/28/theseus-javascript-debugger-for-chrome-and-nodejs/) improves on this slightly by capturing arguments at the entrace to each event callback, so that you don't have to repeatedly rerun. Ko and Myer's [causal debugger](http://repository.cmu.edu/cgi/viewcontent.cgi?article=1165&context=hcii) explicitly answers the questions *why* and *why not* by tracing the tree of causes for each state change, so that the process of walking backwards from effect to cause is entirely automated and you can just focus on figuring out where things went wrong.

The problem gets even worse with scale. Debugging by following control flow works poorly in large systems where what really matters is *data flow*. Answering questions like 'why do orders sometimes get lost' requires tracing through an enourmous graph, one which is not even recorded in most systems and instead has to be inferred from logs, like piecing together ancient civilisations from broken pottery. [BOOM analytics](http://db.cs.berkeley.edu/papers/eurosys10-boom.pdf) deals with this by reflecting all data, from error logs and persistent data to message queues and profiling data, into relational tables that are made available to overlog - the same distributed query language that runs the rest of the system. This means you can directly run queries over the causality graph, such as 'for each order that was entered into the system but not completed, give me a list of every message was linked to that order by some chain of rules'. Since the recording of this data was itself governed by overlog rules you can switch on detailed logging at runtime for specific kinds of data eg 'record all messages concerning order 197 originating from cluster C and forward them to me'.

### Change

Outside of the software world, version control and collaboration software is limited to saving lunch\_app.v07 and attaching it to an email. Collaborating on a single project is difficult and slow. The standard tools of the trade for programmers (git, mercurial etc) are vastly more powerful and solve a pressing problem but present an interface that [baffles and frustrates many users](http://steveko.wordpress.com/2012/02/24/10-things-i-hate-about-git/). The underlying model is elegant and powerful but even the graphical interfaces require a significant investment of time and effort to understand.

What Anon needs is somewhere between [undo-tree](http://www.emacswiki.org/emacs/UndoTree) (without the ascii art) and [etherpad](http://etherpad.org/). Changes should be automatically recorded and (optionally) retroactively tagged with commit messages. Real-time collaboration should be as simple as clicking on a coworkers face. Undoing changes and checking out different versions should just be a matter of moving around on the [commit graph](https://github.com/blog/39-say-hello-to-the-network-graph-visualizer). Dragging a piece of code out of the editor should produce a link to the VCS page for that code. If the editor understands the structure of the code we even can track semantic changes to individual units of code (eg rename function, reorder expressions) rather than diffing text in a file, making both automatic and manual merges easier since we have a better record of the intent of the change.

Similary, when Anon 2 the accountant wants to modify their client-side copy of the lunch form to remember their favourite lunches it should be a simple process. No hunting down and recompiling system binaries, no installing greasemonkey scripts from the filesystem. Just click to open the source, modify to your satisfaction, click to save. I've never seen anything come close to this basic interaction. The [OLPC view source button](http://blog.printf.net/articles/2006/10/29/the-view-source-key/) promised exactly this experience but as far as I know it never materialised (it certainly didn't work on mine).

### Learning

Programming tools generally pay very little attention to producing helpful error messages (with [one](http://cs.brown.edu/~sk/Publications/Papers/Published/mfk-measur-effect-error-msg-novice-sigcse/paper.pdf) or [two](http://research.microsoft.com/pubs/64590/helium.pdf) exceptions). There is a modest amount of [evidence](http://www.amazon.com/Man-Who-Lied-His-Laptop-ebook/dp/B003YUC7BI/ref=sr_1_1?ie=UTF8&qid=1400099030&sr=8-1&keywords=man+who+lied+to+his+laptop) that people interact with computers as if they were people. Many of the results of this research are suprising and counter-intuitive eg [personifying the compiler](http://faculty.washington.edu/ajko/papers/Lee2011Gidget.pdf) can improve learning rates in students. Given that, do you really want to spend lots of time with the kind of person who just repeatedly shouts 'cannot call method undefined of undefined!' in your face without so much as hinting how you might fix the problem or where you might start looking?

Our programming environments are absurdly hostile. Interfaces either [overwhelm with detail](http://www.suggestsoft.com/images/quest-software/comparerocket-for-visual-studio.gif) or [hide everything](http://i.stack.imgur.com/VqPMv.png). Most actions cannot be undone (eg changing a variable, defining a function, installing a library). Runtimes default to exiting on uncaught exceptions, throwing away all the context that would be useful for solving the problem and forcing the user to try to recreate the crashing state. When any action can lead to confusing breakage and ruined work, inexperienced users suffer from fear and paralysis and an unwillingness to experiment. This cripples their ability to learn.

Error messages should at the very least identify what might have caused the error and preferably offer options for fixing it. Intellij, for example, will highlight spelling mistakes and offer to correct them ("did you mean..."). Good end user applications will link common errors to FAQs. Suggestions don't need to be perfect, just accurate. Everyone hated Clippy because his advice was useless and repetitive and lacked context. The golden rule is if you don't have something useful to say, don't say nothing at all. One ambitious project (ref?) crowd-sourced examples of causes of and solutions to type-checking errors. Large-scale data collection and testing may end up being the best path to providing helpful feedback.

Environments also need to be more proactive. Uncaught errors should drop the user into the debugger where they can [edit and continue](http://www.gigamonkeys.com/book/beyond-exception-handling-conditions-and-restarts.html). Editors can spot common mistakes and suggest corrections (Intellij is pretty good at this, as is [kibit](https://github.com/jonase/kibit), but many people are still working with editors that don't even warn them of typos or shadowed variables). Profilers can heuristically explore bottlenecks and suggest solutions (eg if foo was indexed this query would run 10x faster). Rather than rely on users to create their own tests we can prompt them for examples and invariant properties and [search for counter-examples](www.scs.stanford.edu/11au-cs240h/notes/testing-slides.html). [Opis](http://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&cad=rja&uact=8&ved=0CD0QFjAC&url=http%3A%2F%2Finfoscience.epfl.ch%2Frecord%2F136776&ei=gNpzU7yeKMTesASLr4KoCA&usg=AFQjCNGyqXOAavdVfGxGuBFZbTzobRmXtQ&sig2=jvF_qeyNTyCOffdJkC-twA&bvm=bv.66917471,d.cWc) comes equipped with a profiler that automatically estimates the asymptotic complexity of each function in the model and a finite-state model checker that can prove invariants always hold by efficiently and exhaustively checking every possible state. Bloom features a [generative testing framework](http://db.cs.berkeley.edu/papers/dbtest12-bloom.pdf) that uses an SMT solver to efficiently explore the space of possible and a [static analysis pass](http://www.bloom-lang.net/calm/) that warns of missed coordination points in distributed programs. Does your IDE even run your unit tests for you?

Finally, environments can't be black boxes. Beginners need a simple experience but if they are to become experts they need to be able to shed the training wheels and open the hood. Many attempts at end-user programming failed because they assumed the user was stupid and so wrapped everything in cotton wool. Whenever we provide a simplified experience, there should be an easy way to crack it open and see how it works. Nothing should be magic forever. Ensure that the users curiousity is never frustrated and they won't need teaching for long.

---

Some of the things I have described are just a matter of paying attention to the details. Others require doing things very differently. The key parts of our plan for Aurora are:

* storing code in a networked database with version control and realtime sync
* a [structured editor](http://en.wikipedia.org/wiki/Structure_editor) to enable rich ASTs with unique UUIDs
* managing environments declaratively so that evaluating code is always safe
* a uniform (logical) data model where every piece of state is globally addressable
* a model for change that tracks history and causality
* a powerful query language that can be used for querying code, runtime state, causal graphs, profiling data etc
* composable gui tools with transparent guts
* a smooth interface to the old world so we don't end up sharing a grave with smalltalk

We will dive into these in more detail in the coming months.

None of this will be at all easy, but it's no harder than continuing what we are doing now and much of the groundwork has already been laid if you know where to look. If one thing is certain, it is that the future is not 80 characters wide.
