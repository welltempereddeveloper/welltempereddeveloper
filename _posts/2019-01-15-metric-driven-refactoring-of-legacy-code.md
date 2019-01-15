---
title:  "Metric-Driven Refactoring of Legacy Code"
date:   2019-01-15
categories: refactoring
---

Chances are that you've worked with legacy code in the past. You may even be
working on it today.  There are many tools and techniques that help with this
sometimes-daunting task.  However, one method that I'm not hearing all that much
about is the use of metrics to help guide refactoring.

The premise of metric-driven refactoring is that you have available some metrics
related to what your application is doing. Those metrics should give you
valuable insight into the health of your application.

Before we get into details, I just want to highlight that metrics shouldn't be
your _primary_ tool for refactoring. Observing metrics for determining success
of a refactor has a very slow feedback loop. Metrics should be used as a
_secondary_ mechanism that provides quantitative data about your refactor.

So, for example, use metrics to see if your refactor really helped speed
something up. Use metrics to see if your Garbage Collection tuning efforts did
result in better garbage collection. And finally, use metrics to see whether or
not you broke something without knowing it.

### So let's get a few examples of metrics

Take, for instance, a service that performs authentication. A few good metrics
for such an app could be:

* rate of succesful auths v.s. failed auths
* average response time for successful auths and failed auths

Another app could be some batch app that processes some large files that your
clients upload. This app could publish metrics such as:

* average duration per file
* number of retries per file (if relevant)
* number of files processed per minute
* number of "things" imported

Now, say that you're tasked with tweaking this batch app that has few or no
tests. Your job is to make it do this "one small additional" thing.

On the surface, the task sounds easy, but in reality, this batch app hasn't been
touched in 2 years. Nobody is familiar with it and the code is less-than
pristine.

If you have metrics, then clean up some code, add some tests, make the necessary
changes, deploy, and watch what happens. Did processing time increase? Is the
process still importing the "same" number of things? When all metrics are still
"looking good" after your changes, you can be fairly confident that things are
still dandy.

### Ok that's cool, but this thing doesn't export metrics

So this is where things will take a bit longer and will likely be a two step
process. Why? Because you'll be adding metrics.

You need metrics. Metrics are your window into the world of the application at
hand. It's your answer to "is this thing even doing anything?" It's the
application's EKG.

If you're adding metrics, make sure to do your due diligence and add _relevant_
and _informative_ metrics. This means that you'll first need to understand what
the app is doing, if you don't already, and second, you'll need to find out
where is the best place to collect and publish these metrics.

It is important that you _do not_ perform other refactoring at this point.
You'll get to do that later, but if we have no insight into what our application
is doing, you want to minimize or completely eliminate any possibility of
breaking things or slowing them down. Again, it may be tempting, but withold
yourself from any refactorings at this point. Add the metrics, get out, and
deploy.

In the case of something with extremely high throughput, you'll likely want to
do some sort of buffering/aggregation _before_ publishing, so as not to slow the
thing down.

Once you have some metrics in place, redeploy. Make sure to test as thoroughly as
possible before production, and get it out into production to start collecting
real metrics. Again, I can't stress this enough, make sure you aren't actually
deploying any refactorings or other changes at this point. We only want to get
our hands on numbers to which we can later refer later, when we _do_ perform
refactorings and optimizations.

### How much data do I need?

There is no hard data here. This is really application-specific and also
dependent on your risk tolerance. If you have something that's churning along
constantly, all day every day, then a few hours may be sufficient to get good
insight into what's happening. If you can afford a few days, even better.

If you haven't actually done any work, other than the refactoring, then you'll
probably need a day or two to actually finish your task. That may be enough to
collect good data.


Other applications may actually need a week or two. If you have traffic that
greatly fluctuates (i.e. high peeks only on the weekends), then you may have to
wait until the weekend to get data that you need to make a good decision with
regards to "did I break anything or slow anything down". You may have to wait a
couple of weekends.

Worst case scenario, and get ready for this, you may have to tweak the metrics
you're collecting. Take for example, a set of metrics that you were expecting to
fluctuate, but upon deploying them you discover that they're actually pretty
stable. Perhaps you're collecting the wrong metrics? Perhaps you're
misunderstanding how the application you're dealing with actually works?

Figure these things out. it's important to know the "why's" here. If you've
already went through the effort of collecting metrics, but you're collecting the
_wrong_ metrics, then you're actually worse off then you were before. Best case
scenario, you have useless metrics. Worst case scenario, you have _misleading_
metrics, which can lead to bad decisions, which case lead to lots of headaches.
