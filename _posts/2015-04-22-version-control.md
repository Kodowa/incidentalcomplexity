---
layout: post
title: "Version control, collaborative editing and undo"
author: "Jamie Brandon"
tags: []
---

Collaborative editing, undo/redo and version control are all instances of the same problem. They are also all legendarily hard to get right. With that in mind, I would like to have more eyes on the design I'm proposing for Eve.

### Past

The standard solution to any hard problem is to find someone who solved it already and steal their answer. Let's look at existing distributed version-control systems and collaborative editors.

The hardest problem that a DVCS has to solve is figuring out how to take changes made in one context and apply them in another context. This is a hard problem because most of the important semantic information is missing - while the user is thinking about changes like 'inline function foo into function bar' the DVCS can only see changes like 'add this text at line 71'.

This difficulty is compounded by recording changes after the fact (by diffing) rather than as they are made. Detecting even simple operations like moving a function into another file has to rely on heuristics about textual similarity instead of just observing the cut and paste.

With this kind of information the problem isn't even well-defined. There is no guarantee that merging two good branches will result in a project that even parses, let alone compiles and passes tests. All the widely used tools settle for heuristics that try to balance reducing surprise with reducing manual effort. In practice, predictable merges are valued more than clever merges.

In [Git](https://www.kernel.org/pub/software/scm/git/docs/git-merge.html) and [Mercurial](http://hgbook.red-bean.com/read/a-tour-of-mercurial-merging-work.html) the input to these heuristics is the two versions to be merged and their most recent common ancestor (if there are multiple candidates they can be recursively merged).

[Darcs](http://en.wikibooks.org/wiki/Understanding_Darcs/Patch_theory) additionally considers the chain of patches that led to each version. It transforms individual patches one by one in order to be able to apply them in the context of the other branch. The upside of this bag of patches model is that it [makes cherry-picking and rebasing easy](http://darcs.net/manual/Introduction.html). The downside is that it occasionally hits [dramatically slow cases](http://darcs.net/FAQ/ConflictsDarcs1#darcs-is-really-slow-is-this-the-big-conflicts-bug-in-action) and [doesn't track the project history](http://markmail.org/message/vk3gf7ap5auxcxnb).

 [Operational Transform](http://en.wikipedia.org/wiki/Operational_transformation) algorithms solve collaborative editing with a similar approach. Each editor broadcasts changes to the text as they happen and transforms incoming changes to apply to the current text. The editors do not track the merge history and it is [very hard to prove](http://en.wikipedia.org/wiki/Operational_transformation#Critique_of_OT) that every editor will eventually reach the same state. In fact, [Randolph et al](http://arxiv.org/pdf/1302.3292.pdf) found that every proposed algorithm in the academic literature can fail to converge and that guaranteeing convergence is not possible without additional information.

 [Treedoc](https://hal.inria.fr/inria-00445975/document) and various other [CRDTs](http://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) provide this additional information in the form of hidden identity tokens for each chunk of text. The change events refer to these tokens rather than to line and column numbers, which makes it easy to apply changes in any order. The proof of convergence is simple and, more importantly, the algorithm is obvious once you have seen the identity tokens. Practical implementations are tricky though since the tokens can massively inflate the memory required to store the document.

 So, things to think about when designing our solution:

 * Recording changes as they happen is easier than inferring them after the fact

 * Preserving history - the context in which a change was made - is necessary for merging correctly

 * Having stable identities reduces the amount of context needed to understand a change

 * Being predictable is more important than being smart

### Present

Eve is a [functional-relational](http://shaffner.us/cs/papers/tarpit.pdf) language. Every input to an Eve program is stored in one of a few insert-only tables. The program itself consists of a series of views written in a relational query language. Some of these views represent internal state. Others represent IO that needs to be performed. Either way there is no hidden or forgotten state - the contents of these views can always be calculated from the input tables.

The code for these views is stored similarly. Every change made in the editor is inserted into an input table. The compiler reads these inputs and emits query plans which will calculate the views.

Eve is designed for live programming. As the user makes changes, the compiler is constantly re-compiling code and incrementally updating the views. The compiler is designed to be resilient and will compile and run as much of the code as possible in the face of errors. The structural editor restricts partially edited code to small sections, rather than rendering entire files unparseable. The pointer-free relational data model and the timeless views make it feasible to incrementally compute the state of the program, rather than starting from scratch on each edit.

We arrived at this design to support live programming but these properties also help with collaborative editing. In particular:

* Tables are unordered, so inputs can be inserted in any order without affecting the results

* The editor assigns unique ids to every editable object on creation, so changes require very little context

* Partial edits and merge conflicts only prevent the edited view from running, not the whole program

* Runtime errors only prevent data flow through that view, rather than exiting the program

* If all users are editing on the same server they can share runtime state

### Future

As a thought experiment, let's suppose we connect two editors by just unioning their input tables. What would go wrong?

Firstly, the compiler crashes. While each independent editor respects the invariants that the compiler relies on, the union of their actions might not. For example, each editor might set a different human-readable name for some view, breaking the unique-key invariant on the human-readable-name table.

As a first pass, we can resolve this with last-write-wins. Both users share a server and the server is authoritative. For online collaborative editing this is actually good enough - we are able to write programs with multiple editors without problems.

To support offline editing, version control and undo/redo we need to be more careful about what we mean by 'last write'. We can track the [causal history](http://scattered-thoughts.net/blog/2012/08/16/causal-ordering/) of each change. When two conflicting changes are causally ordered we pick the later change. If the changes are concurrent we tag it as a merge conflict and disable that view until the conflict is resolved.

The difficult question is when do two changes conflict? There is no 'correct' answer to this question - merging is an under-defined problem. In any VCS you can make code changes that are individually valid, will be merged automatically and will break in the final program (a simple example in dynamic languages is defining two functions with the same name). Existing tools simply try to guarantee that the user is not surprised by the result and that no information is lost. We will add an extra constraint - the result will always compile (even if we have to emit warnings and disable some views).

This last constraint gives us our answer - two changes conflict when applying them both would break an invariant and crash the compiler (this is remarkably similar to [I-confluence](http://www.vldb.org/pvldb/vol8/p185-bailis.pdf)). The invariants fall into a few categories:

* __Types__: Reject any ill-typed errors and warn the user.

* __Unique keys__: If one change is causally later than all other changes then it wins. If there multiple changes which are not ancestors of any other change, disable the view and warn the user.

* __Foreign keys__: Replace the missing value with a default value and warn the user.

* __Ordering__: Instead of using consecutive integers use an ordered CRDT. Resolve ties arbitrarily but consistently (eg by comparing the items uuid).

To summarise: most changes don't conflict, some can be resolved by recording additional information, the remainder are flagged as conflicts and are not compiled until they are fixed.

I glossed over the recording of causal history. This is a core difference in philosophy between darcs and git. In darcs, the history is a mutable bag of patches and is easily rearranged. This makes features like cherry-picking easy to use and implement. In git, the history is an immutable DAG of commits and is hard to change. This is useful for auditing and bisecting changes. The choice of how we record causal history has important ramifications for how we implement undo/redo.

I really wanted undo to behave like [undo-tree](http://www.emacswiki.org/emacs/UndoTree), where undo moves up the tree of changes, editing creates a branch and redo moves down the current branch. Unfortunately, I can't reconcile this with collaborative editing, where undo should only undo operations made by the same user (imagine Alice is off working in a different section of the program and hits undo, undoing Bob's last change in mid edit). To undo something that is not on the tip, git offers [revert](http://git-scm.com/docs/git-revert) and [rebase](http://git-scm.com/book/en/v2/Git-Branching-Rebasing). Rebase creates an entirely new chain of commits which causes confusing merges if someone else is concurrently working on the same branch. Revert doesn't allow for undo-tree -like behaviour.

I'm considering an approach that gives us both the easy cherry-picking of darcs' bag of patches model and the auditable history of git. It's able to be simpler than both because we are solving an easier problem - the changes we are merging contain much more information about intent than raw text diffs.

There are three kinds of data tracked by the system:

* a __patch__ has a unique id and a set of input tuples to be added
* an __insert__ has a unique id, a set of parent commits and names a patch to be applied
* a __delete__ has a unique id, a set of parent commits and names an *insert* to be removed

The inserts and deletes form a causal history DAG. Collecting all the inserts and removes between the current tip and the root gives you an [observed-removed set](https://github.com/aphyr/meangirls#or-set) of patches. The union of these patches is passed to the compiler, which flags any merge conflicts.

### Example

Suppose Alice and Bob are collaborating together on a web page. Alice sets the title of the page to 'Bobs Sandwiches'. Bob sees this, sets the background to 'burger.jpg' and edits the title to read 'Bobs Burgres'.

```
Alice: Patch{id=P0, inputs=[Title{text="Bobs Sandwiches"}]}
Alice: Insert{id=I0, parents=[], patch=P0}
Bob: Patch{id=P1, inputs=[Background{src="burger.jpg"}}
Bob: Insert{id=I1, parents=[I0], patch=P1}
Bob: Patch{id=P2, inputs=[Title{text="Bobs Burgres"}]]}
Bob: Insert{id=I2, parents=[I1], patch=P2}
```

There can only be one title. I0 is an ancestor of I2 so 'Bobs Burgres' wins the conflict.

Now Alice and Bob both notice the typo in the title and try to fix it at the same time.

```
Alice: Patch{id=P3, inputs=[Title{text="Bobs Sandwiches and Burgers"}]}
Alice: Insert{id=I3, parents=[I2], patch=P3}
Bob: Patch{id=P4, inputs=[Title{text="Bobs Burgers"}]}
Bob: Insert{id=I4, parents=[I2], patch=P4}
```

These inserts are concurrent - neither is an ancestor of the other - so the editor displays a merge conflict warning. Alice resolves the conflict with a compromise:

```
Alice: Patch{id=P5, inputs=[Title{text="Bobs Burgers (and Sandwiches)"}]}
Alice: Insert{id=I5, parents=[I3, I4], patch=P5}
```

Every other insert is an ancestor of I5 so the new title wins all conflicts.

Meanwhile, Bob is now tired of the whole thing and hits undo twice:

```
Bob: Remove{id=R0, parents=[I5], insert=I4}
Bob: Remove{id=R1, parents=[R0], insert=I2}
```

This removes P2 and P4 from the set of patches that the compiler sees. Removing P4 has no effect, because it has already been superseded by P5. Removing P2 removes the burger background.

### Problems?

Redo is still tricky. Suppose Bob liked the background so he panics and mashes redo.

```
Bob: Insert{id=I6, parents=[R1], patch=P2}
Bob: Redo{id=I7, parents=[I6], patch=P4}
```

This brings back the background, but also makes P4 the new winner of the title conflict. This is probably pretty surprising to Bob. One possible solution to this is when Bob hits undo it should skip P4, because P4 has lost a conflict and has no effect on the current state.

Another potential problem is that we might automatically merge changes which are individually innocent but together cause some unexpected effect. This is possible in any VCS but we automatically merge a much higher percentage of changes. We may want to add a merge review which highlights sections that may need more attention.

I haven't yet figured out how to handle metadata such as branch pointers. If I make edits offline and then reconnect, should other users in the same session receive my changes or should I be booted into a new session? How do we undo changes to metadata (eg accidentally overwriting a branch)? I'm considering storing metadata in branch of its own so that changes are recorded and can be undone.
