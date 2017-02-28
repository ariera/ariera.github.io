---
layout: post
title: MBP Trackpad, right click (two-finger tap) not working on Mac OS X Lion
comments: true
---
Don't know why but my MacBook Pro (early 2008 ~ MBP 4,1) won't enable the two-finger tap through the trackpad preferences panel o_O?

Solution, from your console:

{% highlight bash %}
defaults -currentHost write -g com.apple.trackpad.enableSecondaryClick -bool YES
{% endhighlight %}

Then log out and log in again.

_src: [https://discussions.apple.com/thread/3189975?start=0&tstart=0](https://discussions.apple.com/thread/3189975?start=0&tstart=0)_

