---
title:  "Vim Shortcuts: Automatically Wrap Long Lines"
date:   2018-11-19
categories: vim
---

Shortcuts make the world go round.

Well, not really. But they sure help get things done. They _help_, to a certain
extent, get easy and mundane things out of the way so that you focus on more
interesting problems.

Writing this blog, one shortcut that I reach for all the time is `gq`. I'll also
reach for this shortcut when I'm writing a helpful commit message.

### What does it do?

A few ways that I can think of, to describe what the shortcut does, are "insert
hard wraps", "auto line wrap", "format paragraph", or something along those
lines–_yes, pun intended, big time_. It essentially takes a paragraph and makes
sure that no line is longer than 80 characters. For lines that are less than 80
characters, vim will steal a word or two (or as many as necessary) from the
lines _below_ it, and tack them onto the end of the current line in order to get
that line as close as possible to 80 characters.

If you have `textwidth` set to some number, then it'll use that value
instead of 80. You can `set textwidth=40` (or whatever value you want) in your
current session, or in your `~/.vimrc` to get that set each time, and always
force wrapping to that length.

### See it in action

Go ahead and type out a super-long line. Highlight the line (using `shift+v`)
and then type `gq` to se the wrap happen. Works like a charm.

A variation of this, if you want to avoid highlighting, is `gq}`. Since `}` is
just a motion that means "to-end-of-paragraph", vim will perform wrapping for
you your current position all the way to the end of the paragraph. No
pre-highlighting required.

One caveat with this is that if you're in the middle of a paragraph, it won't do
everything above your current position (but that doesn't mean you can't `{`
right before to get yourself to the beginning of the paragraph).

Either way, whatever floats your boat, just know that `gq` is a thing and works
quite well for those cases when you need those hard line breaks, but just don't
want to do them yourself. Enjoy!
