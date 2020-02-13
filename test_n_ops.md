# Testing and Operations

<br/>

## Testing

### Unit tests
Testing a module/class/function in isolation (dependencies mocked).
Tooling: 
- pytest 
- pytest_mock

### Functional tests
Testing the whole microservice functionality and still mocking network interactions.

Tooling:
- Flask provides `FlaskClient` class that simulates HTTP interaction
- OR WebTest can be used instead

```python
import json
from flask_basic import app as tested_app

class TestApp():
    def test_help(self):
    # creating a FlaskClient instance to interact with the app
    app = tested_app.test_client()
    # calling /api/ endpoint
    hello = app.get('/api')
    # asserting the body
    body = json.loads(str(hello.data, 'utf8'))
    assert body['Hello'] == 'World'
```

### Integration tests
Testing microservices integration with each other and other network resources.

### Load tests
Testing performance and maximum workload.

Tooling:
- boom (simple load test)
- molotov (load tests with scenarios)
- flask-profiler (collects metrics while load-testing)

### End-to-end tests
Testing the whole system from the user perspective.

Tooling: 
- Selenium (scripts for automating browser)

### Additional testing tools

- pytest-cov (measuring Coverage)
- pytest-flake8 (style checks)
- tox (testing on multiple interpreter versions)

<br/>

## Documentation

Tooling:
- Sphinx (tool for generating documentation)
- Autodoc (extension for Sphinx that injects docstrings into docs)
- ReadTheDocs (free docs hosting for Sphinx docs; maybe hooked to CI)


## Continuous Integration

CI tools:
- Travis CI
- GitLab CI
- GitHub Actions

### Additional tools
Coveralls: hosts your coverage data to show a nice green button in GitHub.