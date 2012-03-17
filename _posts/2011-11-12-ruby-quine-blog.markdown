---
layout: post
title: Ruby quine (we all came from Turing machine)
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '16929416'
  jabber_published: '1321105874'
  email_notification: '1321105875'
---
{% highlight ruby %}
s="Ruby quine (we all came from Turing machine)%cs=%c%s%c;
printf s,10,34,s,34,10%c";printf s,10,34,s,34,10
{% endhighlight %}
