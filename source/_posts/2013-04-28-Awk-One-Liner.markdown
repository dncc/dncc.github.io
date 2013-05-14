---
layout: post
title: Parsing Gem Versions With Awk
tags:
- Unix Awk
status: publish
type: post
published: true
---
I installed latest Ruby (2.0.0-p0) today and along with it the smallest gem
versions from the previous Ruby (2.0.0-rc1). Here is awk one-liner that helped
me do that.

{% highlight bash %}

chruby 2.0.0-rc1

gem list | \
    awk '{ gsub(/\(|\)/, ""); line = line " -v=" $NF " " $1; } \
    END  {print line; }' > gems-versions

chruby ruby-2.0.0-p0
gem install `cat gems-versions`

{% endhighlight %}

First, I switched to ruby-2.0.0-rc1 with 'chruby' in order to make a list of available
gems. Then, in order to parse the list of gems I piped it to the awk command. Here is
what it does:

* Removes parenthesis that enclose gem versions in each line with _gsub(/\\(|\\)/, "")_.
* Concatenates gem version with its name in a single line in the form that the
   _gem install_ command understands (i.e -v=\<gem-version\> \<gem-name\>) with
   _line=line " -v="$NF" " $1_. _$NF_ and _$1_ denote the gem's version number
   and name respectively.
* Prints the concatenated line in the _'gems-versions'_ file at the end.

After I created the list of gems I switched to ruby-2.0.0-p0 and installed gems with:
_gem install \`cat gems-versions\`_.

It is easy to adjust this to the similar use cases. If we want, for instance,
to install all gem versions that are already present in the previous Ruby
version the awk that will parse all gem versions will look something like this:

{% highlight bash %}

chruby ruby-2.0.0-p0

gem list | \
    awk '{ gsub(/\)/,""); gsub(/\(|,\s/, " " $1":"); \
    $1=""; line = line " " $0} END { print line }' > gems-versions

chruby ruby-2.0.0-p0
gem install `cat gems-versions`

{% endhighlight %}

Everything is almost the same as in the previous case except there are
a couple of extra awk commands to add.

* _gsub(/\)/,""); gsub(/\(|,\s/, " " $1":")_ replaces string
  _xgem (2.0, 1.0)_ with _xgem xgem:2.0 xgem:1.0_
* _$1=""_ sets the first field (a gem name) to the empty string so the
  output string becomes _gem_x:1.0 gem_x:2.0_ which should be valid
  input for the _gem install_ command.
* _line = line " " $0_ concatenates the output string _$0_ to the _line_
  variable which will be saved to the _gems-versions_ file.
