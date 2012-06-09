---
layout: post
title: Rotate Log Files in Rails 3.2
tags:
- Rails 3
status: publish
type: post
published: true
---

The easiest way to rotate Rails 3.2 log files (and they can get
really huge) is to put the following line in the 'config/environment/developers.rb' file:

{% highlight ruby %}
AppName::Application.configure do
  ...
  # auto rotate 2 log files with 5MB each
  config.logger = Logger.new(config.paths["log"].first, 2, 5.megabytes)
  ...
end
{% endhighlight %}

*Note: in the earlier versions you could access log file name with 
"config.paths.log.first" call. Not the case any more. 
Now you have to use hash notation as in the snippet above.*

