---
layout: post
title:  Browse and change code history with Git
tags:
- Git
status: publish
type: post
published: true
---

Sometimes, if I really want to understand a peace of software, I embark on browsing its whole history available in Git. I usually checkout its initial commit and than slowly make my way through the forest of subsequent commits and branches. If the project is the important one and has live and vibrant community I also consult mailing list archives for the point of time when a particular commit is made. This way, not only that I learn what were the original driving ideas of the current design, but often I gain deep insights about evolution and dynamics of the project .


I'll write now about some Git commands from its time-traveling-and-history-changing toolbox that I use in this setting.


Let's assume the following history exists and the current branch is "develop":


{% highlight text %}

               A---B---C---D---E---X develop

{% endhighlight %}


Now let's say that I'm on commit B where some particularly complicated algorithm is introduced. As I have hard time understanding the algorithm I write lots of printf's, comments and proof-of-concept code all over the place. Also, I often try to do the same thing that is done in the code differently and sometimes (especially when I make a mistake) I see why the author had chosen the particular way of doing it.

This meta-code helps me to really understand the original source and forces me to learn and adopt good practices and ideas from it. It has a tremendous value for me because if I need to check the algorithm at the point B again I will, hopefully, understand it and know how to reuse it much more quickly with this meta-code explanations. As I want to preserve this for the future I create commits along the way and usually group them in separate 'code-review' branch.

{% highlight text %}

git checkout -b code-review <B-sha1>

{% endhighlight %}

Next, I write explanatory comments, log messages and tests for commit B in commits F, G, H on the code-review branch so my git repository looks something like this

{% highlight text %}

                     F---G---H code-review
                    /
               A---B---C---D---E---X develop

{% endhighlight %}

When I'm done with reviewing commit B I move further down the main developing line to commits C, D, and E. Let's say the code there consists just of smaller tweaks of the algorithm that is introduced at the point B. I understand everything without need to write additional meta-code and I want to integrate them in my code-review branch so I can move on to another commits down the develop branch.

The easiest way to integrate C, D and E with *all* previous code-review commits (F, G, H) is to use git *rebase* command (at this point I would be in 'code-review' branch):

{% highlight text %}

git rebase <E-sha1>

{% endhighlight %}


So I end up with the following repository:


{% highlight text %}

                                 F---G---H code-review
                                /
               A---B---C---D---E---X develop

{% endhighlight %}

However, let's say I want to integrate only commits F and G with the C, D, and E from the develop branch. In that case I would use git *rebase* with --onto option as follows:

{% highlight text %}

git rebase --onto <E-sha1> <B-sha1> <G-sha1>
git checkout -b temp
git checkout code-review
git reset --hard temp
git branch -d temp

{% endhighlight %}

The first command basically says *move all commits between B (not including it) and G (including it) onto E commit*. After *rebase* is successfully finished the repository should look like this:

{% highlight text %}

                       F---G---H code-review
                      /
                     /           F---G (no-branch)
                    /           /
               A---B---C---D---E---X develop

{% endhighlight %}

There is a new (no-branch) HEAD that is in so called *detached* state.
In other words it's simply a commit hash which isn't pointed to by a tag or a branch.
Basically, whenever you check out a non-referenced head, you end up with a detached
head. In practice some conflicts may arise at this stage. To efficiently resolve them I recommend [git-rm-conflicts](https://github.com/jwiegley/git-scripts/blob/master/git-rm-conflicts) script written by John Wiegley.

Next I create *'temp'* branch to point to detached HEAD, and then checkout to *'code-review'*. The *'code-review'* branch pointer is then forcibly moved with *git reset --hard temp* to point to the same place as *'temp'*. At the end *'temp'* branch is deleted
so final repository should look like this:

{% highlight text %}


                                 F---G code-review
                                /
               A---B---C---D---E---X develop

{% endhighlight %}


And for the end, here is the script that is very handy for reviewing branches and project history:

{% highlight text %}

git log --graph --decorate --pretty=oneline --abbrev-commit develop code-review
some-other-branch

{% endhighlight %}

You can add *-p* to see detailed diffs, leave off *--pretty=...* to see whole log messages, or tweak it some other way to suit your needs.
