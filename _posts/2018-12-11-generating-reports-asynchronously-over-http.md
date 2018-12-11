---
title:  "Generating Reports Over HTTP"
date:   2018-12-11
categories: asynchronous reports framework
---

Reports. We love them. We hate them. We need them.

Reports give us valuable insight into our business. They provide us with a
deeper look into what's happening within companies or with products. Reports
provide various perspectives on information which we need to make educated
decisions for the future.

## Generating Reports over HTTP

There are many ways to do this, but from the perspective of our users, we
essentially have two options: synchronous or asynchronous.

Both options have their perks and drawbacks. For this comparison, I'm going to
ignore the merky details of _producting_ the reports (since those details are
application-specific anyway) and only focus on what goes into _requesting_ and
_delivering_ the report.

## Synchronous Reports

### Benefits

These are simple. When a client needs a report, they make an HTTP request to
some endpoint and potentially provide a few filters or options. A web server
accepts the request, produces the report, and sends it back in the body of the
response to the client.

Again, ignoring any details of actually _producing_ the report, there is really
not much to add there. There are (relatively) few things that can go wrong in
this scenario. The general flow is: accept request, create report, send report
back. Done.

If there is a network error, the client can retry. If the report needs
regeneration, the client can retry. And for small reports, this may be all that
we need.


### Drawbacks

The drawbacks of this approach really begin to manifest themselves when we
begin experiencing failures, especially during periods of high load. When
things start to go sideways in a synchronous reporting framework, that's when
it starts to hurt.

It's worth noting that although we can certainly engineer around these issues
with a synchronous framework, it's much easier to do so with an asynchronous
framework. Some of these aren't even a problem in an asynchronous framework.

#### Retries due to failure

Things can and will go wrong. When generating gigantic, resource-intensive
reports, things will fail right at the very end. And, if we have a synchronous
framework in place, then the only thing the client can do is to retry the
request again. What that entails on our end is doing all of that
resource-intensive work all over again (with no guarantees that it will pass
the second, or third time around).

#### Tied-up server threads

When reports are synchronous, we will end up with connections from clients that
are simply waiting for reports to be generated. This will eat up server threads
that are blocked doing nothing. They _could_ be serving other requests, but
instead they are twiddling their thumbs waiting for some report to be generated.

Ideally, connections to the web server are utilized as much as possible. A quick
request-response turn-around time is desireable, as it mitigates the potential
for network partitions and other "gotchas".

#### High chance-of-failure for time-consuming reports

This is really a culmination of the previous two points. It will be practically
impossible for a synchronous reporting framework to reliably service
time-consuming reports. A network or system failure somewhere is imminent.
Something will go boom, and clients will need to retry, over, and over again.

At this point, we will be _forced_ to implement something smart on the
reporting end to allow for "retries" without actually retrying from the
beginning. We will likely be looking at asynchronous solutions.


#### Retries due to page refresh

Clients can get impatient. If it is a human requesting a report, and things are
taking slightly longer than usual, we will end up generating the same exact
report multiple times. These repeated requests will up even more resources. If
things are still going slow, then clients will request the report many, many
times over. This will trigger an avalanche of chaos and our entire system will
have a very, very bad day.

## Asynchronous Reports

The general flow of asynchronous reports is:

1. Client makes an HTTP request for a report.
1. Server accepts request during which:
1.1 It serializes the request onto a queue for later processing.
1.1 It creates a token of some sort, which will later identify the finished
   report.
1.1 It response to the client with the generated token.
1. Some out-of-band mechanism, ideally outside of the webserver, begins
   generating the report.
1. The client will request the report, providing the token from the initial
   request.
1. At this point, if the report is ready, it is returned to the client.
1. If the report is _not_ ready, a 404 (or whatever we'd like) is returned and
   the client can try again later.

### Drawbacks

Let's start with drawbacks here because there is one main drawback that's
obvious: complexity. Asynchronous reports are, without a doubt, more complex to
implement.

On top of that, they are a bit harder to consume. You may be doing yourself a
great disfavor if you provide an asynchronous reoprting mechanism but no one is
able to actually implement a proper consumer. You will be reporting to, nobody.

### Benefits

In addition to avoiding all of the drawbacks mentioned for synchronous reports,
asynchronous reports give you a lot of wiggle room for many improvements.


#### Cheap re-requests

The first and foremost benefit here is that once a report has been generated,
subsequent requests for that report are very cheap. The only overhead at that
point is network bandwidth. That's not to say that you _couldn't_ do this with
synchronous reports, but this optimization comes for free, out-of-the-box with
asynchronous reports.


#### Long-running reports do not eat up web resources

In addition, they're actually much more likely to succeed. With every second
that passes on a live connection, there is a possibility that the connection can
be dropped, for many reasons. Upon the request of a report, if the only thing
that we're doing is dropping a message on a queue somewhere (for later
processing) then that request/response loop becomes very quick. At this point,
there is really very little room for failure.

#### Smart retries

Because we are no longer constrained by a live network connection just waiting
to close on us, we can get smarter about retrying portions of the report. When
something _does_ go wrong during generation, we can take our time to recover
gracefully and actually complete the report, as opposed to abandoning it
completely and trying again, from scratch, on a subsequent request.

#### Many open possibilities

Lastly, asynchronous reporst open the door for optimizations and features that
can be implemented later, with much greater ease.

A few of them include:

1. A "Schedule for later" feature.
1. Making changes to an existing report request before it is completed.
1. Cancelling a resource-intensive report.
1. Moving report generation to a separate cluster, while leaving the
   client-facing UI together with the main app to keep things simple (more on
   this later).
1. Multiple consumers of the "report requests", such as an accounting app that
   charges users based on the number and types of reports that were requested.


## Other Considerations

Although not directly related to the decision of asynchronous v.s. synchronous,
there are a few other things to consider when providing a reporting system.

### Resources

If the demand for reports is expected to be anything _but_ negligible, then it's
probably a good idea to break up reporting entirely from other portions of your
system (i.e. don't put reporting in your main web app). Give the reporting
system it's own servers to minimize contention for these resources. If
possible, a separate set of database replicated servers is not a bad idea.

A side-benefit to isolated resources will be visible if and when performance
issues arise. For example, if the web app is seeing an increase in error rates,
we can be pretty certain that it isn't reports that are bogging it down.

If you have the actual generation of reports separate, you may be able to leave
the client-facing app alongside your main app, if you don't want to manage
multiple applications. This decision will be specific to your business and IT
resources.

### Who are your clients

Knowing the users of your systems greatly helps you make better choices. As
mentioned before, asynchronous reports are more than just "hit this endpoint".
Be sure that the clients of your reporting service are capable of integrating
with it, otherwise you will have a glorious, shiiny reporting system that is
just sitting, smiling, and waiting for users that will never come.
