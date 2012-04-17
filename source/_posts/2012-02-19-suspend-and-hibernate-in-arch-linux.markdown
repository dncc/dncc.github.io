---
layout: post
title: Suspend and Hibernate in Arch Linux
tags:
- Linux
status: publish
type: post
published: true
meta:
  _edit_last: '16929416'
  jabber_published: '1329682794'
  email_notification: '1329682794'
---
In order to suspend your session in Arch Linux (to RAM or disk) you should try to follow the steps from bellow:

First install pm-utils tools with the pacman. No surprises here, just type:
{% highlight text %}
sudo pacman -S pm-utils
{% endhighlight %}
and heavy lifting will be done with the awesome pacman program. In the case that 'vbetool' and 'acpi' packages are not installed, you will need to install them too with similar incantations.

Suspend to RAM should be enabled imidiatelly. You can check that with the following command:
{% highlight text %}
sudo pm-suspend
{% endhighlight %}
After this, current session should be saved to RAM and machine put on standby mode.

Hibernation (suspend to disk) needs additional configuration. Receipt that worked for me is listed bellow:

Add to GRUB's config file 'resume' option ('resume=/path/to/swap/drive') by editing the kernel line in the next paragraph:

{% highlight text %}
title Arch Linux
root (hd0,2)
kernel /vmlinuz-linux root=/dev/sda4 ro
initrd /initramfs-linux.img
{% endhighlight %}

as follows:

{% highlight text %}
kernel /vmlinuz-linux root=/dev/sda4 resume=/dev/sda5 ro
{% endhighlight %}

*Note:
GRUB config file on my machine is '/boot/grub/menu.lst', but if you have GRUB2 it should be '/boot/grub/grub.cfg', also the configuration is somewhat different, look for details [here](https://wiki.archlinux.org/index.php/Pm-utils#Hibernation_.28suspend2disk.29) .
The swap partition on my machine is '/dev/sda5', of course you should change this with the path of your swap partition.*

Additionally, resume hook should be added to 'etc/mkinitcpio.conf' as follows:

{% highlight text %}
HOOKS="base udev autodetect ide scsi sata lvm2 resume filesystems"
{% endhighlight %}

Ordering matters here, so it is important to place resume after ide, scsi, sata and lvm2 and before filesystems. After this, recreate ramdisk environment with:

{% highlight text %}
mkinitcpio -p linux
{% endhighlight %}

If everything went well set user permissions for pm-utils in '/etc/sudoers' config file as follows:

{% highlight text %}
username,hostname=NOPASSWD:/usr/sbin/pm-hibernate, /usr/sbin/pm-suspend
{% endhighlight %}

where username and hostname placeholders shoud be actual values. This way you authorize your user to execute pm-hibernate and pm-suspend w/o need to login as superuser each time.

Add aliases to config file of your preferd shell and you are good to go:

{% highlight text %}
alias hibernate="sudo pm-hibernate"
alias suspend="sudo pm-suspend"
{% endhighlight %}

Happy hacking!
