---
title:  "Simple Jenkins Execute Shell Scripts"
date:   2018-11-16
categories: jenkins
---

I know it's 2018, but I'm pretty sure there are Jenkins jobs out there that use
the Jenkins "Execute Shell" step. Chances are that those jobs are _not_ going
away tomorrow, they are _not_ moving to Jenkinsfile's, and they are here to
stay–at least for a little while longer.

_Ok, fine._

If we're not touching them, and they work, and there is no reason to touch
them, then just leave them be and carry on with your business as usual.

_But!_ Suppose that we _are_ modifying them. Can we do anything to make them
better?

Yes! Let's start by clicking on the help icon, directly to the right of the
giant text box where our beautiful shell scripts live. And let's pay special
attention to the very last paragraph in that help box:

![Jenkins Execute Shell Help](/assets/images/jenkins-execute-shell-help.png)

That line is _exactly_ how we can and should be invoking any and all shell
scripts relevant to the job. We can store them in source control and simply
invoke them in that gigantic textbox. That textbox should never ever have more
than one line in there, which is the invocation of the script.

### Why?

There are many reasons, but one of the most important ones is at the very end
of that help box: "so that you can track changes in your shell script." There
is great value in having any sort of scripts versioned and in source control,
but to put it succinctly, you want to know what's changing and when it's
changing. It's a good thing.


### Anything else?

Other reasons include the ability to test changes to that script _without_
affecting everybody on the team who depends the job (and without disrupting the
job itself). If the script lives in source control, then testing out changes to
the script–no matter how small or how drastic–is a matter of making changes on
a branch and running the job against that branch. That means, no disruptions.

### What else do you get with that?

Well, if you can branch off and make improvements on a branch, you get a little
bit of collaboration with that. Create a pull request with your changes to
solicit feedback from your peers. That's also a good thing.

### Still not convinced?

Suppose two weeks pass and people start noticing some weird behavior about the
job. Source control to the rescue! Let's take a look at our revision history to
see what was changed that may be relevant. Easy.

### What were the drawbacks again?

Changing the script outside of source control (and within Jenkins) is just
painful. Not only are you coding in an HTML textbox, but you may also be
disrupting others (_unless you've cloned the job and are testing there, but who
wants to do that_).

There is no easy way to rollback. There is no way to see when changes were
made, what changes were made, and by whom they were made. It is error-prone and
just plain dirty. You do _not_ want to be doing any serious scripting in a
Jenkins HTML textbox.

### So, now what?

If you find yourself coding in an HTML textbox in Jenkins, then please move all
code _out of_ that textbox and into a file in your repository. In the textbox,
invoke that file and save the job. From that point on, any and all changes to
whatever script used to live in the textbox should be made to the file in your
repository. Enjoy!
