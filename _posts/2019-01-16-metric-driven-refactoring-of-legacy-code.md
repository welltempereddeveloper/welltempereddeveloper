---
title:  "Metric-Driven Refactoring of Legacy Code"
date:   2019-01-16
categories: refactoring
---

Chances are that you've worked with legacy code in the past. You may even be
working on it today.  There are many tools and techniques that help with this
sometimes-daunting task.  However, one method that I'm not hearing all that much
about is the use of metrics to help guide refactoring.

The premise of metric-driven refactoring is that you have available some metrics
related to what your application is doing. Those metrics should give you
valuable insight into the health of your application. If you refactor something
and metrics get worse, then you should probably revert.

Before we get into details, I just want to highlight that metrics shouldn't be
your _primary_ tool for refactoring. Observing metrics for determining success
of a refactor has a very slow feedback loop. Metrics should be used as a
_secondary_ mechanism that provides quantitative data about your refactor.

### Examples

Take, for instance, a service that performs authentication. A few good metrics
for such an app could be:

* rate of succesful auths v.s. failed auths
* average response time for successful auths and failed auths

Another example can be some batch app that processes large files uploaded by
your clients. Relevant metrics for this app may be:

* average processing duration per file
* number of files processed per minute
* number of "things" imported
* number of retries per file (if relevant)

Now, say that you're tasked with tweaking this batch app. Your job is to make
it do this "one small additional" thing.

On the surface, the task sounds easy. In reality, this batch app hasn't been
touched in 2 years. Nobody is familiar with it and the code is less-than
pristine.

If you have metrics, then clean up some code, add some tests, make the necessary
changes, deploy, and watch what happens. Did processing time increase? Is the
process still importing the "same" number of things? When all metrics are still
"looking good" after your changes, you can be fairly confident that everything
is still ok.

### Ok that's cool, but this thing doesn't export metrics

So this is where things will take a bit longer and will likely be a two step
process. Why? Because you'll be adding metrics.

You need metrics. Metrics are your window into the world of the application at
hand. It's your answer to "is this thing even doing anything?" It's the
application's EKG.

If you're adding metrics, make sure to do your due diligence and add _relevant_
and _informative_ metrics. This means that you'll first need to understand what
the app is doing, if you don't already. Secondly, you'll need to find out where
is the best place to collect and publish these metrics.

It is important that you _do not_ refactor anything at this point. You'll get
to do that later. While we have no insight into what our application is doing,
we want to minimize or completely eliminate any possibility of breaking things
or slowing them down. Again, it may be tempting, but withold yourself from any
refactorings at this point. Add the metrics, deploy, and observe.

### How much data do I need?

This depends on, well, lots of thigns. It depends on the app, on what it's
doing. If you have something that's churning along constantly, all day every
day, then a few hours may be sufficient to get good insight into what's
happening. If you can afford a few days, even better. If the thing doesn't
really do all that much, then you may need a couple of days or weeks.

There is also a risk-tolerance factor that you have consider. Mission-critical
code on which the entire business depends will likely need more metrics than
batch process on which two people depend.

Still, other applications may actually need a week or two. An application that
has fluctuating traffic (i.e. high peeks only on the weekends) may need a couple
of weekends of data before you can make a good decision with regards to "did I
break anything or slow anything down".

Worst case scenario, and get ready for this, you may actually have to go back
and add more metrics. Take for example, you instrumented your app to start
collecting a set of metrics that you were expecting to fluctuate. However, upon
deploying the app you discover that they're actually pretty stable. Perhaps
you're collecting the wrong metrics?  Perhaps you're misunderstanding how the
application you're dealing with actually works? Understand why and act
accordingly.

Take time to understand these things. It is important to answer all of the
"why's".  If you go through the effort of collecting metrics, but end up
collecting the _wrong_ metrics, then you're actually worse off then you were
before. Best case scenario, you have spent time collecting useless metrics.
Worst case scenario, you have _misleading_ metrics, which can lead to bad
decisions, which case lead to ... bad things.

### What's next?

Refactor away. As with everything, take small steps here. Don't drop the app on
the butcher block. Be deliberate about how much you actually change in between
releasing.

Always monitor the metrics you're collecting for any changes that could be
indicative of a problem (i.e. lower throughput, longer processing times).
Ideally, you have alerts set up for when key metrics cross some thresholds.
Hopefuly, these will give you an extra level of confidence on your current and
future refactoring endeavors.
