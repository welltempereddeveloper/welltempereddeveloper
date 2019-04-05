---
title:  "Jenkins High Availability"
date:   2019-04-08
categories: ci cd
---

[Jenkins][jenkins]. A pretty common choice for CI/CD pipelines, scheduled tasks,
and many, many other things. Life is good when Jenkins is up and running. Life
is also good when Jenkins is patched and up-to-date.


Unfortunately, we all know that this isn't always the case. Jenkins goes down
because we have to update some things, or because the host on which it's running
goes down, or because we run it in [Kubernetes][kubernetes] and our pods have to
migrate every once in a while.

## So, can we make Jenkins highly available?

Jenkins is [about 15 years old][jenkins-wikipedia] now. It's default and (only)
supported setup is to run a single master powered by an filesystem "database".
It keeps lots of things in memory, and any restart of the main process is
anything _but_ transparent.

It simply wasn't designed to be HA and is unlikely to ever be that way.

So how can we get our 100% uptime, keep it quick, and get all our goodies? Well,
the answer is, we don't.

Search the internet for "jenkins high availability" or "jenkins multi-master"
and we'll get a lot of results for how we can do this and try that and, with
various levels of effort, get ourselves a _little_ closer to HA. Some suggest a
standby master reading off the same data directory as primary. Others suggest
file-replication between two masters running off their own directories.

There are many ways to get _something_ that _sort-of_ _almost_ feels like HA, or
gets us one step closer to it. Some approaches may get us what we're looking
for, but none of them really give us the full-blown HA that we (think we need)
or would like to have. Jenkins was not designed for HA, so any solution we
attempt at making it HA may be pitted with "gotchas" and head scratchers (i.e.
replication failed during on an important job run, two masters accidentally
wrote to same file, now what?)

## Are there other options?

Yes. Run multiple, independent Jenkins masters.

It is very likely that our jobs aren't all dependent on each other. If they are,
we've got bigger fish to fry.

But in reality, chances are that we can split up jobs into some logical
partitions and then distribute those across two or more _different_ Jenkins
instances. What we end up with then isn't exactly HA as some may define it. It's
not redundant masters either.

What we have here are isolated instances of Jenkins masters that exist to run a
specific subset of jobs. This in turn means that we can take down one of those
instances, for whatever reason, _without_ affecting the entire organization.

In fact, the more Jenkins masters we have running, the higher the chances that
any particular one is sitting around idle. Shutting down or restarting any of
these instances is unlikely to affect anyone.

If a tree falls in the forest and there is no one around to hear it, [does it
still make a "sound"?][tree-sound]

 _And no, I'm not saying to go ahead and run master for each job either._

## What does it cost?

It's certainly not free. Let's first cover the downsides to this setup, since
they aren't particularly large.

### Management

The biggest downside is probably the fact that we now have to manage multiple
Jenkins masters. In reality, however, this shouldn't be a big deal. It's 2019.
We have infrastructure-as-code and there is almost nothing that we do manually.
There should be minimal or no overhead in managing multiple instances, and no
difference between managing 5 Jenkins masters or 50 Jenkins masters. Everything
should be tested and fully-automated so that rolling an update across all
instances or rolling back an update is a quick and safe.

### Many domains

Depending on how it's used, Jenkins may be a very UI-heavy tool. People visit it
to get information and run tasks. Each master will likely needs it's own domain
name. If we have 10 Jenkins masters running, finding the "correct" master on
which a specific job runs may be painful.

Fortunately, the number of masters will likely scale linearly with the size of
our organization. We may find it convenient to partition masters based on teams,
so each team has their own instance. In this case, each team only needs to
remember where their own instance runs. And if we include team names in the
domain name of each master, then finding corresponding masters for _other_ teams
is also trivial.

We can partition by projects, or departments, or something else that makes
sense. It doesn't have to be `jenkins-1.our-org.local` and
`jenkins-2.our-org.local`. [Name the instances well][naming-things] and
finding the one we are looking for may actually be a non-issue.

### Webhooks

This may or may not be an issue, depending on the existing setup. If we're using
a single hook in GitHub at the organization level, then we have a few options:

1. Define one hook per Jenkins instance at the GitHub organization level
2. Define hooks on each repository to point at the proper Jenkins instance
3. If we have a reverse-proxy, mirror webhook traffic across all Jenkins
   instances

Obviously, none of these are as appealing as "just send all hooks to one place",
but perhaps with the exception of option #3, these are all pretty
straightforward approaches.

## What are the benefits?

There are many.

### Distribution of Load

Since we're splitting up jobs across multiple masters, we naturally get load
distribution. I'm not referring to the load incurred by running jobs themselves;
those can be distributed entirely across Jenkins agents, regardless of the
number of masters that we run. I'm referring to the overhead that one Jenkins
master incurrs when managing 1,000 or 10,000 jobs.

This is probably most visible during a restart when the master must go through
all plugins and all jobs and their configuration. Each plugin takes time to
load. An instance with thousands of jobs (even a few hundred jobs) very likely
has a few hundred plugins.  That's a big hit on startup time.

There is overhead as well during normal operation. Scheduling, log gathering,
agent communication, and archiving are just a few things that the master is
responsible for. On top of that, if an instance has thousands of jobs, then it's
also probably serving a good deal of human traffic. All of this needs compute
power which, by default, only one master instance can serve. The only thing we
can do is throw CPU and memory at it, and that only goes so far (and comes with
its own drawbacks). Any sort of scaling here is out of the question here.

Splitting up jobs across multiple masters means we get faster startup time, a
faster UI, and the ability to move our masters across different hosts to spread
the load. Faster startup, in turns, opens the door to do more frequent patching and
updating. One minute of downtime is a lot better than ten minutes of downtime.

### Isolation

Some Jenkins jobs go bananas and wreak havoc. A misbehaved job, a typo in a
Jenkinsfile, a gigantic log file or artifact can bring an agent and its master
to a crawl.

If we have one master, then everybody is affected. If we have ten masters,
nicely partitioned, then only 10% of everybody is affected. I prefer the latter.

### Redundant locations for jobs

We can separate different jobs across different masters, but we can just as
easily duplicate jobs on multiple masters. There is nothing preventing us from
designating three or four masters for our job. Although we nullify the previous
two benefits by doing this, that may be desired in some scenarios.

When one master is down for upgrades, or just having a bad day, we can still run
our job on a different master. Have a job that gets running manually but quite
often? Put it on each master so people don't have to look for it.

Sometimes good. Not always necessary. But hey, it's an option now.

### Jenkins "Staging"

Am I really talking about a staging environment for Jenkins, where we can test
upgrades, new plugins, backups and restore procedures, any pretty much anything
else _without_ affecting any "production" workloads? Yes!

We can have this now, and with no additional effort. As I mentioned before, if
we go the route of many masters, then we should have everything automated at
least to the point where it doesn't make a difference if we're running 10
masters or 50 masters. What's running another master for the Jenkins staging
environment?


## Do we really need this?

It depends.

For some, it may be complete overkill. If there are only 20-30 jobs that run
during the day, and they're not production-critical jobs, then we can probably
get away with just one master.

If an organization doesn't value stability or high availability, then it
also doesn't make sense to do this. It's effort to get in place, and if the
benefits afforded by it are not valued, then that quickly translates to "we're
wasting time".

If an organization lacks the resources resources and discipline to fully
automate the management of many Jenkins masters, then pursuing it may not be
worthwhile in the long run. Especially if we are looking to run more than two or
three instances.

However, if the organization _can_ automate management of the entire fleet, then
there is much to be gained and such a setup is worth pursuing and implementing.

[jenkins]: https://jenkins.io/
[kubernetes]: https://kubernetes.io/
[jenkins-wikipedia]: https://en.wikipedia.org/wiki/Jenkins_(software)
[tree-sound]: https://en.wikipedia.org/wiki/If_a_tree_falls_in_a_forest
[naming-things]: https://martinfowler.com/bliki/TwoHardThings.html
