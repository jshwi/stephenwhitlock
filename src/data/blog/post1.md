---
title: Adding versioning to Flask-SQLAlchemy with SQLAlchemy-Continuum
pubDatetime: 2025-12-15T05:17:00
slug: adding-versioning-to-flask-sqlalchemy-with-sqlalchemy-continuum
tags:
  - sqlalechmy-continuum
description: Versioning with Flask-SQLAlchemy and SQLAlchemy-Continuum
---

`SQLAlchemy` has a limited versioning extension that does not support entire 
transactions. 
[SQLAlchemy-Continuum](https://pypi.org/project/SQLAlchemy-Continuum/) offers a 
flexible API for implementing a versioning mechanism to your `SQLAlchemy`
[ORM](https://en.wikipedia.org/wiki/Object%E2%80%93relational_mapping) 
database.

If you want to ensure nothing is lost after editing your database I have found 
this package to be a good place to start. There are drawbacks when it comes to 
data retention; hashing algorithms and data compression aren't built into the 
API.

This article assumes you are using `Flask-SQLAlchemy`.

```shell
$ pip install sqlalchemy-continuum
```

In the module where your `SQLAlchemy` models are defined call 
`make_versioned()` before their definition and add `__versioned__` to all 
models you wish to add versioning to.

```python file=app/models.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from sqlalchemy.orm import configure_mappers
from sqlalchemy_continuum import make_versioned

app = Flask(__name__)

db = SQLAlchemy()

db.init_app(app)

make_versioned(user_cls=None)


class Post(db.Base):
    __versioned__ = {}
    ...


configure_mappers()
```

Database queries will now have a list-like object called `versions`

```python
from app.models import Post, db

id = 1  # first item in the db; we are using `get` and the db starts at 1
revision = 1  # however, with `post.versions` we are working with list indexes

# for a post object with at least 1 entry, and at least 2 versions
post = Post.query.get(id=id)
post.versions[revision].revert()
db.session.commit()
```

With the above, a third item has been added to `post.versions` , which is 
basically a duplicate of the first.

We can then craft a route that will restore the previous version:

```python
from app.utils.models import Post, db
from flask_login import login_required
from flask import redirect, url_for, Blueprint

bp = Blueprint("views", __name__)
...

@bp.route("/<int:id>/version/<int:revision>")
@login_required
def version(id, revision):
    post = Post.query.get(id=id)
    post.versions[revision].revert()
    ...
    db.session.commit()
    return redirect(url_for("views.index"))

...
```

Accessing `/1/version/1` after your URL will restore version 1 from post 1

If you already have an update view, much like the one demonstrated here 
https://flask.palletsprojects.com/en/2.0.x/tutorial/blog/ then we can also 
load the previous version with a query-string. With this we don't have to 
blindly restore versions (and add more duplicate versions along the way).

by default, the revision returned will be the last one i.e. `/1/update` is the 
same as `/1/update?revision=-1`.

This time we will return the version object - this way the form will be 
pre-loaded with the revision. Once the form's submission is validated the 
current `post` object will be replaced with restored revision. Once this is 
committed either a duplicate or edited version will be added to 
`post.versions`.

```python
from app.utils.models import Post, db
from flask import redirect, Blueprint, request, render_template
from flask_login import login_required
from app.utils.forms import PostForm

bp = Blueprint("views", __name__)
...


@bp.route("/<int:id>/update", methods=["GET", "POST"])
@login_required
def update(id):
    # by default, revision returned will be the last one
    revision = request.args.get("revision", -1, type=int)
    
    post = Post.query.get(id=id)
    
    # this time we will return the version object
    version = post.versions[revision]
    
    # this way the form will be preloaded with the revision
    form = PostForm(title=version.title, body=version.body)
    
    # once the form's submit button is pressed the current `post` object 
    # will be replaced with restored revision
    if form.validate_on_submit():
        post.title = form.title.data
        post.body = form.body.data
        ...
        
        # once this is committed either a duplicate or edited version 
        # will be added to `post.versions`
        db.session.commit()
        return redirect(url_for("views.index"))

    return render_template("update.html", post=post, form=form)

...
```