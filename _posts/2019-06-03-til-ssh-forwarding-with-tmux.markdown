---
layout: post
title:  "TIL: ssh forwarding with tmux"
date:   2019-06-03 17:10:22 +0200
---


Here is the simple recipie that I have to search for time and again. Check the sources linked below for a much better explanation of how and why.

### In my computer

Make your key available to `ssh-agent`

{% highlight bash %}
# for mac
ssh-add -K ~/.ssh/id_rsa
{% endhighlight %}


and add the following to `~/.ssh/config`

{% highlight bash %}
    # ~/.ssh/config
    Host *
        SendEnv LANG LC_*
        ForwardAgent no
{% endhighlight %}


### Configure `tmux` in the remote server:


{% highlight bash %}
    # ~/.tmux.conf
    set-environment -g 'SSH_AUTH_SOCK' ~/.ssh/ssh_auth_sock
{% endhighlight %}

{% highlight bash %}
    # ~/.ssh/rc
    if [ -S "$SSH_AUTH_SOCK" ]; then
        ln -sf $SSH_AUTH_SOCK ~/.ssh/ssh_auth_sock
    fi
{% endhighlight %}

### Debbuging: is ssh forwarding working?


{% highlight bash %}
    echo "$SSH_AUTH_SOCK"
    # Should print something like this:
    # /tmp/ssh-4hNGMk8AZX/agent.79453
{% endhighlight %}



### Sources

* [Happy ssh agent forwarding for tmux/screen @ werat.github.io](https://werat.github.io/2017/02/04/tmux-ssh-agent-forwarding.html)
* [Using SSH agent forwarding @ developer.github.com](https://developer.github.com/v3/guides/using-ssh-agent-forwarding/)