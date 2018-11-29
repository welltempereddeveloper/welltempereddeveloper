---
title:  "Ansible Playbooks Should Not Know About Hosts"
date:   2018-11-28
categories: ansible
---

I've seen this here and there. Going through the tasks in a role, I'll see
something along the lines of: `when: ansible_hostname == "web-server-x"`.

And that's when the red flag goes up.

### The role is now dependent on inventory

That's _not_ a desirable thing. Dependencies in general are not something to
envy, but this one in particular should be avoided.

What this means is that anytime a hostname changes (yes, that _does_ happen in
the real world), the role must change. Not very nice.

What this means is that if that task ever needs to run on another hosts, then
the task (and the role that contains it) must change. Not very nice.

What this means is that if that host ever goes away, the role is left with a
task that will never ever be used. That's the equivalent of unreachable branches
in code. Again, not very nice.

What this means is if someone is ever debugging why on earth a role would behave
differently when run against this host v.s. that host, they have to dig deeply
into a role to find that specific when clause. Depending on the complexity of
the role, that could mean hours wasted. Not very nice and, in addition, very
frustrating.

### So what's the alternative?

First of all, I would say get rid of this special scenario. If a role does
something different for host X and then different for host Y, maybe it should
really be two separate roles? Perhaps that one specific task doesn't belong in
this role? The causes for this special scenario may be numerous, and so would
the solutions, but finding ways to avoid this would be my first recommendation.

But suppose we've done our due diligence and we can't make that happen. We need
that when clause because we really only need that to happen in some scenario.

One strategy that works well is the introduction of a role-specific variable
which can be set on specific hosts that need it. It's not ideal, but it does
decouple the hosts from the roles. And that's a good thing.

What this looks like is the `when` clause no longer looks at the the hostname,
but at a different variable defined in the role (with a sane default). The only
thing it knows about is the existance of this variable. It knows nothing about
hosts. In order to determine whether or not the task should execute, it will
check against the value of that variable.

Hosts that need this task executed are no longer hardcoded in the role. Instead,
the role-specific variable for the corresponding hosts is set accordingly, and
the outcome is the same.

### Got a specific example?

Yes.

Suppose we have ourselves cluster of three servers that are responsible for
logging. We want to prune logs every so often and we'll set that up as a
crontab.  However, because of some constraints, only one of those three nodes
are allowed to perform the pruning.

When we provision the servers with our hypothetical logging role, we only need
to make sure that appropriate variable is set for logging-server-1 and we're all
set. Our role knows absolutely nothing about any of our inventory.

So in the end, we have something like this:

{% highlight yaml %}
# defaults.yaml in role
prune_logs: false

# tasks.yaml in role
- name: Prune logs
  cron:
      name: "Prune logs"
      special_time: "hourly"
      job: "echo pruning old logs"
  when: prune_logs

# inventory host_vars for logging-server-1
prune_logs: false
{% endhighlight %}


With this setup we get none of the drawbacks we described earlier. Our role is
much less likely to change. We can change where to run (or not to run) the cron
without touching our cron.

Debugging becomes easier as well. Looking at host-specific (and group-specific)
vars is an intuitive place to go to when trying to determine why a role would
behave differently on different places.

### Closing it up

This is a very specific example, but the general concept can be applied in many
different concepts. Be diligent about introducing dependencies and coupling
entities that do not need to be coupled. Consider alternate solutions to the
problem at hand and always be wary of sneaky dependencies creeping in.
