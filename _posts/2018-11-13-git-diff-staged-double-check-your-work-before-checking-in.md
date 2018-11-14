---
layout: post
title:  "Double Check Your Work: git diff --staged"
date:   2018-11-13
categories: scm quality
---

Double-checking your work is a good habit to get into. Whether you're
developing software or measuring wood for milling, validating your
intents will certainly save you time, frustration, and money in the long run.

Being the 10x developer that you are, I'm sure that you're using [git][git] for
source control.

An often overlooked and under-utilized feature of git is `git diff --staged`.
That is the "double-check your work" of source control. It essentially provides
a diff of, well, that which is staged and about to be committed. It's a "let
me make sure I'm committing what I _think_ I'm committing".

After running `git diff --staged`, you should carefully review that diff and
make sure that nothing is missing and, more importantly, contains only that
which _needs_ to be there.  If something is off, then fix it before you commit.
Simple.

The primary use case for `git diff --staged` is preventing problems that fall
under the category of "Oh, how did that get in here?" It makes us a little more
diligent with the things we commit.

Often times, I'll see devs run an innocuous `git commit -a` and immediately
follow that up with a `git push`. That's fine for weekend throw-away projects
or in scrap repositories for testing some things out.  But when developing
professional software, `git commit -a` should be seldomly used, if ever.

`git diff --staged`. Try it out, start using it, make it a habit.

Make it an alias, like `gds`. That's almost as easy to type as `fds`. In fact,
go ahead and alias it to `fds` if that'll help you use it more.

For best results, `git diff --staged` should be used immediately _after_ a `git
add` and just _before_ a `git commit`.


Pro tip: don't want to stage everything that's currently modified? Try a `git add
-p`. Likewise, want to remove a small thing that has already been staged? Try a
`git checkout . -p`.

[git]: https://git-scm.com/
