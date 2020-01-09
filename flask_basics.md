# Flask basics 

<br/>

## Routing


Routing rules are stored in `app.url_map`, where `type(app) == Flask`.


> **NOTE**: `OPTIONS` and `HEADS` methods are automatically added to all routing rules by default. It can be configured by `provide_automatic_options` setting.


GET-mapping an endpoint to view:
```python
@app.route('/api')
def my_view(...):
```


Specifying other HTTP methods for rule:
```python
@app.route('/api', methods=['GET', 'POST', 'DELETE'])
def my_view(...):
```

<br/>

### Variables and converters

Parametrized url-mapping:
```python
@app.route('/api/person/<person_id>')
def person(person_id):
```


Parametrized url-mapping with converter:
```python
@app.route('/api/person/<int:person_id>')
def person(person_id):
    # type(person_id) == int
```


Built-in converters:
- string (**default**)
- int
- float
- path
- any
- uuid


> NOTE: You can create your own custom coverters if needed.


Use `flask.url_for` function to get a URL for particular view.

<br>

## Helper functions:
- jsonify(dict) -> Response

<br>

## Globals per Request Context (thread-safe):
- _request_ - an instance of Request + some mixins
- _session_ - dict-like object, that holds a session cookie. When serializing a session data, Flask signs it using `itsdangerous` library and `secret_key` (defined at the application level). Default signing algorithm is HMAC + SHA1, but it is customizable.

You can add your own globals to request context via `flask.g` object. 

```python
from flask import Flask, jsonify, g, request
app = Flask(__name__)

@app.before_request
def authenticate():
    if request.authorization:
        g.user = request.authorization['username']
    else:
        g.user = 'Anonymous'

@app.route('/api')
def my_microservice():
    return jsonify({'Hello': g.user})

if __name__ == '__main__':
    app.run()
```

<br>

## Response
Usually a view returns a callable that accepts WSGI's `environ` and `start_response` just like a WSGI application.  

If not Flask expects to get:
- str/bytes/bytesarray to put into response body
- 2-3 elements tuple: (reponse, [status], [headers])

<br>

## Signals
**_ATTENTION: To use Flask signals feature you must have Blinker library (https://pythonhosted.org/blinker/) installed._**

**Signals** are events that you can handle **inside** of one service, for example to log something. If you want to trigger some external job, use some message queue instead, like RabbitMQ.
There is a set of built-in signals, some extensions provide their own signals and you also can create your own ones.
Some signals are similar to builtin decorators like `request_started` vs. `app.before_request()`, but:
- signals **cannot** modify data and affect further request handling
- signals handlers are executed in undetermined order

<br>

## Extension and Middlewares
**Flask extensions** are used to add some custom reusable functionality (admin panel, auth, analytics), to integrate Flask app with other libs/services (email, database) or toprovide frameworks to solve some typical problems (REST).

**Middlewares** are WSGI interseptors that take place before Flask app gets a request. So in middleware you deal with WSGI abstractions (environ, start_response) instead of Flask abstractions (request, response, session). 
Unless you want some functionality to be pluggable to any WSGI framework, you better use Flask extensions.

<br>

## Templates
For any text-based documents (HTML, PDF, email) Flask provides Jinja.

Jinja supports:
- for loops
- if statements
- blocks
- and more...

Example:
```
Date: {{date}}
From: {{from}}
Subject: {{subject}}
To: {{to}}
Content-Type: text/plain
Hello {{name}},
Below is the list of items we will deliver for lunch:
{% for item in items %}- {{item['name']}} ({{item['price']}} Euros)
{% endfor %}
{% if be_nice %}
Thank you for your business!
{% endif %}
--
Tarek's Burger
```

<br>

## Configuration
Flask stores all the configuration in app.config which is a dict-like object.

```python
# Changing the configuration manually
app.config['DEBUG'] = True

# Loading from python object
app.config.from_object('configs.ProdConfig')

# Loading from static config file
import json
dev_config = json.load('settings.json')
app.config.update(dev_config)
```

<br>

## Blueprints
**Blueprint** - is a part of a Flask app, that just like app does, may have its own routing, views, templates and static files. But Blueprint can't be independant and must be registered as a part of some Flask app.

Why Blueprints:
- breaking an app into logical parts to keep them simpler, smaller and easier to work with (Single Responsibility principle)
- separating handlers of different endpoints (e.g. 1 blueprint = 1 endpoint = 1 model in REST)
- one Blueprint can be register multiple times with different URLs and other parameters

The following is from the Flask-Restless documentation (Person is SQLAlchemy model):
```python
blueprint = manager.create_api_blueprint(Person, methods=['GET', 'POST'])
app.register_blueprint(blueprint)
```

<br>

## Error Handling
By default Flask will return an HTML page with corresponding error code (404, 500, etc).

Customization of error handling:
```python
@app.errorhandler(500)
def error_handling(error):
    return jsonify({'Error': str(error)}, 500)
```
