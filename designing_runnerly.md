# Designing Runnerly

In this chapter author creates a tiny social network for runners. It is integrated with Strava service, where runners log their runs and motinor the progress.

<br/>

## User Stories

- As a user, I can create an account on Runnerly with my email and activate it through a confirmation link I receive in my mailbox.
- As a user, I can connect to Runnerly and link my profile to my Strava account.
- As a connected user, I can see my last 10 runs appear in the dashboard.
- As a connected user, I can add a race I want to participate in. Other users can see the race as well in their dashboard.
- As a registered user, I receive a monthly report by email that describes how I am doing.
- As a connected user, I can select a training plan for a race I am planning to do, and see a training schedule on the dashboard. A training plan is a simple list of runs that are not done yet.

## Functionality

- The app needs a **registration** mechanism that will add the user to our database and make sure they own the email used for registration.
- The app will **authenticate** users with a password.
- To pull data out of **Strava**, a strava user token needs to be stored in the user profile and used to call the service.
- Besides runs, the database needs to store **races and training plans**.
- **Training programs** are a list of runs to be performed at specific dates to be as performant as possible for a given race. Creating a good training plan requires information about the user, such as their age, sex, weight, and fitness level.
- **Monthly reports** are built by querying the database and generating a summary sent by email.

## Monolothic design

The application is going to follow classic MVC design pattern. 

**Model**: represents/manages the data
**View**: responsible for application output format (Web UI, PDF, JSON, etc.)
**Controller**: handles incoming requests and manipulates the data

### Model

Models:
- User
- Run
- Race 
- Plan

**flask_sqlalchemy** extension integrates Flask with SQLAlchemy ORM.

User model definition:
```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    email = db.Column(db.Unicode(128), nullable=False)
    firstname = db.Column(db.Unicode(128))
    lastname = db.Column(db.Unicode(128))
    password = db.Column(db.Unicode(128))
    strava_token = db.Column(db.String(128))
    age = db.Column(db.Integer)
    weight = db.Column(db.Numeric(4, 1))
    max_hr = db.Column(db.Integer)
    rest_hr = db.Column(db.Integer)
    vo2max = db.Column(db.Numeric(4, 2))
```

### View and Template

Simple app initialization with view for getting a list of users:

```python
from flask import Flask, render_template
app = Flask(__name__)
@app.route('/users')
def users():
users = db.session.query(User)
return render_template("users.html", users=users)
if __name__ == '__main__':
db.init_app(app)
db.create_all(app=app)
app.run()
```

`users.html` template:
```
<html>
    <body>
        <h1>User List</h1>
        <ul>
            {% for user in users: %}
            <li>
                {{user.firstname}} {{user.lastname}}
            </li>
            {% endfor %}
        </ul>
    </body>
</html>
```

#### Forms

For editing data with server-side rendering architechture, **WTFroms** can be used. **Flask-WTF** wraps WTForms for integration with Flask.

User create/edit form definition:
```python
from flask_wtf import FlaskForm
import wtforms as f
from wtforms.validators import DataRequired

class UserForm(FlaskForm):
    email = f.StringField('email', validators=[DataRequired()])
    firstname = f.StringField('firstname')
    lastname = f.StringField('lastname')
    password = f.PasswordField('password')
    age = f.IntegerField('age')
    weight = f.FloatField('weight')
    max_hr = f.IntegerField('max_hr')
    rest_hr = f.IntegerField('rest_hr')
    vo2max = f.FloatField('vo2max')
    display = ['email', 'firstname', 'lastname', 'password', 'age', 'weight',
               'max_hr', 'rest_hr', 'vo2max']
```

Create/edit view:
```python
@app.route('/create_user', methods=['GET', 'POST'])
def create_user():
    form = UserForm()
    if request.method == 'POST':
        if form.validate_on_submit():
            new_user = User()
            form.populate_obj(new_user)
            db.session.add(new_user)
            db.session.commit()
            return redirect('/users')
    return render_template('create_user.html', form=form)
```

```
<html>
    <body>
        <form action="" method="POST">
            {{ form.hidden_tag() }}
            <dl>
            {% for field in form.display %}
                <dt>{{ form[field].label }}</dt>
                <dd>{{ form[field]() }}</dd>
                {% if form[field].errors %}
                    {% for e in form[field].errors %} <p>{{ e }}</p> {% endfor %}
                {% endif %}
            {% endfor %}
            </dl>
            <p>
            <input type=submit value="Publish">
        </form>
    </body>
</html>
```

With **WTForms-Alchemy** form definition can be as follows:
```python
from wtforms_alchemy import ModelForm

class UserForm(ModelForm):
    class Meta:
        model = User
```

### Background Tasks

Runnerly will perform some background tasks for pulling data from external Strava service.

A popular way to run repetative background tasks is **Celery**.
Celery needs a message broker, like:
- Redis
- RabbitMQ
- Amazon SQS

`background.py`:
```python
from celery import Celery
from stravalib import Client
from monolith.database import db, User, Run

BACKEND = BROKER = 'redis://localhost:6379'

celery = Celery(__name__, backend=BACKEND, broker=BROKER)
_APP = None

def activity2run(user, activity):
    """Used by fetch_runs to convert a strava run into a DB entry."""
    run = Run()
    run.runner = user
    run.strava_id = activity.id
    run.name = activity.name
    run.distance = activity.distance
    run.elapsed_time = activity.elapsed_time.total_seconds()
    run.average_speed = activity.average_speed
    run.average_heartrate = activity.average_heartrate
    run.total_elevation_gain = activity.total_elevation_gain
    run.start_date = activity.start_date
    return run

@celery.task
def fetch_all_runs():
    global _APP
    # lazy init
    if _APP is None:
        from monolith.app import app
        db.init_app(app)
        _APP = app
    else:
        app = _APP
    
    runs_fetched = {}
    with app.app_context():
        q = db.session.query(User)
        for user in q:
            if user.strava_token is None:
                continue
        runs_fetched[user.id] = fetch_runs(user)
    return runs_fetched

def fetch_runs(user):
    client = Client(access_token=user.strava_token)
    runs = 0
    for activity in client.get_activities(limit=10):
        if activity.type != 'Run':
            continue
        q = db.session.query(Run).filter(Run.strava_id == activity.id)
        run = q.first()
        
        if run is None:
            db.session.add(activity2run(activity)
            runs += 1
            db.session.commit()
    return runs
```

View for synchronous fetching all runs:
```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/fetch')
def fetch_runs():
    from monolith.background import fetch_all_runs
    res = fetch_all_runs.delay()
    res.wait()
    return jsonify(res.result)
```

