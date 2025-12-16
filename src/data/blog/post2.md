---
title: Integrating Flask-Admin with the app factory pattern
pubDatetime: 2025-12-15T05:18:15
slug: integrating-flask-admin-with-the-app-factory-pattern
tags:
  - flask
  - flask-admin
  - python
  - web
description: Factory pattern with Flask-Admin
---

If you are entering `Flask` from `django` you'll notice that `Flask` doesn't
come with an admin interface. As per the `django` documentation for its admin
interface:

One of the most powerful parts of Django is the automatic admin interface.
One of the most powerful parts about `Flask`, however, is its lack of these things. Everything is up to you as the developer.

`Flask-Admin` can be used to set up an admin interface, so you can manage your
database through your browser.

There are 3 main issues, I found, when using `Flassk-Admin` features
out-of-the-box

It does not offer, for reasons of opinion, any security provisions; for this
we'll use `Flask-Login`
It reserves end point prefixes that you may want to use for other routes
it does not use the [app factory pattern](https://flask.palletsprojects.com/en/2.0.x/patterns/appfactories/)
(any attempt to use the app factory pattern will result in an error): This is
due to the extension registering its blueprints on instantiation

```text
AssertionError: A blueprint's name collision occurred between <flask.blueprints.Blueprint object at 0x25e5d90> and <flask.blueprints.Blueprint object at 0x21b89d0>.  Both share the same name "user".  Blueprints that are created on the fly need unique names.
```

`Flask-Admin`, however, is really easy to set up despite this.

```shell
$ pip install flask-admin flask-login
```

For a package to be recognised as an official `Flask` extension it should
include the `init_app` method, so that the app does not need to passed directly
to the extension upon instantiation. This enables the extension to be passed
around within the app's modules before it even exists. This prevents circular
imports, as is the reason for the app factory pattern. Luckily, `Flask-Admin`
does not need to passed around, and therefore works when it is instantiated
with the app object.

Your `extensions` module might look like something like this

```python file=app/extensions.py
from flask_login import LoginManager
...
from flask_sqlalchemy import SQLAlchemy

...
login_manager = LoginManager()
...
sqlalchemy = SQLAlchemy()
...


def init_app(app):
    ...
    login_manager.init_app(app)
    ...
    sqlalchemy.init_app(app)
    ...
```

To be registered in the app's `__init__.py` file like this:

```python file=app/__init__.py
from flask import Flask

...
from app import extensions
...


def create_app():
    app = Flask(__name__)
    ...
    extensions.init_app(app)
    ...
    return app
```

Create a new app/admin.py module and write up a new `init_app` function

Solution to 1

`Flask-Login` comes with the extremely useful `login_required` decorator. Put
this on top of your routes, and the user is required to be logged in to access
it. We can do this with an admin user too. Add a boolean value to your user
model; `admin`

```python file=app/models.py
from flask_login import UserMixin

from app.extensions import db

...


class User(UserMixin, db.Model):
    ...
    admin = db.Column(db.Boolean, default=False)
    ...


...
```

Now we can decorate the `login_required` functionality with a more restricted,
`admin` function. `login_required` , under the hood, returns a
`LoginManager.unauthorized` function if the user is not logged in. This raises,
among other things, raises a `401 Unauthorized` error.

How you make your user an admin is pretty simple, but I won't go into it here.

```python
import functools

from flask_login import current_user

from app.extensions import login_manager


def admin_required(func):
    """Handle views that require an admin be signed in.

    :param func: View function to wrap.
    :return: The wrapped function supplied to this decorator.
    """

    @functools.wraps(func)
    def _wrapped_view(*args, **kwargs):
        if not current_user.admin:
            return login_manager.unauthorized()

        return func(*args, **kwargs)

    return _wrapped_view
```

Create a new admin.py module. `Flask-Admin`, by default, constructs its index
page with the `AdminIndexView` class. We can override this class and its
`index` method to be decorated with our `admin_required` function. Make sure to
continue using `login_required` as a sort of hierarchy of logins, otherwise the
`admin_required` decorator will fail. This is because the decorator needs the
user to logged in to check for admin.

```python file=app/admin.py
from flask_admin import AdminIndexView, expose
...
from flask_login import login_required

from app.utils.security import admin_required


class MyAdminIndexView(AdminIndexView):
    """Custom index view that handles login / registration."""

    @expose("/")
    @login_required
    @admin_required
    def index(self) -> str:
        """Requires user be logged in as admin."""
        return super().index()
```

We can then write our init_app function like this.

```python app/admin.py
from flask_admin import Admin
...


def init_app(app):
    admin = Admin(  # noqa
        app,
        index_view=MyAdminIndexView(),  # type: ignore
    )
    ...
```

Solution to problem 2

We can subclass `flask_admin.contrib.sqla.ModelView` to register their
blueprints under a different name.

Now you can use the `/user` , `/post` , etc. prefixes on your user, post etc.
routes, and not waste them on the admin routes.

Because the views display, usually, multiple user, post etc. tables, it makes sense to end them as a plural.

It is also worth pointing out that we have blocked access to these routes for non-admins too.

```python file=app/admin.py
from flask_admin import Admin
from flask_admin import AdminIndexView
from flask_admin.contrib import sqla
from flask_login import current_user

from app.extensions import db
from app.utils.models import User, Post, Task, Message, Notification


class MyAdminIndexView(AdminIndexView):
    ...


class MyModelView(sqla.ModelView):
    """Custom model view that determines accessibility."""

    def __init__(self, model: db.Model, session: db.session) -> None:
        super().__init__(model, session)
        self.endpoint = f"{model.__table__}s"  # change the table name here

    def is_accessible(self) -> None:
        """Only allow access if user is logged in as admin."""
        return current_user.is_authenticated and current_user.admin

    ...

def init_app(app):
    admin = Admin(  # noqa
        app,
        index_view=MyAdminIndexView(),  # type: ignore
    )
    admin.add_view(MyModelView(User, db.session))
    admin.add_view(MyModelView(Post, db.session))
    admin.add_view(MyModelView(Message, db.session))
    admin.add_view(MyModelView(Notification, db.session))
    admin.add_view(MyModelView(Task, db.session))
```

Finally, as a solution to 3, we initialize the app

```python file=app/__init__.py
from flask import Flask

...
from app import extensions
from app.routes import admin
...


def create_app():
    app = Flask(__name__)
    ...
    extensions.init_app(app)
    admin.init_app(app)
    ...
    return app
```
