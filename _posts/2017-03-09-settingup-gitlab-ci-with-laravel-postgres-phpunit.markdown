---
layout: post
title:  "Setting up GitLab's Continuous Integration with Laravel, PostgreSQL and PHPUnit"
date:   2017-03-9 18:06:56 +0100
---

I recently set up a continuous integration process based on GitLab for a project based on Laravel and PostgreSQL. It took a while to get it right, so I am sharing here my final config.

## Basic structure
The main GitLab CI configuration is held by a file at the root of your project that should be called `.gitlab-ci.yml`. From this file it is possible to call other shell scripts, but I have tried to hold up everything in just one place.

We will however manage a second file called `.env.gitlab-ci` that will hold all ENV variable configuration optios specific to GitLab CI.

Last but not least you will need a PHPUnit xml configuration file.

## Step by step look into `.gitlab-ci.yml`
We'll go line by line in the config file, but you can see the final result at the end of the article.


&nbsp;
{% highlight yaml %}
image: php:7.0
{% endhighlight %}

Here we select one of the available docker images with php already configured. You can find a complete list here: from https://hub.docker.com/r/_/php/


&nbsp;
{% highlight yaml %}
services:
  - postgres:latest
{% endhighlight %}

From GitLab's official documentation [src](https://docs.gitlab.com/ce/ci/docker/using_docker_images.html#what-is-a-service):
>The `services` keyword defines just another docker image that is run during your job and is linked to the docker image that the `image` keyword defines. This allows you to access the service image during build time.

It is **very important** to remark that if you add `postgres` as service to your application, the service container for PostgresSQL will be accessible under the hostname `postgres`. So, in order to access your database service you have to connect to the host named `postgres` instead of a socket or `localhost`. This will be important later when we configure our `.env.gitlab-ci` file.


&nbsp;
{% highlight yaml %}
variables:
  POSTGRES_DB: mydb-test
  POSTGRES_USER: runner
  POSTGRES_PASSWORD: ""
{% endhighlight %}
Here you may use all the values you want as long as they match with those of `.env.gitlab-ci` and your `phpunit.xml`.


&nbsp;
{% highlight yaml %}
before_script:
{% endhighlight %}
Again from the official documentation [src](https://docs.gitlab.com/ce/ci/yaml/README.html#before_script)
>`before_script` is used to define the command that should be run before all jobs, including deploy jobs, but after the restoration of artifacts. This can be an array or a multi-line string.

Lets explore all these commands one by one.


&nbsp;
{% highlight yaml %}
- >
  set -xe
  && apt-get update -yqq
  && apt-get install -yqq
  git
  libicu-dev
  libpq-dev
  libzip-dev
  zlib1g-dev
{% endhighlight %}

Simply installing some dependencies
* `git` is needed to later install `composer`
* `libicu-dev` will be necessary for the internationalization php extension (`intl`)
* `libpq-dev` are the header files for postgres
* `libzip-dev` and `zlib1g-dev` are necessary for zip manipulation and needed to activate the corresponding php extension. Without them our `composer install` command will throw a lot of warnings and try to download each package twice **slowing the whol process down for 2 or more minutes**


&nbsp;
{% highlight yaml %}
- >
  docker-php-ext-install
  pdo_pgsql
  pgsql
  sockets
  intl
  zip
{% endhighlight %}
`docker-php-ext-install` is a handy script for installing php extensions provided by the official php docker image. I restricted myself to the bare minimun of extension necessary for my project to run. You may need more.


&nbsp;
{% highlight yaml %}
- curl -sS https://getcomposer.org/installer | php
- php composer.phar self-update
- php composer.phar install --no-progress --no-interaction
{% endhighlight %}
Install `composer` and use it to install all of our project dependencies.


&nbsp;
{% highlight yaml %}
- cp .env.gitlab-ci .env
{% endhighlight %}
Set up the correct ENV variables. We'll look into this file in a moment.


&nbsp;
{% highlight yaml %}
- php artisan key:generate
- php artisan help config:clear
- php artisan route:clear
- php artisan migrate:refresh
{% endhighlight %}
Set application key, clear a couple of caches just in case and run the migrations.



&nbsp;
{% highlight yaml %}
test:app:
  script:
  - vendor/bin/phpunit --configuration phpunit-gitlabci.xml
{% endhighlight %}
And finally run our PHPUnit test suite using our `phpunit-gitlabci.xml` config file.

## Small break
That was the most complicated file, the rest is a piece of cake, but you deserve a rest :P
<iframe src="//giphy.com/embed/a3ANjL4bRwsO4" width="480" height="274" frameBorder="0" class="giphy-embed" allowFullScreen></iframe>

&nbsp;
## A brief look into `.env.gitlab-ci`
The file itself is quite self explainatory, and you can see the final version at the end of the article. But I wanted to highlight the database configuration.
{% highlight yaml %}
DB_CONNECTION=pgsql
DB_PORT=5432
DB_HOST=postgres
DB_DATABASE=mydb-test
DB_USERNAME=runner
DB_PASSWORD=
{% endhighlight %}
Notice how the values for `DB_HOST`, `DB_DATABASE` and `DB_USERNAME` are the same as those of inside of the `variables` key on the `.gitlab-ci.yml` configuration file.


&nbsp;
## A brief note on `phpunit-gitlabci.xml`
This file may be optional depending on your local configuration. If your normal `phpunit.xml` file doesn't define any `DB_DATABASE` env variable you would not need this. But if it happens to define one, and has a different value from that that you chose for GitLab's CI you have to remember to set it correctly:

{% highlight yaml %}
<env name="DB_DATABASE" value="mydb-test"/>
{% endhighlight %}


&nbsp;
## Summary
GitLab's continuous integration system can be a blessing once is properly configured, but it can get a bit confusing if you trail off of the usual path and all you hear is *docker image configuration linking* and you've never worked with it before. Hopefully you'll find this small guide useful
&nbsp;
#### .gitlab-ci.yml
{% highlight yaml %}
image: php:7.0
services:
  - postgres:latest
variables:
  POSTGRES_DB: mydb-test
  POSTGRES_USER: runner
  POSTGRES_PASSWORD: ""

before_script:
- >
  set -xe
  && apt-get update -yqq
  && apt-get install -yqq
  git
  libicu-dev
  libpq-dev
  libzip-dev
  zlib1g-dev
- >
  docker-php-ext-install
  pdo_pgsql
  pgsql
  sockets
  intl
  zip
- curl -sS https://getcomposer.org/installer | php
- php composer.phar self-update
- php composer.phar install --no-progress --no-interaction
- cp .env.gitlab-ci .env
- php artisan key:generate
- php artisan help config:clear
- php artisan route:clear
- php artisan migrate:refresh

test:app:
  script:
  - vendor/bin/phpunit --configuration phpunit-gitlabci.xml
{% endhighlight %}

&nbsp;
#### .env.gitlab-ci
{% highlight sh%}
APP_ENV=testing
APP_KEY=key
APP_DEBUG=true
APP_LOG_LEVEL=debug
APP_URL=http://localhost:8000

DB_CONNECTION=pgsql
DB_HOST=postgres
DB_PORT=5432
DB_DATABASE=mydb-test
DB_USERNAME=runner
DB_PASSWORD=

BROADCAST_DRIVER=log
CACHE_DRIVER=array
SESSION_DRIVER=array
QUEUE_DRIVER=sync
MAIL_DRIVER=log
{% endhighlight %}
