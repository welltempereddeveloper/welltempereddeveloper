---
title:  "Programming 101: Keep Your Functions Short"
date:   2018-12-03
categories:  quality
---

It's becoming blatently obvious that this isn't being stressed enough, in
schools nor in the workplace. A long function is very detrimental to progress in
the long run. If there is one thing that all devs could do today that would
drastically increase productivity in the long run, it would be to simply stop
writing long functions.

I'm not even talking about going back and refactoring existing functions. I'm
just talking about new functions that get introduced today and going forward.
Every day, as new features are being added to software throughout the world,
there are new piles and piles of functions that are tens, hundreds, and even
thousands of lines long. These functions do many, many things.

### A function should do one thing, and one thing only.

Set a very low upper-limit on the maximum number of lines that a function can
span.  Let's say, even as low as ten lines and target that. If the function of a
function can't be implemented in ten lines or less, it's probably doing several
things. We don't want that.

### Why?

Modularity, modularity, modularity. Modularity has been at the core of
[Unix][unix-philosophy-1] [philosophy][unix-philosophy-2] for decades. It is
largely responsible for helping Unix get to where it is today.

Functions (and programs) that do one thing can progressively be chained together
to do something slightly more interesting. A chain of well-named functions,
performing some task, is much easier to comprehend than a single function
performing the same task.

If a function has one, well, function, then that function can be reused easily
(without copy+paste) in other places. When a function does two things, reuse
gets harder.

### And also ...

Functions that do one thing can easily be removed when that one thing is no
longer needed. This helps keep code bases cleaner and free of dead code. That,
in turn, keeps them simpler and easier to maintain.

Debugging is easier. Functions that are modular are easier to debug. It does
one thing, it should be fairly easy to verify that it actually does do that one
thing.

### It's harder to get sloppy

When you limit yourself to ten lines to do something interesting, you're going
to try much harder to make those lines worth-while. There won't be dead
branches, unused variables, "just-in-case" blocks. There will be no dependencies
between two pieces of code that span 100 lines that are impossible to spot from
a quick glance.


### So give it a shot

To close things off, I'm going to leave you with one of my favorite quotes from
[Martin Fowler][martin-fowler].

> "Any fool can write code that a computer can understand. Good programmers write
> code that humans can understand."

[unix-philosophy-1]: http://www.linfo.org/unix_philosophy.html
[unix-philosophy-2]: https://en.wikipedia.org/wiki/Unix_philosophy
