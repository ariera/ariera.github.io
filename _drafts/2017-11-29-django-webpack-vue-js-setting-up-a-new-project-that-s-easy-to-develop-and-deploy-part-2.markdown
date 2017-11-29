---
layout: post
title:  "Django + webpack + Vue.js - setting up a new project that's easy to develop and deploy (part 2)"
date:   2017-11-29 08:41:15 +0100
---

# Some notes:

from this email with Charles Aracil

    Issue was related on the fact that, in production mode, django does not serve static files (even with the 'runserver'). A webserver, like Apache or Gunicorn has to do it. Didn't know that.

    Anyway, to bypass that for testing in a development-like mode, we can call the runserver with an "--insecure" option.

    > python manage.py runserver --insecure

    That way I can assert built files are okay without requiring a real production rollout. Although I will test it anyway.

    So, have to wrap it up a little, but the repo should be fine soon :)

    Still got some issue with e2e test on Linux, probably due to a Selenium misconfiguration.

