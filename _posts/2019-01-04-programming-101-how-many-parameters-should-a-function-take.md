---
title:  "Programming 101: How Many Parameters Should a Function Take?"
date:   2019-01-04
categories:  quality
---

If we're [keeping our functions short]({% post_url 2018-12-03-programming-101-keep-your-functions-short %}),
then we're probably not running into functions with [too many
parameters][too-many-parameters].

Why?

Because if a function really does just one thing, then there is really no need
for it to take, say, 10 parameters.


### So how many parameters _should_ a function take?

As many as it needs, _but_ with the caveat that the function is really doing
one thing and one thing only. If that's the case, we should never end up with
monster functions that need to be fed 20 parameters to do their thing.

Now, if we come across the need, here and there, to take 5 parameters, then it's
not the end of the world. It's ok, really. _But_, if we're constantly doing
that, then we'll soon be running into "now this function needs 6 parameters" and
"that function needs 7 parameters" territory. Whatever threshold we try to stay
under, we're at risk of crossing it.

### So what do we do about that?

Set a low threshold. We don't need to make it a hard fast rule, but a low
threshold will cause us to revisit these parameter-hungry functions sooner
rather than later. The sooner the visit, the less likely that things will get
out of control.

In addition, the lower the threshold, the _more frequently_ we work on
refactoring and cleaning up these functions. This in turn means we get better at
it. If we're really good at it, then naturally we begin to avoid writing them in
the first place. We'll be able to _foresee_ when things like this are likely to
get out of control and stop them from happening in the first place.

Practice makes perfect, right?

### But I really, really, really need 12 parameters!

There are exceptions. If we're configuring some object, it's totally possible
that we need to pass in five, ten, or even twenty different configuration
parameters. Although not all that common in every day coding, the need for this
does come up.

For this specific use case, would could use the [Parameter Object
pattern][parameter-object]. It replaces the need to pass a slew of arguments when
calling a function in favor of setting fields (or keys) on one specific
"configuration" object. That single object then gets passed into the relevant
function.

The result? Clean(er) code.


### Alternate options?

Yes. Depending on your situation, you may want to look at the [Factory Method
pattern][factory-method] or [Provider model pattern][provider-model] instead.
In some use cases, these patterns are more appropriate than Parameter Object.


[too-many-parameters]: http://wiki.c2.com/?TooManyParameters
[parameter-object]: http://wiki.c2.com/?ParameterObject
[factory-method]: https://en.wikipedia.org/wiki/Factory_method_pattern
[provider-model]: https://en.wikipedia.org/wiki/Provider_model
