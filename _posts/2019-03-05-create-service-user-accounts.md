---
title:  "Create Service User Accounts"
date:   2019-03-05
categories: automation
---

Automation implies "no human involved", at least to some extent, and depending
on who we ask. It almost always involves communicating with some system, which
in turn involves authentication/authorization.

If a user is performing an action, then things are pretty simple. The person
logs in, does whatever needs to be done, logs out, end of story.

Things get a little slippery whe we bring automation into the mix. Now, instead
of user A changing things, we have system A changing things. As whom does the
system authenticate in order to perform those actions?

Things can get _very_ slipper if strict security requirements must be met. But
no matter what the scenario may be, there is one thing that we can do to help
keep things simple.

## Create Service User Accounts

Whenever we have a system performing an action, we make sure that the system is
performing an action _as itself_. In other words, we don't generate keys
specific to our personal user account and provide them for the system to use.

Why? There are several reasons.

Five months down the road, when tracing activity trying to determine what
happened to this and that, we don't want somebody's name showing up in the
audit trail when it was actually a system performing those actions. If System B
is responsible for making a change to the load balancers, then we want the
audit trail to show System B, not John Doe.

Another reason may be a little more obvoius. When John Doe finds a new job and
all of his accounts are disabled, then we don't want system B (which may be
using John's credentials) to also stop working. Take this a bit further and
imagine if John Doe was responsible for setting up 15 of these systems, all
configured with credentials specific to his account. When John leaves the
company, we will either be left with 15 broken systems, _or_ we will be
unable to disable John's account until we identify, locate, and rotate keys
for all of those 15 systems. On top of that, we should do our due diligence
and verify that the new credentials have the permissions necessary for those
systems to keep working as they used to work.

The system is not any particular individual in the organization. So we create a
service user for the system and configure the system with credentials specific
to that service user.

## Create _More_ Service User Accounts

This may or may not be obvious, but life is good when different systems have
their own user accounts and/or (at minimum) their own tokens/keys which can
uniquely identify them.

There are a few reasons for this as well. One of them, again, being auditing.
It's good to be able to see that System A (and not System B or C) performed
some action. It's _not_ good to find that key ABC performed some action, and
then find out that this key is actually used by 15 different systems.

That's reason number one.

Reason number two? Permissions (security).

Different systems are different and likely do different things. Go figure.

Each system requires different permissions. There is no reason to union all
permissions necessary by all our systems, create one service user with all
those permissions, and then allow all systems to use that one single user.

It _may_ be convenient, but certainly not the way to go.

The third reason is cleanliness. Yes, cleanliness.

Systems come and go. In a growing and ever-evolving organization, we will build
things, use them, grow out of them, and then not need them. That's when we
bust out the recycling bin and toss the things that are no longer need. Along
with those things, should go all of their service user accounts and credentials.
There is absolutely no need for a power-user from 5 years ago with access to
the primary AWS account to be hanging out when the system which used that user is no
longer in service. If the user used by this system is not shared by any other
system, then it's easy and risk-free to simply remove the user entirely.

If an unused system doesn't exist, then there is no chance for someone to
stumble accross it and message all of engineering asking "Hey what's this used
for?" Unused things are guaranteed to burn _at least_ 30 minutes, if not more,
for each distinct incident of someone accidentally stumbling upon it and
questioning the essence of its existence. If it's not used, just get rid of it.

## Are There Any Downsides?

I heard someone say "srsly? setting up all these system accounts is crazy!"

There is, without a doubt, more overhead to getting things setup this way.
Storing, securing, revoking, and rotating these credentials is a whole other
story. So yes, there are tradeoffs to take into account when considering this
approach.

For a five-person organization, it's probably not worth the hassle. For a
50-person organization? Well, now it's time to stop and think.

Things get further complex when we start considering security. For example, if
a person leaves the company, how quickly, easily, and safely are we able to go
through all keys and replace them with new keys? Can we automate this so that
an individual doesn't have to spend hours (or days) doing this? Once we've
automated it, how do we secure this system so that someone with access to the
system can't get their hands the newly generated keys? Who controls this system?
What if that person leaves?

Security is always a whole different conversation. The answers to these things
questions are about as clear as mud and a typical product company likely has
minimal experience doing this or resources which it can invest into getting
them done right.

Even with that being the case, we can still reap other benefits of service user
accounts, such as isolated permissions, safe removal of unused accounts, safe
addition of new accounts, and auditing.
