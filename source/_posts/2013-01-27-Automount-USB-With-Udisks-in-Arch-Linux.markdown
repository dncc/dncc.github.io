---
layout: post
title:  Automount With Udisks in Arch Linux
tags:
- Arch, Udisks, USB, Automount
status: publish
type: post
published: true
---

When it became obvious that *systemd* was going to replace *sysvinit* and
*init scripts* I switched to *systemd* and experienced several issues in the
process. One of those was that the system couldn't automount usb drives
any more.

Before the transition I used *devmon* which was started in *.xinitrc*
before windows manager with *ck-launch-session bash -c "devmon & ..."*
command and it worked seamlessly. However, after the upgrade
the devmon was broken. No matter whether it was issued manually in
terminal or from the .xinitrc script, it reported the following error:

{% highlight text %}

Poll for media failed: Not Authorized
    device: [/dev/sdb1]
    ...
    mountdev /dev/sdb1 DNC vfat
devmon: mount /dev/sdb1 --mount-options noexec,nosuid,noatime    (DNC)
Mount failed: Not Authorized
devmon: error mounting /dev/sdb1 (0)
...
udisks functions are not authorized through policykit,
so devmon cannot automount drives.
...
{% endhighlight %}

I also tried *udiskie*, another automatic disk mounter that uses
*udisks*, and it failed to mount disk with the similar message:

{% highlight text %}

attempting to mount device /org/freedesktop/UDisks/devices/sdb1 (vfat:[])
failed to mount device /org/freedesktop/UDisks/devices/sdb1:
  org.freedesktop.UDisks.Error.PermissionDenied: Not Authorized

{% endhighlight %}

Obviously the problem was not *devmon*, nor *udiskie*. Somehow *udisks* functions
were no longer authorized through *policykit*... It was clear that I should
famialirize myself with *polkit* and find out how to authorize *udisks* actions.

From *polkit(8)* manpages I learned that in order to authorize
disk mounting I should write *polkit* rule. These rules are placed in
*/etc/polkit-1/rules.d/* and */usr/share/polkit-1/rules.d/* directories in
files with *.rules* extension. Both directories are monitored by *polkitd*
process. If rule is changed, added or removed all rules are processed
again.

So I added */etc/polkit-1/rules.d/10-udisks-automount.rules* file with
the following rule:

{% highlight javascript %}

polkit.addRule(function(action, subject) {
        if (action.id == "org.freedesktop.udisks.filesystem-mount" &&
                subject.isInGroup("storage")) {
                return polkit.Result.YES;
        }
});

{% endhighlight %}

Basically it says that *udisks* mount action should be allowed to the users
in the *storage* group. Values for *action.id* as well as other information
about *polkit* authorization policies can be found in */usr/share/polkit-1/actions*
directory. Particularly, the policy for the *udisks* is placed in
*org.freedesktop.udisks.policy* file. All rules are written in JavaScript.

After I'd added this rule, *devmon* and *udiskie* started to work as expected
so I haven't searched further. For that reason I suppose that what I've presented
here is probably quick and dirty solution. It should serve just as a remainder
to me in the future and it might also help someone with the similar problem.
More comprehensive coverage of *udev*, *udisks* and *polkit* can be found on the
[Arch wiki pages for udev](https://wiki.archlinux.org/index.php/Udev#Udisks)
and with *man polkit(8)*.

