---
title:  "Ansible Playbooks Should Not Know About Hosts"
date:   2018-11-28
categories: ansible
---

Going through the tasks in a role, I'll occasionally see something along the
lines of: `when: ansible_hostname == "web-server-x"`. That raises a red flag.

### The role is now dependent on inventory

That's _not_ a desirable thing. Dependencies in general are not something to
envy, but this one in particular should be avoided.

What this means is that anytime a hostname changes (yes, that _does_ happen in
the real world), the role must change. Not very nice.

What this means is that if that task ever needs to run on another hosts, then
the task (and the role that contains it) must change. Not very nice.

What this means is that if that host ever goes away, the role is left with a
task that will never ever be used. That's the equivalent of unreachable branches
in code. Not the end of the world, but still, not very nice.

What this means is that if someone is trying to answer the question, why on
earth would a role behave differently when run against host X v.s. host Y, they
would have to dig deeply through the bowels of this particular role in order to
find the answer. Depending on the complexity of the role, that could mean hours
wasted. Not very nice and very frustrating.

### So what's the alternative?

First of all, I would say get rid of this special scenario. If a role does
something different for host X and then different for host Y, maybe it should
really be two separate roles. Perhaps that one specific task doesn't belong in
this role. The causes for this special scenario may be numerous, and so would
the solutions, but finding ways to avoid this would be my first recommendation.

But suppose we've done our due diligence and we can't make that happen. We need
the `when` clause because it's something that absolutely needs to happen on one
specific host.

One strategy that works well is the introduction of a role-specific variable
which can be set on specific hosts that need it. It's not ideal, but it does
decouple the role from inventory. And that's a good thing.

The `when` clause no longer checks the value of a variable specific to
inventory, but rather it checks the value of a variable defined in the role
itself. The only thing it knows about is the existance of
this variable. It knows nothing about hosts.

Hosts that need this task executed are no longer hardcoded into the role.
Instead, the role-specific variable for the corresponding hosts is set
accordingly, and the outcome is the same.

### Got a specific example?

Yes.

Suppose we have ourselves a cluster of three servers that are responsible for
logging. We want to prune logs every so often and we'll set that up as a
crontab.  However, because of some constraints, only one of those three nodes
are allowed to perform the pruning.

When we provision the servers with our hypothetical logging role, we only need
to make sure that appropriate variable is set for `logging-server-1` and we're
all set. Our role knows absolutely nothing about any of our inventory.

To put into yaml, what we end up with is something line the following:

{% highlight yaml %}
# defaults.yml in role
prune_logs: false

# tasks.yml in role
- name: Prune logs
  cron:
      name: "Prune logs"
      special_time: "hourly"
      job: "echo pruning old logs"
  when: prune_logs

# inventory host_vars for logging-server-1
prune_logs: false
{% endhighlight %}

Which is preferred over:

{% highlight yaml %}
# tasks.yml in role
- name: Prune logs
  cron:
      name: "Prune logs"
      special_time: "hourly"
      job: "echo pruning old logs"
  when: ansible_hostname == "logging-server-1"
{% endhighlight %}

With this setup we get none of the drawbacks we described earlier. Our role is
much less likely to change. We can change where to run (or not to run) the cron
without modifying the task that sets it up.

Debugging becomes easier as well. Looking at host-specific (and group-specific)
vars is an intuitive place to go to when trying to determine why a role would
behave differently on different machines.

### Closing it up

This is a very specific example, but the general concept can be applied in many
different contexts. Be diligent about introducing dependencies and coupling
entities that do not need to be coupled. Consider alternate solutions to the
problem at hand and always be wary of sneaky dependencies creeping in.
