# How to (and not to) store values

You've used a database. You've written data, updated data, and read data.
You've manipulated data and sent it off to glorious APIs.

### Let's focus here on the formatting (or lack thereof) of stored data.

Once in a while, you'll come accross the need to use data that has,
unfortunately, been pre-formatted for one reason or another. That's
inconvenient.  The data is there, but you actually have to do a little extra
work to really get your hands on it.

As an example, let's take a password to a third-party service. This third-party
service isn't ideal–as often times is the case–and for most interactions with
that service a REST-ful API is available. For a few legacy, yet crucial
features, an FTP service is available to which files can be uploaded for
processing.

The same username and password must be used for authenticating with both
services.  Now, for simplicity, let's say the REST-ful API uses HTTP basic
authentication over SSL. This means that in order the authenticate, you must
provide the `Authorization` HTTP header with a value of `Basic <value>`, where
`value` is the base64-encoded version of your username concatenated together
with your password and a colon (`:`) in between them. For example, if your
username is `john` and your password is `password`, then the value of the
header would be: `Basic am9objpwYXNzd29yZA==`.

### Now what?

Since that is essentially what we use to authenticate, a few developers will be
tempted to store that exact string as the "password" (wherever it is to be
stored), so that upon retrieval it is ready to just plop directly into the HTTP
header when communicating with the REST-ful API. Done.

### But what about the FTP integration?

That needs to authenticate as well.

Oh, well, that integration can just base64 decode that string ... but only
after the `Basic ` prefix is parsed out. Oh yeah, once it is base64 decoded it
must then be split again on the colon in order to separate out the username
from the password. And, oh yeah, _because_ the password _may possibly_ contain
a colon itself, pulling out the password is not as simple as calling a function
like `string.split(':')` available in many libraries. The password is really
everything after the first colon.

So, as you can see, things just got a little complicated. Taking a innocuous
shortcut on one side (the REST-ful API integration) ended up causing some
headaches and left a lot of room for error elsewhere.

A better alternative, as may be obvious by now, is to have stored the username
and password separately, in their raw format, un-encoded. If a
protocol-specific operation needs to take place before that password can
successsfully be used (i.e. concatenation and base64 encoding), then that
operation should happen wherever that is relevant (in our case, the service
that uses the REST-ful API). The protocol and other application-specific
details should not leak into the storage of the data.

Things get further complicated when it comes time to rotate the password.
Someone without much context on where or how this gets used (or even, the
original author a few months down the road) may not know that this string
needs to be base64 encoded and will end up simply changing the value to the
un-encoded password. Obviously, not good.

What someone with little or no context would _expect_ to see is one field for
password (the raw password) and a separate field for username (raw username).
Such a setup leaves _very_ little room for error, and that's a good thing.

### Be mindful of what and how you format before persisting!

Lastly, please be mindful that this article was focused on the _format_ in
which data is stored. I hope to discuss concerns such security for data at rest
or in transit in a separate article. There is, without a doubt, _much_ to cover
on that front.

If you enjoyed this article, then you may also like [Principle of Least
Astonishment](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)
