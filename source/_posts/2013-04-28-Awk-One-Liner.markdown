---
layout: post
title: Parsing Gem Versions With Awk
tags:
- Unix Awk
status: publish
type: post
published: true
---
I installed today latest Ruby (2.0.0-p0) and along with it the smallest gem
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
gems. Then, I piped list of gems to the awk command in order to parse it. Here is what it does:

* Remove parenthesis that enclose gem versions in each line with _gsub(/\\(|\\)/, "")_.
* Concatenate gem version with its name in a single line in the form that the
   _gem install_ command understands (i.e -v=\<gem-version\> \<gem-name\>) with
   _line=line " -v="$NF" " $1_. We take last field (gem version number)
   with _$NF_ and than name of the gem with _$1_ and concatenate that to line
   variable.
* Print the concatenated line in the 'gems-versions' file at the end.

After I created the list of gems I switched to ruby-2.0.0-p0 and installed gems with:       _gem install \`cat gems-versions\`_.

Variations of this are easy. In case that, for instance, we need to install all gem
versions that are already present in the previous Ruby version then awk command
should look probably something like this:

{% highlight bash %}

chruby ruby-2.0.0-p0

gem list | \
    awk '{ gsub(/\)/,""); gsub(/\(|,\s/, " " $1":"); \
    $1=""; line = line " " $0} END { print line }' > gems-versions

chruby ruby-2.0.0-p0
gem install `cat gems-versions`

{% endhighlight %}

Everything stays the same as in the previous case except we have few additional
awk commands to type:

* With _gsub(/\)/,""); gsub(/\(|,\s/, " " $1":")_ we are replacing string
  _"gem_x (1.0, 2.0)"_ with "gem_x gem_x:1.0 gem_x:2.0"
* Then we are setting the first field (a gem name) to the empty string with
  _$1=""_ so we are left with _"gem_x:1.0 gem_x:2.0"_ which should be valid
  input for the _gem install_ command.
* And last, we concatenate the remaining string _$0_ to the _line_ variable
  which will be saved to the _gems-versions_ file.
