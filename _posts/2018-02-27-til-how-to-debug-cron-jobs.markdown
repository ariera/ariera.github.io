---
layout: post
title:  "TIL: How to debug cron jobs"
date:   2018-02-27 12:05:28 +0100
---

Today I learned how to properly debug cron jobs, thanks to [https://serverfault.com/a/85906](https://serverfault.com/a/85906).

They key is to recreate the correct environment in which your scripts get executed. I'm going to do this for `root` but you can do this with every user.

First: edit your crontab

{% highlight bash %}
crontab -e

### Add this line:

* * * * *   /usr/bin/env > /root/cron-env
{% endhighlight %}

Once it has had the chance to execute once you can remove it.

Second: let's create a new script called `run-as-cron` that will run your scripts with the correct environment cron is running.


{% highlight bash %}
cd /root
touch run-as-cron
chmod +x run-as-cron
vim run-as-cron

### Add these lines:

#!/bin/bash
/usr/bin/env -i $(cat /root/cron-env) "$@"
{% endhighlight %}

Finally, debug your problematic script like this:

{% highlight bash %}
/root/run-as-cron /the/problematic/script --with arguments --and parameters
{% endhighlight %}
