Django Hello World Example for Stackato
=======================================

This example demonstrates on how you can deploy a Django application to Stackato on v3, which introduced a couple of changes to how applications are deployed.

# Deploying this Application

## Locally

### Prerequisites

Before you deploy this application locally, there are a couple of packages that you'll want to set up first:

 * [pip](http://www.pip-installer.org/en/latest/)
 * *optional*: [uwsgi](http://projects.unbit.it/uwsgi/)
 * *optional*: [virtualenv](http://www.virtualenv.org/en/latest/)
 * *optional*: [foreman](http://ddollar.github.io/foreman/)

I'm using uWSGI to deploy this example application, but you're more than welcome to use [gunicorn and nginx](http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/). The choice is yours.

Assuming that you're on an Ubuntu-based distribution (I use [Xubuntu](http://xubuntu.org/), but that's beside the point), here are the steps to download those dependencies. First, let's install uWSGI, postgresql, pip, git, and virtualenv in two easy steps:

```
$ sudo apt-get update
$ sudo apt-get install -qy python-pip python-virtualenv uwsgi-core libpq-dev postresql
```

Now, let's use RVM to install Ruby and Foreman:

```
$ sudo apt-get install -qy curl
$ curl -L https://get.rvm.io | bash -s stable --ruby=1.9.3 --gems=foreman
```

### Running the Application

Now that we have the dependencies installed, let's start our application:

```
$ virtualenv ~/venv
$ source ~/venv/bin/activate
$ git clone https://github.com/Stackato-Apps/django-polls.git
$ cd django-polls
$ pip install -r requirements.txt
$ python manage.py syncdb --noinput
$ python manage.py collectstatic --noinput
$ python manage.py createsuperuser
$ python manage.py runserver
```

You should now see your application running on port 8000. Head on over to http://localhost:8000/admin to see the administrator page.

## On Stackato

Deploying this application couldn't be any easier:

```
$ # target and login to the Stackato cluster
$ stackato target api.stackato-xxxx.local
$ stackato login
$ # if you haven't created a space yet, create one!
$ stackato create-space dev && stackato switch-space dev 
$ stackato push -n --as django-polls
$ # create a superuser in postgresql so we can gain access to the admin console
$ stackato run python manage.py createsuperuser
```

Then open up your browser to django-polls.stackato-xxxx.local/admin to see the administration page.

### Scaling up and down

You can also scale this app up and down!

```
$ stackato scale django-polls --instances 3
```

This command will spawn two more [Docker][docker] containers of the same application, bind them to the same postgresql data service, and the router will be updated to notify that it should balance the load amongst the two application instances.

### Destroying the application

Once you're done, delete the application with

```
$ stackato delete django-polls
```

# Changes in Deploying Applications to v3

In Stackato v2.10.6 and earlier, applications had a `stackato.yml` file that required the runtime and framework that you wanted to deploy your application. For v2.10.6, the runtimes and frameworks available looked like this:

```
$ stackato runtimes

+--------------+------------------+---------+
| Name         | Description      | Version |
+--------------+------------------+---------+
| java         | Java (default)   | 1.6     |
| java6        | Java 6           | 1.6     |
| java7        | Java 7           | 1.7     |
| node         | Node.js          | 0.6.20  |
| node010      | Node.js          | 0.10.1  |
| node08       | Node.js          | 0.8.22  |
| perl514      | ActivePerl 5.14  | 5.14    |
| perl516      | ActivePerl 5.16  | 5.16    |
| php          | PHP 5            | 5.3     |
| python27     | ActivePython 2.7 | 2.7     |
| python32     | ActivePython 3.2 | 3.2     |
| ruby18       | Ruby 1.8.7       | 1.8.7   |
| ruby19       | Ruby 1.9.3       | 1.9.3   |
+--------------+------------------+---------+

$ st frameworks

+---------------------+
| Name                |
+---------------------+
| buildpack - clojure |
| buildpack - go      |
| buildpack - java    |
| buildpack - play    |
| buildpack - pypy    |
| generic             |
| grails              |
| java_ee             |
| java_web            |
| lift                |
| node                |
| perl                |
| perlcgi             |
| php                 |
| play                |
| python              |
| rack                |
| rails3              |
| sinatra             |
| spring              |
| standalone          |
+---------------------+
```

In Stackato v3, the runtimes/frameworks keys have been deprecated and are now picked up by a [Heroku Buildpack's][buildpack] runtime/framework detection hooks. For a more detailed example on how these frameworks are detected, take a look at [Heroku's documentation on bin/detect](https://devcenter.heroku.com/articles/buildpack-api#bin-detect).


[buildpack]: https://devcenter.heroku.com/articles/buildpacks
[docker]: http://www.docker.io/
