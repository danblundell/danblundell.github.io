---
layout: post
title: "Solving Amazon EC2 Memory Issues"
localgov: false
tag:
date: "2014-04-30"
meta: "A way to work around Amazon EC2 linux, apache and mysql crashing due to insufficient memory"
---

Often with the Amazon micro instances, and other instances with low memory allocations, your web server will 'crash', especially where MySQL's involved. After a few page requests the MySQL engine can lock up because it doesn't have enough memory.

Amazons micro instances don't come with any swap memory by default so if you experience issues and don't want the costs of bumping your server to the next instance size, adding some swap space can improve performance of the micro instances no end:

{% highlight bash %}
sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
sudo /sbin/mkswap /var/swapfile
sudo /sbin/swapon /var/swapfile
{% endhighlight %}

Once again, swap won't default to on after a system reboot. To automatically enable swap after a reboot, you'll need to update /etc/fstab, open the file with the following:

{% highlight bash %}
sudo vi /etc/fstab
{% endhighlight %}

The file will open in an editor, paste the following line at the end of the file

{% highlight bash %}
/var/swapfile swap swap defaults 0 0
{% endhighlight %}

Then type

{% highlight bash %}
:wq
{% endhighlight %}

Then hit enter. This says 'write and quit' which is essentially save and exit the editor.

After each reboot your server will now start Apache, MySQL and swap memory.

Depending on what you're running you may still have memory issues but keep in mind that the Amazon micro instances only have just over half a gig of memory so it may be that your application is just too resource heavy to run on the free tier.