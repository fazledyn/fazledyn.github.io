---
layout: post
title:  "Deploying Django 3.1 on Heroku"
categories: [ notes ]
tags: [ featured ]
image: assets/images/7.jpg
lang: en
---

It’s been long since I started learning and working with Django. Though it doesn’t give much editability like Flask. But it sure it does provide a lot of pre-built components such as the Admin Panel and pre-defined user models.

First time I deployed Django on Heroku, it was a pain in the ass. I followed Heroku’s official documentation, Django’s official documentation, Medium blogs, Stack Overflow and what not. But nothing really helped completely. As a result, I had to trial and error.

Django requires a gunicorn server to serve itself through the wsgi.py file. So yeah, it’s the gunicorn server and the wsgi.py file that plays vital role while deploying in a SaaS system like Heroku. I am sure that I’ll have to deploy another Django app in Heroku and I’ll face the same problems over and over again. So, I have thought of documenting the whole procedure here-

---

### Wsgi.py

Go to Django project’s wsgi.py file.

Add this `os.environ["DJANGO_SETTINGS_MODULE"] = "<PROJ_NAME>.settings"` instead of `os.environ.setdefault('DJANGO_SETTINGS_MODULE',**'<PROJ_NAME>.settings')`
Make sure the Procfile has the following line (only): `web: gunicorn <PROJ_NAME>.wsgi --log-file -`

The `–log-file –` attribute helps generating logs for gunicorn server. This is where I made most of my mistakes while deploying Django apps.
If you’re deploying Django in debug mode, add this line to the end of project’s urls.py file. Otherwise, no need to add this. This is basically for handling media files (not static files)

{% highlight python %}
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
{% endhighlight %}


DO NOT add `DISABLE_COLLECTSTATIC=1` in Heroku configuration/ENV file. This just makes it worse. Disabling the collect static will just tell Gunicorn not to collect static files. Well, this might work well for serving own static files but if you try using the admin site for example, the site won’t load as usual. Because the CSS file that admin site uses, won’t get collected like other template files.

For Python 3.6+ it’s better to add a `runtime.txt` file. It just so happens that some of Python’s module require the newer version of Python. And as far as I know, Heroku’s Free Dynos don’t comes with the newest version (3.7+) of Python. You need to mention it explicitly.

### Settings.py

- Install `dj_database_url` package using `pip install dj_database_url`
- Add website URL to `ALLOWED_HOST` list.
- No need to install & import `django_heroku` package. This thing is complete shit. Doesn’t work as told in Heroku’s documentation. Otherwise I wouldn’t be writing this.
- No need to add Whitenoise middleware as well.

### Database Config

The newer version of Django (3.0.8+) creates the settings.py DATABASE dictionary in the following way: `'NAME': BASE_DIR/'db.sqlite3'`

**This is where the problem arises**. The gunicorn in Heroku’s Dyno can’t interpret this easily. So, we need to use .join() method to create the database directory. It should be like this: `'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),`

Be sure to declare MEDIA_URL, MEDIA_ROOT, STATIC_URL, STATIC_ROOT, STATICFILE_DIRS constants in the following way:

{% highlight python %}
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
STATICFILE_DIRS = ["static/images", "static/css", "staticfiles",]
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
{% endhighlight %}

Finally, add production configuration in settings.py like this:

{% highlight python %}

if os.getcwd() == '/app':

import dj_database_url
db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)

SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
ALLOWED_HOSTS = ['<APP_NAME>.herokuapp.com']
DEBUG = True
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(file)))

{% endhighlight %}

That's it, it is now good to go.