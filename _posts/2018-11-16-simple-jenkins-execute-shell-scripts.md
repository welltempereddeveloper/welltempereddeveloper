---
title:  "Simple Jenkins Execute Shell Scripts"
date:   2018-11-16
categories: "continuous integration" "continous deployment" jenkins
---

I know it's 2018, but I'm pretty sure there are Jenkins jobs out there that use
the Jenkins "Execute Shell" step. Now, for one reason or another, those jobs
are _not_ going away anytime soon, they are _not_ moving to Jenkinsfile's, and
they are here to stay ... at least for now.

_Ok, fine._

If we're not touching them, and they work, and there is no reason to touch
them, then just leave them be and carry on with your business as usual.

_But_! Suppose that we _are_ modifying them. Can we do anything to make them
better?

Yes! You can start by clicking on the help icon, directly to the right of the
giant text box where you can paste the entirety of your beautiful shell script.
Please pay special attention to the very last paragraph in that help box:

![Jenkins Execute Shell Help](/assets/images/jenkins-execute-shell-help.png)

That line is _exactly_ how you can and should be invoking any and all shell
scripts relevant to your job. Store them in source control and invoke them in
that gigantic textbox. That textbox should never ever have more than one line
in there, the path to your script relevant to the job in which you're in.

### Why?

There are many reasons, but one of the most important ones is at the very end
of that help box: "so that you can track changes in your shell script." There
is great value in having any sort of scripts versioned and in source control,
but to put it succinctly, you want to know what's changing and when it's
changing. It's a good thing.


### Anything else?

Other reasons include the ability to test changes to that script _without_
affecting everybody on the team who depends the job. Testing out changes to the
script, no matter how drastic, is a matter of making changes on a branch and
running the job against that branch. That means, no disruptions.

### What else do you get with that?

Well, if you can branch off and make improvements, you get a little bit of
collaboration with that. Create a pull request with your changes to sollicit
feedback from your peers. That's also a good thing.

### Still not convinced?

Suppose two weeks pass and people start noticing some weird behavior about the
job. Source control to the rescue! Let's take a look at our source control
history to see what was changed that may be relevant. Easy.


### What were the drawbacks again?

Changing the script outside of source control (and within Jenkins) is just
painful. Not only are you coding in an HTML textbox, but you may also be
disrupting others (_unless you've cloned the job and are test there, but who
wants to do that_).  There is no easy way to rollback. There is no way to see
when changes were made, what changes were made, and by whom they were made. It
is error-prone and just plain dirty. You do _not_ want to be doing any serious
scripting in a Jenkins HTML textbox, and you may have already felt the pains of
doing that.


### So, now what?

If you find yourself coding in an HTML textbox in Jenkins, then please move all
code _out of_ that textbox and into a file in your repository. In the textbox,
invoke that file and save the job. From that point on, any and all changes to
whatever script used to live in the textbox should be made to the file in your
repository. Enjoy!
