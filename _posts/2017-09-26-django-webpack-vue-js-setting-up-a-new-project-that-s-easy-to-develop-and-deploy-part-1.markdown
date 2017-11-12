---
layout: post
title:  "Django + webpack + Vue.js - setting up a new project that's easy to develop and deploy (part 1)"
date:   2017-09-26 08:55:25 +0200
---

I recently started a new web project based on both Django and Vuejs. It took a good deal of reading half-related and/or outdated posts, tons of documentation, and a good deal of trial and error to get it right.

In this post we will cover the key points on **setting up a nice and solid development environment that is consisten with production and hence easy to deploy**. The specifics of how to deploy, final configuration touches and (automation) will be discussed in a future article.

This is the solution that I found, there may be better alternatives (which I'd be happy to read) and it is utimately written with the intention of helping others as well as my future-self :)

<h1 class="no-border">
  <img src="{{ site.url }}/assets/django-vue-dev.png" alt="django webpack vue logos">
</h1>
<h2 class="mt-30">The problem</h2>
Both Django and Vue offer local web servers to facilitate development, but that confused me in the beginning. Does that mean that I need to be running both servers in parallel? And do I need to switch from one to another while developing? That sounds cumbersome! If you are building a full SPA with Vue as your frontend, and Django as a simple API backend that might be okey, but I want something else.

I want to remain flexible: **some parts of my project will behave like an SPA, other parts will be rendered by Django, and some other will be a mixture: Django-rendered with Vue components**. I want to be able to use my webpack-compiled css in my Django templates, as well as other static assets.

<h2 class="mt-30">My goals</h2>

1. minimise differences in between prod & dev
1. use just one url in development (ie Django's runserver should proxy to webpack's dev server, or the other way around)
1. preserve hot reload (ie. HMR) in development
1. django backed pages should work without much ceremony

<h2 class="mt-30">My solution</h2>
The solution I found is based around the django library [`django-webpack-loader`](https://github.com/ezhome/django-webpack-loader) and the [`webpack-bundle-tracker`](https://github.com/ezhome/webpack-bundle-tracker) webpack plugin. The django module will read a manifest file, and a webpack plugin take care of updating it. That way, webpack will compile all the assets and Django will be able to use them in your templates.

All you will need to do while developing is launch both local web servers and point your browser to the Django one :)

_Versions i'm using:_
* Vue.js 2.4.2
* Django 1.11
* vue-cli 2.8.2
* Python 3.6.1

<h1 class="no-border">Setting up the dev environment</h1>
<h2 class="mt-30">Creating the project</h2>
I have used the [vue-cli](https://github.com/vuejs/vue-cli) to create a new project based on the [webpack template](https://vuejs-templates.github.io/webpack/), it will do most of the configuration and setup for us. Then, inside the new directory holding my project, I had created a new django project:

{% highlight bash %}
vue init webpack my_project
cd my_project
django-admin startproject my_project .
{% endhighlight %}

Now install both libraries

{% highlight bash %}
yarn add webpack-bundle-tracker --dev
pip install django-webpack-loader
pip freeze > requirements.txt
{% endhighlight %}

Add `django-webpack-loader` to your `INSTALLED_APPS`, we will get to the specific configuration in a minute.

{% highlight python %}
# ./my_project/settings.py
INSTALLED_APPS = [
    'webpack_loader',
]
{% endhighlight %}

And configure webpack to use `BundleTracker`.

{% highlight javascript %}
// build/webpack.base.conf.js
let BundleTracker = require('webpack-bundle-tracker')
module.exports = {
  // ...
  plugins: [
    new BundleTracker({filename: './webpack-stats.json'}),
  ],
}
{% endhighlight %}


## Defining the statics paths

This part was one of the trickiest, there are several configuration options all with very similar names and they feel very confusing to new commers to the framework. I'll spend a little explaining what and why every option does.

My goal here was to have in the end a directory named `./public` (at the root of the project) where all the assest will be stored, with no unnecesary subdirectories (ie. grouping all javascript inside a `js` dir would be okey, but theres no need for deeper levels). The final `./public` folder would look like this

```
- public/
    - js/ (webpack generated)
        - app.c611f8fd9b27da4ec95f.js
        - app.c611f8fd9b27da4ec95f.js.map
        - ...
    - css/ (webpack generated)
        - ...
    - img/ (django generated)
        - ...
    - admin/ (django generated)
        - ...
```

In order for this to happen we have to configure both worlds properly. Let's take a look at the final configuration of both Django and webpack, and then we will discuss them.

{% highlight javascript %}
// ./config/index.js
module.exports = {
  build: {
    assetsRoot: path.resolve(__dirname, '../dist/'),
    assetsSubDirectory: '',
    assetsPublicPath: '/static/',
    // ...
  },
  dev: {
    assetsPublicPath: 'http://localhost:8080/',
    // ...
  }
}

{% endhighlight %}

{% highlight python %}
# ./my_project/settings.py
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'dist'),
    os.path.join(BASE_DIR, 'static'),
)
STATIC_ROOT = os.path.join(BASE_DIR, 'public')
STATIC_URL = '/static/'

WEBPACK_LOADER = {
    'DEFAULT': {
        'BUNDLE_DIR_NAME': '',
        'STATS_FILE': os.path.join(BASE_DIR, 'webpack-stats.json'),
    }
}
{% endhighlight %}

Let's analyse this step by step.

1. **`build.assetsRoot: path.resolve(__dirname, '../dist/')`**

    This is the output directory of our `npm run build` task, that takes care of compiling and building all of our webpack-controlled assets. We will run this task as part of the deployment process. It will take care of analysing all our webpack entry points, traversing and compiling them. The out put will be stored in the `./dist` directory at the root of our project (which should be gitignored).

1. **`STATICFILES_DIRS`** and **`STATIC_ROOT`**

    When deploying to production we will also run the Django script `collectstatic` that takes care of finding all the static files our project makes use of, and putting them in the correct folder. With `STATICFILES_DIRS` we are telling it were to look for these files. With `STATIC_ROOT` we specify where to place them.

    In `STATIC_DIRS` we have specified 2 directories: one is the `./static` (Django's default), and the above defined `./dist` where webpack will place its compiled assets.


1. **`build.assetsSubDirectory: ''`** and **`build.assetsPublicPath: '/static/'`**

    Now we tell webpack that (in production) any asset reference should point to `/static` (ie at the root of the domain). For example, if our code reads:
    ```
    <img src="images/logo.png">
    ```

    the final compiled version will be:

    ```
    <img src="/static/images/logo.png">
    ```

1. **`STATIC_URL = '/static/'`**

    The same thing but for Django. Everytime one of our templates referes to a static file it will render it under the `/static` path.

1. **`dev.assetsPublicPath: 'http://localhost:8080/'`**

    This is one of the key points in which we achieve our goal number 2 (use only one url in dev). We want the webpack dev server to serve our webpack assets. While we point our browser to Django's `localhost:8000`, webpack will compile our code to point at `localhost:8080` (notice those are 2 different ports).

1. **`WEBPACK_LOADER`**

    In `WEBPACK_LOADER['DEFAULT']['STATS_FILE']` we point to the json manifest file generated by webpack's `BundleTracker` (see above in file `build/webpack.base.conf.js`).

    And we set to blank `WEBPACK_LOADER['DEFAULT']['BUNDLE_DIR_NAME']` because we don't want to nest our webpack assets into any subdirectory.




<h3 class="no-border"> Referencing static files from both Vue and Django</h3>

One final touch. I would like to have a place to put images, fonts, or generic statics assets that can be used from both worlds: Django tempaltes and Vue components. In other words, I want to have a `./static` directory at the root of the project where I could put (for example) my logo and be able to write this code:

{% highlight html %}{% raw %}
<!-- in a Django template -->
<img src="{% static 'logo.png' %}">

<!-- in a Vue component -->
<img src="static/logo.png">
{% endraw %}{% endhighlight %}

The Django example already works thanks to the config above described (see `STATICFILES_DIRS`), but for Vue/Webpack to work you'd need to add an alias to the wbepack config, like this.

{% highlight javascript %}
// build/webpack.base.conf.js
module.exports = {
  resolve: {
    alias: {
      // ...
      '__STATIC__': resolve('static'),
    },
}
{% endhighlight %}

Thanks to this we can now write our Vue code like this (mind the `~`):

{% highlight html %}
<img src="~__STATIC__/logo.png">
{% endhighlight %}

## Hot realod or Hot Module Replacement (HMR)

While in development we are only accessing Django's server, and proxying our asset requests to the webpack server (see above `dev.assetsPublicPath`). Because of this HMR or hot reload is broken. We need to configure webpack's hot middleware to point to the right path, and we need to add a new header to the webpack dev server so it allows for CORS requests.

{% highlight javascript %}
// build/dev-client.js
// before
var hotClient = require('webpack-hot-middleware/client?noInfo=true&reload=true')
// after
var hotClient = require('webpack-hot-middleware/client?noInfo=true&reload=true&path=http://localhost:8080/__webpack_hmr')
{% endhighlight %}

{% highlight javascript %}
// build/dev-server.js
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  headers: {
    "Access-Control-Allow-Origin":"\*"
  },
  // ...
})
{% endhighlight %}


## The basic Django template

We will define a Django template reusable by other parts of our application that will take care of including the webpack assets.

{% highlight html %}{% raw %}
<!-- ./templates/my_project/base.html -->
{% load render_bundle from webpack_loader %}
<html>
  <body>
    {% block content %}
    {% endblock %}
    {% render_bundle 'app' %}
  </body>
</html>
{% endraw %}{% endhighlight %}

Now other parts of our project could just extend this template and benefit from having all the styles from css or javascript accessible. In fact, let's create a second template, extending this one, to serve as the root of our SPA:

{% highlight html %}{% raw %}
<!-- ./templates/my_project/spa.html -->
{% extends "my_project/base.html" %}
{% block content %}
  <div id="app"></div>
{% endblock %}
{% endraw %}{% endhighlight %}

Notice that in order for this template to be detected by Django you need to add the following to your `settings.py`

{% highlight python %}
# ./my_project/settings.py
TEMPLATES = [
  {
    'DIRS': [os.path.join(BASE_DIR, 'templates')],
    # ...
  }
]
{% endhighlight %}

And in our `main.js` where we define the root component of our SPA, we will point to this `div#app`:

{% highlight javascript %}
import App from './App'
new Vue({
  el: '#app',
  template: '<App/>',
  components: { App },
})
{% endhighlight %}

The last step would be to define a new url route in our Django settings:
{% highlight python %}
# ./my_project/settings.py
from django.views.generic import TemplateView
urlpatterns = [
    url(r'^$', TemplateView.as_view(template_name='my_project/spa.html'), name='home'),
    # ...
]
{% endhighlight %}

And we can now open `http://localhost:8000` in our browser and enjoy our fully functional dev environment :)

## Clean unnecessary files and options
Before we conclude let's do a little cleanup.

Remove the `index.html` file at the root of your project and stop using `HtmlWebpackPlugin` in development. Since Django will render the initial html and include any assets there we don't need `HtmlWebpackPlugin` to generate an initial html file for us.

{% highlight javascript %}
// build/webpack.dev.conf.js
// remove these lines:
new HtmlWebpackPlugin({
  filename: 'index.html',
  template: 'index.html',
  inject: true
}),
{% endhighlight %}

For the same reason we want to remove it from the production config, but we want to keep it in tests. Simply move this bit of conde from `build/webpack.prod.conf.js` to `build/webpack.test.conf.js`

{% highlight javascript %}
new HtmlWebpackPlugin({
  filename: process.env.NODE_ENV === 'testing'
    ? 'index.html'
    : config.build.index,
  template: 'index.html',
  inject: true,
  minify: {
    removeComments: true,
    collapseWhitespace: true,
    removeAttributeQuotes: true
    // more options:
    // https://github.com/kangax/html-minifier#options-quick-reference
  },
  // necessary to consistently work with multiple chunks via CommonsChunkPlugin
  chunksSortMode: 'dependency'
}),
{% endhighlight %}

And last but not least lets add the following to your `.gitignore` file:

{% highlight git %}
webpack-stats.json
public/
dist/
{% endhighlight %}

# Conclusions:

To have a good dev environment we have used Django as our main dev server, which will in turn proxy any asset request to webpack. This required a good deal of configuration. In doing so we have simplified the development workflow so we can spend more time programming and less time switching between browsers or fine tuning the config for new cases that may appear.

We have also paved the path to production deployment, but that's something that we will cover in the next post.

Until then,

**Happy hacking!**

#### References used

* [https://stackoverflow.com/questions/41342144/webpack-hmr-webpack-hmr-404-not-found](https://stackoverflow.com/questions/41342144/webpack-hmr-webpack-hmr-404-not-found)
* [https://github.com/webpack/webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware)
* [https://gist.github.com/Belgabor/130e7770575e74581b67597fcb61717e](https://gist.github.com/Belgabor/130e7770575e74581b67597fcb61717e)
* [https://github.com/ezhome/django-webpack-loader/tree/master/examples](https://github.com/ezhome/django-webpack-loader/tree/master/examples)
* [https://github.com/rokups/hello-vue-django](https://github.com/rokups/hello-vue-django)
* [https://gist.github.com/genomics-geek/81c6880ca862d99574c6f84dec81acb0](https://gist.github.com/genomics-geek/81c6880ca862d99574c6f84dec81acb0)
* [https://cscheng.info/2016/08/03/integrating-webpack-dev-server-with-django.html](https://cscheng.info/2016/08/03/integrating-webpack-dev-server-with-django.html)
* [http://vuejs-templates.github.io/webpack/backend.html](http://vuejs-templates.github.io/webpack/backend.html)
* [http://owaislone.org/blog/webpack-plus-reactjs-and-django/](http://owaislone.org/blog/webpack-plus-reactjs-and-django/)


<div style="display:none">
  # Getting ready for production
  https://docs.djangoproject.com/en/1.11/ref/templates/api/#writing-your-own-context-processors
  https://git.embl.de/mainar/my.embo.org/compare/vue-bulma-bootstraping...django-with-vue?view=parallel
</div>

