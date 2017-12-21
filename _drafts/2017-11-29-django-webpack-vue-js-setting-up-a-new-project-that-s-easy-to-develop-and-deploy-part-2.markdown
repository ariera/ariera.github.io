---
layout: post
title:  "Django + webpack + Vue.js - setting up a new project that's easy to develop and deploy (part 2)"
date:   2017-11-29 08:41:15 +0100
---

In this second part we will look into:

* configuring the project to run smoothly on production mode
* the basics of setting up a production server
* how to deploy and automating it

# Running the project on production mode:

## run `npm run build`

This task will compile all of our assets and place them in `./dist/` as per specified in `./config/index.js` (see part1).

{% highlight javascript %}
// ./config/index.js
build: {
  // ...
  assetsRoot: path.resolve(__dirname, '../dist/'),
  assetsSubDirectory: '',
  assetsPublicPath: '/static/',
}
{% endhighlight %}

This will also create a new file at the root of your project `./webpack-stats.json` specifying all you webpack chunks, in our case these are `vendor`, `app` and `manifest`:

{% highlight json %}
{
  "status": "done",
  "publicPath": "/static/",
  "chunks": {
    "vendor": [],
    "app": [],
    "manifest": []
  }
}
{% endhighlight %}


## run `python manage.py collectstatic --noinput`

This Django task will look for all the assets stored in `./static/` and `./dist/` and place them in `./public/`. This is achieved in the `./project/settings.py`:

{% highlight python %}
# ./project/settings.py
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'dist'),
    os.path.join(BASE_DIR, 'static'),
)
STATIC_ROOT = os.path.join(BASE_DIR, 'public')
{% endhighlight %}



# Some notes:

from this email with Charles Aracil

    Issue was related on the fact that, in production mode, django does not serve static files (even with the 'runserver'). A webserver, like Apache or Gunicorn has to do it. Didn't know that.

    Anyway, to bypass that for testing in a development-like mode, we can call the runserver with an "--insecure" option.

    > python manage.py runserver --insecure

    That way I can assert built files are okay without requiring a real production rollout. Although I will test it anyway.

    So, have to wrap it up a little, but the repo should be fine soon :)

    Still got some issue with e2e test on Linux, probably due to a Selenium misconfiguration.

