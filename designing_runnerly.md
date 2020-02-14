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


