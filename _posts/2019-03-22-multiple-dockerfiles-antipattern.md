---
title:  "Multiple Dockerfiles: An Antipattern"
date:   2019-03-22
categories: quality automation
---

Docker has become very popular. It makes development and operation of an
application much easier, when used properly (as is the case with any other
tool). However, when used carelessly, it can cause frustration and issues that
one would not expect.

One antipattern that I've seen with Docker is the use of multiple `Dockerfile`s
for different environments. Some projects will have a `Dockerfile.dev` for
local development, `Dockerfile.pre-prod` for the pre-prod, and
`Dockerfile.prod` for production. Let's talk about this.

While such a setup will work, it has some drawbacks that are worth avoiding. The
most obvious of these is that all environments are actually running
_different_ things. Each of these `Dockerfile`s will produce _different_
images. What this means is that whatever is tested on a dev machine is
_different_ than what is tested in staging, and that is again _different_ than
what is actually deployed to production. Not an ideal situation.

The ability to test locally and in staging that which is about to go out to
production is _a good thing_. Testing anything even mildly different than that
which is about to go to production leaves room for a variety of issues and bugs
that simply cannot be detected in pre-prod.

Separate Dockerfiles for different environments is a bad idea. It is an
antipattern. The closer we get to full environment-partiy, the fewer issues
we encounter in production.

## Other benefits?

Yes. With a separate Dockerfile, we're likely to have separate builds for
production than we do for pre-prod. With every deploy, we spend time and
resources building the "production version" of this thing.

That is, not enticing. If we have just finished testing our next release,
everything looks good, and we're ready to deploy, why should we have to wait for
yet _another_ build to go through before we can actually release the thing?

This is a non-issue when we have a single Dockerfile in place. Whatever was
built and tested for pre-prod can (and should) be pushed to prod. No
re-building or waiting required.

## So why do some projects do this?

Possibly the most common reason for needing multiple Dockerfiles is
configuration. Production has different configuration than pre-prod, and
pre-prod has different configuration than development. Depending on how the
project is structured, and the language and tooling used around it, this is
almost always a problem that can be easily solved. Configuration needs to be
_injected_ into a container at runtime. A container should not startup with all
of its configuration already baked in.

The simplest example of this is probably the classic database connection string.
This value should either be passed in as an environment variable, or live in a
configuration file that can be chosen at startup. Configuration settings such as
this should _not_ be provided at build time. They should _not_ be baked
permanently into artifacts such as docker images.

When starting up a new container, the orchestration system responsible for
starting the container should provide the configuration that is needed by the
container. With this mechanism in place, the _same_ image can be used to run in
dev, pre-prod, or production, but with _different_ configuration _per_
environment, provided at runtime.

This is good. What it affords us is the ability to run the same image in dev,
the same image in pre-prod, and the same image in production. As an added
bonus, it eliminates multiple, nearly-identical Dockerfiles that no longer need
to be maintained.

## Other options?

I mentioned that configuration can be passed in as environment variables. If
there are many settings, this may become hard to manage. Configuration files can
be used when the list of options grows large.  Another option is to use some
external system (i.e. HashiCorp Consul and/or Vault, Apache Zookeeper,
Kubernetes ConfigMaps) to store configuration. This may be a better alternative
when the number of projects grows large and it is desireable to have a central
place for viewing and managing all configuration settings.

With externalized configuration, containers simply get a reference to where they
should pull their config from, and it is up to the container to fetch the
remaining config settings. While this may be better than having the
orchestration framework provide all settings to the container, it is a little
more effort. Make sure to weigh the tradeoffs and the costs before moving to
something like externalized configuration.

## Next steps?

Well, let's find out why we have multiple Dockerfiles. How different are they?
_Why_ do they need to be different?

Maybe we find that they really _don't need_ to be different. The merge may be
simpler than we thought. On a relatively young project, this may very well be
the case.

A project that's been through several world wars may be a different story,
however. Different Dockerfiles have a tendency, over time, to become even _more
different_. Different hacks, tricks, and one-offs have likely been added to one
of the Dockerfiles but not the other. If that is the case, the effort has just
increased by several orders of magnitude. We must now understand why the mess
exists before we start to detangle it. We are likely looking at a marathon
rather than a sprint.

## Booby Traps

Be wary of `Dockerfile` build arguments. Although they can seem like the perfect
solution towards one Dockerfile and environment parity, they might not get us
away from separate builds (and hence, images) for different environments.

A common use case for build-args is so that we can "pass this for pre-prod and
that for prod". There are other legitimate uses for it, but this specific
scenario means that we still have work to do before bridging our environmental
differences.

What is it that we are passing using build args? Can we pass that in as an
environment variable at runtime? Can we pass it in as a command when we create
the container?

Maybe we can't get away from build args right away. Maybe we the best we can do,
without additional signifacant work, is to merge our Dockerfiles into one _and_
use build args. That's still better than what we had and certainly a step in
the right direction.

## Last steps?

Well, now that we have one Dockerfile, we can optimze the build/deploy
pipelines. We may be able to replace our separate builds with just one build.
Our deploy pipelines can simply see what is the last thing that was in pre-prod
that was tested, and deploy _that_ to production.

We can enjoy a little more confidence that things will work as expected and
bugs will now have fewer cracks through which they can slip through into
production.

Enjoy a simpler CI/CD pipeline!
