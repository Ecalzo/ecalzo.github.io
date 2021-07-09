---
layout: post
title: How I Set Up Database Migrations for my Serverless Flask App Deployed with Zappa
---

This blog post assumes you are familiar with
* [Flask](https://flask.palletsprojects.com/)
* [Flask-Migrate](https://flask-migrate.readthedocs.io/en/latest/) (alembic revisions for Flask apps)
* [Zappa](https://github.com/zappa/Zappa) (serverless deployment on aws)

# Setup
My application follows the `create_app` pattern outlined in the Flask [Flaskr](https://flask.palletsprojects.com/en/2.0.x/tutorial/index.html) example blog tutorial. Because of this, we need to make some adjustments to the usual deployment process when using Zappa to deploy our application.

Since we cannot call the `create_app()` function directly from `__init__.py` with Zappa, we will create a supplementary `run.py` file that instantiates our app, we will them point to this in `zappa_settings.json`. Note that `yt_pubsub_handler` is the name of the project. You can see it on [GitHub](https://github.com/Ecalzo/youtube-pubsub-handler).

{% highlight python %}
# run.py

from yt_pubsub_handler import create_app

app = create_app()

if __name__ == "__main__":
    app.run()

{% endhighlight %}

We can now point Zappa to this app variable:

{% highlight json %}
// zappa_settings.json
{
    "dev": {
        "app_function": "run.app",
        ...,
        ...,
    }
}
{% endhighlight %}

Zappa now knows where to look for our app!

# Database Migrations
If you are not familiar with database migrations, there are generally two options: `upgrade` and `downgrade`. An `upgrade` is when you make a new change to your database a structure, and a `downgrade` is when you revert that change. We need to be able to do this with our application database that is hosted on aws - but we can only really do this in production, since we don't want to expose database credentials and permissions to devs. We also would not want to allow anyone to run the changes locally, since they should go through the approval process on GitHub first.

## The Problem
[Flask-Migrate](https://flask-migrate.readthedocs.io/en/latest/) exposes a great CLI, but we cannot run CLI commands with Zappa. So we need a workaround - we need to use the `Flask-Migrate` API to create some helper functions that will allow us to upgrade and downgrade our database.

## The Solution
Note that I have already Introduced `Flask-Migrate` into my `create_app()` function and have run `flask db init` to generate a `migrations/` folder at the root of my project directory.

{% highlight python %}
# yt_pubsub_handler/__init__.py

def create_app():
    ...
    from flask_migrate import Migrate
    migrate = Migrate(app, db)
    return app

{% endhighlight %}

# Implementation

## Functions
Now we need to write some helper functions in order to run the CLI commands `flask db upgrade` and `flask db downgrade` via Zappa's python interface.

{% highlight python %}
# yt_pubsub_handler/db_utils.py

from flask import Flask
from flask_migrate import upgrade, downgrade

def alembic_upgrade(app: Flask):
    """Upgrades the database to the latest revision"""
    with app.app_context():
        upgrade()


def alembic_downgrade(app: Flask):
    """Downgrades the database to the previous revision"""
    with app.app_context():
        downgrade()

{% endhighlight %}

Notice that these functions take in an `app` variable, which is a reference to the Flask `app` object itself, that we first instantiated in `run.py`. So now we need to access these via the `run.py` file so that we can pass that app reference to these functions. Without the `app_context()` our functions will not know which database they should be interacting with. Note that `current_app` will not work in this scenario. So let's extend our `run.py` file:

{% highlight python %}
# run.py

from yt_pubsub_handler import create_app
from yt_pubsub_handler.db_utils import alembic_upgrade, alembic_downgrade

app = create_app()


def run_alembic_upgrade():
    alembic_upgrade(app)


def run_alembic_downgrade():
    alembic_downgrade(app)


if __name__ == "__main__":
    app.run()

{% endhighlight %}

## Creating a Migration
We can now create a branch and alter our database. To demonstrate, I made a change to `yt_pubsub_handler/models.py`, which `Flask-Migrate` automatically picks up.

The flow I use typically includes the following steps:

1. Start a local dev session
{% highlight shell %}
# I keep this in a local_dev.sh file
$ export FLASK_APP=yt_pubsub_handler
$ export FLASK_ENV=development
flask run
{% endhighlight %}

2. Edit the `models.py` file
3. Open a second terminal pane, ensure `export FLASK_APP=yt_pubsub_handler` is also set here
4. Run `flask db migrate` to automatically create a revision in the `migrations/versions/` folder
    * **Review this thoroughly** since it is auto-generated and may contain flaws
5. Commit changes and open a PR to merge the changes into production
6. Once merged and deployed with `zappa update production`, execute the migration with:
{% highlight shell %}
zappa invoke production 'run.run_alembic_upgrade'
{% endhighlight %}

For downgrade simply replace with `run_alembic_downgrade`.

# Conclusion
Here we used Flask-Migrate's API with some additional functionization to work with the Zappa serverless deployment framework. We can now make changes to our application database with ease. 

