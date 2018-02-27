---
layout: post
title:  "TIL: How to debug cron jobs"
date:   2018-02-27 12:05:28 +0100
---

Today I learned how to properly debug cron jobs, thanks to [https://serverfault.com/a/85906](https://serverfault.com/a/85906).

They key is to recreate the correct environment in which your scripts get executed. I'm going to do this for `root` but you can do this with every user.

First: edit your crontab and add one line

```
crontab -e
* * * * *   /usr/bin/env > /root/cron-env
```

Once it has had the chance to execute once you can remove it.

Second: let's create a new script called `run-as-cron` that will run your scripts with the correct environment cron is running.

```
cd /root
touch run-as-cron
chmod +x run-as-cron
vim run-as-cron
#!/bin/bash
/usr/bin/env -i $(cat /root/cron-env) "$@"
```

Finally, debug your problematic script like this:

```
/root/run-as-cron /the/problematic/script --with arguments --and parameters
```
