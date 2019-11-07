# Flask Microservices Cheet Sheet

This is a summary of "Python Microservices Development" book by Tarek Ziade.  

<br/>

## Flask basics 

<br/>

#### Routing


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

##### Variables and converters

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

<br/>

#### Helper functions:
- jsonify(dict) -> Response

<br/>

#### Globals (per thread, so thread-safe):
- _request_ - an instance of Request + some mixins
- _session_ - dict-like object, that holds a session cookie. When serializing a session data, Flask signs it using `itsdangerous` library and `secret_key` (defined at the application level). Default signing algorithm is HMAC + SHA1, but it is customizable.


##### Response
Usually a view returns a callable that accepts WSGI's `environ` and `start_response` just like a WSGI application.  

If not Flask expects to get:
- str/bytes/bytesarray to put into response body
- 2-3 elements tuple: (reponse, [status], [headers])
