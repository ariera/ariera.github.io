---
layout: post
title: Configuring Git after formatting
comments: true
---
Simply copy&paste this in your console:

{% highlight bash %}
#Author
git config --global user.name "Alejandro Riera"
git config --global user.email ariera@whatever.com

# Abbreviations
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch

##
# Pimp-out log:
# From: http://www.jukie.net/bart/blog/pimping-out-git-log
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
{% endhighlight %}


_src: [http://superuser.com/questions/169695/what-are-your-favorite-git-aliases](http://superuser.com/questions/169695/what-are-your-favorite-git-aliases)_

