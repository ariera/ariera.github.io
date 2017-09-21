---
layout: post
title:  "dev-toolbar a web developer's best friend!"
date:   2017-09-20 18:05:28 +0200
tags: django, python, webdev
---

The **dev-toolbar** is a simple concept we came up with many years ago. The idea is to provide developers with a toolbelt full of tricks (shortcuts, toggles, login-as...) hidden in the frontend. It is such an integral part of my workflow that it is usually the first thing I set up in any project.

In this entry we will see how to set it up in a Django web application. We will focus on **the feature that everybody loves**: the ability to login as any user in the database (great time saver). We call it **impersonate** , and this is how it looks like:

![developer toolbar]({{ site.url }}/assets/dev-toolbar.png)

A simple select with all the users in your database. Click on one and you'll be logged in as him/her.

Let's get on with it!

### 1. Create a dev app
We will start by creating a new app that we will call `dev`. It will hold all of our impersonation logic, but is also a useful place to have to keep a style guide for developers, or some handy snippets to share accross the team, or maybe just to house a temporary experiment.

{% highlight bash %}
python manage.py startapp dev
{% endhighlight %}

and don't forget to add it to your `INSTALLED_APPS`


<div class="file-title">./your_app/settings.py</div>
{% highlight python %}
# ./your_app/settings.py
INSTALLED_APPS = [
  'dev.apps.DevConfig',
  # ...
]
{% endhighlight %}

### 2. The view and urls
Here we will control if the current user is allowed to impersonate or not, and perform the actual impersonation. In this case we are allowing any superuser to login as any other user, but you may choose any other criteria, for example only in development environment, or only if `DEBUG` is `True`.

<div class="file-title">./dev/views.py</div>
{% highlight python %}
# ./dev/views.py
from django.shortcuts import redirect, render, get_object_or_404
from django.contrib.auth import get_user_model, login
from django.http import Http404

def impersonate(request, user_id):
  current_user = request.user
  if current_user.is_superuser:
    user = get_object_or_404(get_user_model(), pk=user_id)
    login(request, user)
    return redirect('/dev')
  else:
    raise Http404("Impersonate is only available for superusers")
{% endhighlight %}

<div class="file-title">./dev/urls.py</div>
{% highlight python %}
# ./dev/urls.py
from django.conf.urls import url
from . import views

app_name = 'dev'
urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^impersonate/(?P<user_id>[0-9]+)/$', views.impersonate, name='impersonate'),
]
{% endhighlight %}

<div class="file-title">./your_app/urls.py</div>
{% highlight python %}
# ./your_app/urls.py
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^dev/', include('dev.urls')),
    url(r'^admin/', admin.site.urls),
]
{% endhighlight %}

### 3. Template tag
We want to reuse our toolbar across many templates so we will extract it into it's own template tag, using [Django's inclusion-tags](https://docs.djangoproject.com/en/1.11/howto/custom-template-tags/#inclusion-tags) functionality.

For that you need to create this directory: `./dev/templatetags` and an empty `__init__.py` file inside of it.

Inside our templatetags folder we will create the following file, which acts like the view of our new `dev_toolbar` templatetag. Notice 2 things:

1. First we are linking it to the template `dev/toolbar.html` that we will define later.
2. And second we use `takes_context=True` to be able to access to the current user. This will allow us to do some basic authorization (ie. only superusers are allowed to use this feature). This is redundant with the logic we defined before, but it's good to have a safety net when you're handling delicate data.

<div class="file-title">./dev/templatetags/dev_toolbar.py</div>
{% highlight python %}
# ./dev/templatetags/dev_toolbar.py
from django.contrib.auth import get_user_model
from django import template

register = template.Library()

@register.inclusion_tag('dev/toolbar.html', takes_context=True)
def dev_toolbar(context):
  request = context['request']
  current_user = request.user
  if current_user.is_superuser:
    users = get_user_model().objects.all()
  else:
    users = []
  return {'users': users, 'current_user': current_user}
{% endhighlight %}

The template file contains its own css, javascript and html. Some simple positioning and styling and bit of css+js logic to make the bar more subtle.

We will render a select box with all the users in the database and a javascript will take care of calling the impersonate action whenever we choose another user other than ourselves. Two more handy links to `/dev` and `/admin` areas are included.

Last, but not least, I usually include a small close button, it comes in handy sometimes when doing CSS work, you just want to get rid of the bar.
<div class="file-title">./dev/templates/dev/toolbar.html</div>
{% highlight html %}
{% raw %}
<!-- ./dev/templates/dev/toolbar.html -->
{% if current_user.is_superuser %}
<style>
  #dev-toolbar{
    background-color: #ccc;
    position: fixed;
    bottom: 0;
    right: 0;
    padding: 5px;
  }
  #dev-toolbar.subtle {
    opacity: 0.2;
  }
</style>
<script>
  window.dev = {
    closeToolbar: function closeToolbar(){
      var toolbar = document.getElementById("dev-toolbar")
      toolbar.parentElement.removeChild(toolbar)
    },
    showToolbar: function showToolbar(){
      var toolbar = document.getElementById("dev-toolbar")
      toolbar.classList.remove("subtle");
    },
    impersonate: function impersonate() {
      var select = document.getElementById("dev-impersonate")
      window.location = select.value
    }
  }
</script>
<div id="dev-toolbar" class="subtle" onmouseover="dev.showToolbar()">
  <small><a href="/dev">dev</a></small>
  <small><a href="/admin">admin</a></small>

  {% if users %}
    <select id="dev-impersonate" onchange="dev.impersonate()">
      {% for user in users %}
         <option value="{% url 'dev:impersonate' user.id %}"
             {% if user == current_user %}selected="selected"{% endif %}>
             {{user.email}}
         </option>
      {% endfor %}
    </select>
  {% endif %}

  <button id="dev-close-toolbar" onclick="dev.closeToolbar()">X</button>
</div>
{% endif %}
{% endraw %}
{% endhighlight %}


### 4. Using it

Our feature is complete and ready to be used. Simply load it in any view and call it.

{% raw %}
    {% load dev_toolbar %}
    {% dev_toolbar %}
{% endraw %}

### 5. Show it in the admin panel
We are going to override the default `admin/base.html` template and include our new `dev_toolbar` snippet. For that we need to create a generic `./templates` directory at the root of the project and add it to `TEMPLATES['DIRS']` in the settings of the project.

<div class="file-title">./your_app/settings.py</div>
{% highlight python %}
# ./your_app/settings.py
TEMPLATES = [
  {
    'DIRS': [os.path.join(BASE_DIR, 'templates')],
    # ...
  }
]
{% endhighlight %}

And now create a new file to override the default administration base tempalte.

<div class="file-title">./templates/admin/base.html</div>
{% highlight html %}
{% raw %}
    <!-- ./templates/admin/base.html -->
    {% extends "admin/base.html" %}
    {% load dev_toolbar %}

    {% block footer %}
        {{block.super}}
        {% dev_toolbar %}
    {% endblock %}
{% endraw %}
{% endhighlight %}

#### References

* [Django Http404](https://docs.djangoproject.com/en/1.11/topics/http/views/#the-http404-exception)
* [Django inclusion-tags](https://docs.djangoproject.com/en/1.11/howto/custom-template-tags/#inclusion-tags)
* [SO: Handling request in django inclusion template tag](https://stackoverflow.com/questions/6713022/handling-request-in-django-inclusion-template-tag)
* [SO: How to override and extend basic Django admin templates?](https://stackoverflow.com/a/6586068)
