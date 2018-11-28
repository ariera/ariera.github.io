---
layout: post
title:  "TIL: move files from a git repo to another preserving history"
date:   2018-10-26 08:22:39 +0200
---

Today I learned how to move files and folders from one git repository to another one while preserving history, thanks to [Greg Bayer](http://gbayer.com/development/moving-files-from-one-git-repository-to-another-preserving-history/)


{% highlight bash %}
# First we clone the original repo to a temp location
git clone REPO_A_URL repo-A-temp-dir
cd repo-A-temp
git remote rm origin
# Then we filter our subdirecotory
git filter-branch --subdirectory-filter PATH/TO/YOUR/DIR -- --all
git add .
git commit

# Now we clone (or create) the destination repository
cd ..
git clone REPO_B_URL repo-B-dir
cd repo-B-dir
# Add the filtered subdirectory as a remote a pull from it
git remote add repo-A-branch ../repo-A-temp-dir
git pull repo-A-branch master --allow-unrelated-histories
git remote rm repo-A-branch
{% endhighlight %}

Done :)
