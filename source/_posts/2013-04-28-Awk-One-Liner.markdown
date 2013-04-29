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

* Removes parenthesis that enclose gem versions in each line with __gsub(/\\(|\\)/, "")__.
* Concatenates gem version with its name in a single line in the form that the
   __gem install__ command understands (i.e -v=\<gem-version\> \<gem-name\>) with
   __line=line " -v="$NF" " $1__. We take last field (gem version number)
   with __$NF__ and than name of the gem with __$1__ and concatenate that to line
   variable.
* Prints the concatenated line in the __'gems-versions'__ file at the end.

After I created the list of gems I switched to ruby-2.0.0-p0 and installed gems with:       __gem install \`cat gems-versions\`__.

It is easy to adjust this to the similar use cases. If we need, for instance,
to install all gem versions that are already present in the previous Ruby
version the awk command should look something like this:

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

* __gsub(/\)/,""); gsub(/\(|,\s/, " " $1":")__ replaces string
  __xgem (2.0, 1.0)__ with __xgem xgem:2.0 xgem:1.0__
* __$1=""__ sets the first field (a gem name) to the empty string so the
  output string becomes __gem_x:1.0 gem_x:2.0__ which should be valid
  input for the __gem install__ command.
* __line = line " " $0__ concatenates the output string __$0__ to the __line__
  variable which will be saved to the __gems-versions__ file.


