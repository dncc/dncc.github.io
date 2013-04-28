---
layout: post
title: Parsing Gem Versions With Awk
tags:
- Unix Awk
status: publish
type: post
published: true
---
I installed latest Ruby (2.0.0-p0) and along with it I wanted to install the last
gem versions from the previous Ruby (2.0.0-rc1). Here is awk one-liner that helped
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
gems. Then, I piped list of gems to the awk command in order to parse it. Here is what it does:

* Remove parenthesis that enclose gem versions in each line with _gsub(/\\(|\\)/, "")_.
* Concatenate gem version with its name in a single line in the form that the
   _gem install_ command understands (i.e -v=\<gem-version\> \<gem-name\>) with
   _line=line " -v="$NF" " $1_. We take last field (gem version number)
   with _$NF_ and than name of the gem with _$1_ and concatenate that to line
   variable.
* Print the concatenated line in the 'gems-versions' file at the end.

After I created the list of gems I switched to ruby-2.0.0-p0 and installed gems with:       _gem install \`cat gems-versions\`_.
