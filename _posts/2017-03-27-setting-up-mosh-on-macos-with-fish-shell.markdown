---
layout: post
title:  "Setting up mosh on macOS with fish shell"
date:   2017-03-27 15:34:38 +0200
---

It is actually very easy to install both in macOS and Ubuntu.

On mac you just have to do

{% highlight bash %}
brew install mobile-shell
{% endhighlight %}

and on Ubuntu

{% highlight bash %}
sudo apt-get install mosh
{% endhighlight %}

#### The problem

However, when trying to connect to my server `mosh USER@SERVER` I was getting this error

{% highlight bash %}
The locale requested by LC_CTYPE=UTF-8 isn't available here.
Running `locale-gen UTF-8' may be necessary.

mosh-server needs a UTF-8 native locale to run.

Unfortunately, the local environment ([no charset variables]) specifies
the character set "US-ASCII",

The client-supplied environment (LC_CTYPE=UTF-8) specifies
the character set "US-ASCII".

locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_ALL to default locale: No such file or directory
LANG=
LANGUAGE=
LC_CTYPE=UTF-8
LC_NUMERIC="POSIX"
LC_TIME="POSIX"
LC_COLLATE="POSIX"
LC_MONETARY="POSIX"
LC_MESSAGES="POSIX"
LC_PAPER="POSIX"
LC_NAME="POSIX"
LC_ADDRESS="POSIX"
LC_TELEPHONE="POSIX"
LC_MEASUREMENT="POSIX"
LC_IDENTIFICATION="POSIX"
LC_ALL=
Connection to embodev2.embo.org closed.
/usr/local/bin/mosh: Did not find mosh server startup message. (Have you installed mosh on your server?)
{% endhighlight %}

#### The solution
The solution was as simple as setting your default `LC_ALL` env. Simply add this to your to your `~/.config/fish/config.fish`

<div class="file-title">~/.config/fish/config.fish</div>
{% highlight bash %}
set -x LC_ALL en_GB.UTF-8
{% endhighlight %}

And make sure your server also has that locale availabe:

{% highlight bash %}
sudo locale-gen en_GB.utf-8
{% endhighlight %}
