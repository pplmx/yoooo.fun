---
categories:
    - python
date: 2020-09-18T11:14:26Z
description: Implement middleware in Flask
keywords: flask, wsgi, middleware, django
lastmod: 2020-11-11T16:31:07Z
tags:
    - flask
    - wsgi
    - middleware
title: Create a middleware in Flask
---



# Middleware in Flask

How to implement middleware in Flask, just like in Django?

In Flask, we can implement it by **WSGI middleware**.

## WSGI Middleware

A WSGI middleware component is a Python callable that is itself a WSGI application, but may handle requests by delegating to other WSGI applications.
These applications can themselves be WSGI middleware components.

A middleware component can perform such functions as:

- Routing a request to different application objects based on the target URL, after changing the environment variables accordingly.
- Allowing multiple applications or frameworks to run side-by-side in the same process
- Load balancing and remote processing, by forwarding requests and responses over a network
- Performing content post-processing, such as applying XSLT stylesheets

## Implementation

Here is a [simple demo](https://github.com/pplmx/LearningPython/tree/master/flask).

### log middleware

```python
from werkzeug import Request, Response

AUDIT_LOG = 'cc.log'


class OperationLogMiddleware:
    def __init__(self, app):
        self._app = app

    def __call__(self, environ, start_response):
        req = Request(environ)
        resp = Response(start_response)
        self._process_request(req)
        self._process_response(resp)
        return self._app(environ, start_response)

    @staticmethod
    def _process_request(request):
        with open(AUDIT_LOG, 'a+', encoding='UTF-8') as f:
            print(f'hello request: {request.method}', file=f)

    @staticmethod
    def _process_response(response):
        with open(AUDIT_LOG, 'a+', encoding='UTF-8') as f:
            print(f'hello response: {response.status_code}', file=f)

```

### flask main.py

```python
from flask import Flask, url_for
from werkzeug.utils import redirect

from middleware.operation_log import OperationLogMiddleware

app = Flask(__name__)

# add the custom middleware
app.wsgi_app = OperationLogMiddleware(app.wsgi_app)


@app.route('/')
def hello_world():
    return 'Hello World'


@app.route('/hello/<name>')
def hello(name: str):
    return f'Hello {name}!'


@app.route('/apply/<name>')
def hello_redirect(name: str):
    if name == 'admin':
        return redirect(url_for('hello_admin'))
    else:
        return redirect(url_for('hello_guest', guest=name))


@app.route('/admin')
def hello_admin():
    return 'Hello Admin'


@app.route('/guest/<guest>')
def hello_guest(guest):
    return f'Hello {guest} as Guest'


if __name__ == '__main__':
    app.run(debug=True)

```

