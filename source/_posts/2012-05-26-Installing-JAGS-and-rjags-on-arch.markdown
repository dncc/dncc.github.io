---
layout: post
title: Installing JAGS and rjags on Arch Linux
tags:
- Linux
status: publish
type: post
published: true
---

I have been evaluating available options for developing Bayesian and state-space models in R recently so I stumbled upon JAGS and rjags. Since I had trouble to install them on Arch Linux I thought it might be good to create a brief record of the whole process. the thread that was useful to me to identify and solve the problem is [here](http://sourceforge.net/projects/mcmc-jags/forums/forum/610037/topic/4996525)
.

JAGS installation went well. There was no standard Arch package for JAGS so I built it from the source, which was straightforward:

{% highlight text %}
./configure
make
sudo make install
{% endhighlight %}

I found this [post](http://quote.ucsd.edu/blogs/rogblog/2008/11/09/installing-rjags-on-64-bit-debian-etch/) which describes that you should specify *'--with-jags-modules' and '--libdir'* options at configuration stage if you are on 64-bit architecture, but it seems this is no more needed (All jags libs were placed in '/usr/local/lib' directory, not in 'lib64' as it was the case at the time the post was written).

Next, I tried to install 'rjags' package within R:

{% highlight text %}
install.packages('rjags')
{% endhighlight %}

and I got this error message:

{% highlight text %}
...
*** installing help indices
** building package indices ...
Error : .onLoad failed in loadNamespace() for 'rjags', details:
call: dyn.load(file, DLLpath = DLLpath, ...)
error: unable to load shared object
'/usr/local/lib/R/site-library/rjags/libs/rjags.so':
libjags.so.3: cannot open shared object file:
No such file or directory
ERROR: installing package indices failed *
removing '/usr/local/lib/R/site-library/rjags'
...
{% endhighlight %}

First, I tried various things with the previous R function (some of which, for instance, were to include 'dependecies=TRUE' and jags module paths as arguments to this function). That didn't work. After quite a few failed trials I found out that jags libraries were in '/usr/local/lib' directory and it was not included in my linker path. I checked this with:

{% highlight bash %}
sudo ldconfig -p | grep local
{% endhighlight %}

and since '/usr/local/lib' didn't appear on *stdout* I added this line in the new '/etc/ld.so.conf.d/jags.conf' file. Than, in order to activate linker paths and check if '/usr/local/lib' was included I run:

{% highlight bash %}
sudo ldconfig -v | grep /usr/local/lib
{% endhighlight %}

I tried 'rjags' installation again with:

{% highlight text %}
sudo R --with-jags-modules=/usr/local/lib/JAGS/modules-3 \
CMD INSTALL ~/downloads/rjags_3-2.tar.gz
{% endhighlight %}

and this time it was installed!

It remains just to develop some Bayesian models for predicting a stock price movements or whatever, and make myself awfully rich ;).





