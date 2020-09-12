## Database Connection Pattern
Choosing your database pattern is an early project decision that is critical to settle quickly. Making the wrong decision can result in costly and frustrating rework throughout the entire application should you ever need to change mid-stride. While I'm unable to comment on support issues that may come up in Production, I can comment on the process I had to go through to settle on the database connection pattern that I'll be using in the project

### Decision: Use the SQLAlchemy ORM ... Sort of
As per my [technology component decisions](./02-project-goals-and-design-considerations.md) I said I would be using the SQLAlchemy ORM. How I reached that decision was an evolutionary process based on development efforts I conducted before I began to write this material. To be honest, I'm still not convinced it even makes sense to use SQLAlchemy (I'll explain why once I detail my steps), but at this point in time I don't want to have to retrofit yet again so ... I'll stick with the ORM for now.

### Caveat: Take what I say with a grain of salt
Before I begin, I should make it clear that I'm not 100% in my mastery of this topic. I have tried to learn the SQLAlchemy ORM before and consistently found it to feel unnatural and confusing. With that said, I have not spent huge amounts of time reading through the large body of documentation and - when I did read portions of it - more often than not my reaction was "err, wut?". 

What I'm trying to say is: I *think* I don't like using an ORM.

I understand that using an ORM is supposed to facilitate life by abstracting the mechanics of database interaction and allowing me to stay in a Pythonic syntax world. I understand that an application using an ORM can be more easily forklifted to a different database solution, requiring only minor changes to the database connection settings. I also understand that the ORM offers coding shortcuts to access record data (e.g. relationship definitions and `Object.query.filter_by()...` rather than `db.session.query(Object).filter_by()...`. 

The problem is that I don't find these reasons to be good enough counterbalance the headspace I must devote to keeping track of what the ORM is doing for me and how I'm supposed to use it.

There are tons of opinions on the web supporting the pro- and anti- ORM cause. Both sides appear to be well-informed and can back up their cases with plenty of technical facts. The comment sections are then filled with anecdotes like _"sure you can get away without using an ORM when you are building a small project, but GOOD LUCK trying to maintain a multi-million line entreprise code base without one!"_, _"We built our application using an ORM but now the ORM isn't doing what we need it to do and have spent the last six months struggling to rip it out of our codebase"_, and _"Won't somebody PLEASE think of performance!!"_ (this one is used by both sides).

I don't know who the authors or commenting practitioners are. I don't know what solutions they built. I don't know how those who chose to use an ORM architected their integration. I don't know what solutions those who chose not to use an ORM used to avoid having to write bespoke SQL for every transaction. I don't know if it's realistic to expect to the availability of a complete, up-to-date Logical Data Model if one is brought in as a developer on a large legacy codebase. I don't know who or what to trust, and this makes it exceedingly hard to make an informed decision regarding my own project.

I CAN comment on my own experience trying to get an ORM (`flask-sqlalchemy` and the `SQLAlchemy`) integrated my own small project, and the thoughts I had while observing myself trying to get the initial solution working as well as wondering "How would I get this to scale if this was more than a one-man operation?".

With my digressing out of the way, onto the main points.

### Choosing Your Database Integration Model
I'll just come out and say it: I really don't like using an ORM.







#### How about you stop digressing and get to the point?
Ok ok, my beef with ORMs is centred around the following 4 points:
1. Registering the ORM when using the Flask Application Factory (tight coupling)
2. Intermingling SQL abstractions with object relationships
3. Obfuscation of underlying database interactions
4. Learning the ORM rather than learning SQL

##### Beef #1: Registering the ORM with the Flask Application Factory Model
This is the most minor of the beefs, but important to call out due to its impact on application architecture.

Consider the following application:

```python
# __init__.py

from flask import Flask
from flask_sqlalchemy import SQLAlchemy

# Globally accessible db
db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    db.init_app(app)
    
    with app.app_context():
        db.create_all()  # Create sql tables for our data models
```
Have a look at Todd Birchard's [Demystifying Flask's Application Factory](https://hackersandslackers.com/flask-application-factory/) article for a step-by-step breakdown of how the database is registered with the Flask application, and [Connect Flask to a Database with Flask-SQLAlchemy](https://hackersandslackers.com/flask-sqlalchemy-database-models/) to understand how the database tables are created once the db is registered with the app. 

The two most crucial items to remember are:
1. The database connection is not available for use until it has been registered with the application (_at runtime_).
1. The database is not created until the first time the application is executed.

This design works fine for the immediate need of providing the application with stateful storage. We've done it, however, in a manner that tightly couples the application with its database and this precludes the reuse of our connection code should we wish to interact with the database independently (e.g. perhaps I want to pre-populate the table with records before i run the Flask application). 

#### Split the database instantiation from the application instantiation
I started searching for a way to split the database instantiation from the application instantiation and found a gem of an article in Bob Waycott's [Organizing Flask Models with Automatic Discovery](https://bobwaycott.com/blog/how-i-use-flask/organizing-flask-models-with-automatic-discovery/#sqlalchemy-database-setup) (I was actually looking for a way to define ORM table classes over multiple files, but more on that later).

Waycott creates the database connection object in a separate `db.py`. He then provides a function that can be called by the Flask Application Factory in `__init__.py` that will bind the flask_sqlalchemy object to the Flask application. (_Note: I've omitted some of his code to streamline my example_):
```python
# proj/db.py

from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def init_db(app=None, db=None):
    """Initializes the global database object used by the app."""
    if isintance(app, Flask) and isinstance(db, SQLAlchemy):
        db.init_app(app)
    else:
        raise ValueError('Cannot init DB without db and app objects.')
```

This design means that the `__init__.py` logic changes to:
```python
# __init__.py

from flask import Flask
from proj.db import db, init_db

def create_app():
    app = Flask(__name__)
    init_db(app=app, db=db)
    
    with app.app_context():
        db.create_all()  # Create sql tables for our data models
```

We can also independently interact with the database from any other file in the project like so:
```python
# some_file.py
from proj.db import db
```

#### Great! Problem solved, right?
Not quite. Waycott's solution gives us an independent entrypoint to the database, but we still have to register the flask_sqlalchemy plugin with the Flask application. This probably isn't a deal-breaker but I didn't like this for a few reasons:
1. I had to write an (albeit small) function to tie the objects together.
1. It made me dependent on the flask-sqlalchemy package (which wraps the underlying SQLAlchemy library).
1. flask-sqlalchemy has a slightly different syntax for defining database objects than using SQLAlchemy directly.

Even though the SQLAlchemy documentation itself suggests using a helper library like flask-sqlalchemy !ADD REFERENCE!, something felt not quite right. Furthermore, whenever I subsequently searched for database objection creation help, I would always need to check if the answer was written from a SQLAlchemy or flask-sqlalchemy perspective. My assumption was that - because flask-sqlalchemy wrapped SQLAlchemy - I was better off using the base package as this would give me a greater chance of finding answers to the problems I was trying to solve. This opinion was reinforced by the arguments made in Edward Krueger's [Use Flask and SQLAlchemy, not Flask-SQLAlchemy!](https://towardsdatascience.com/use-flask-and-sqlalchemy-not-flask-sqlalchemy-5a64fafe22a4?gi=6c73d7f74e07) article.

As a result, I decided to abandon flask-sqlalchemy and rework the code to use SQLAlchemy directly. The new `db.py` file looked like:
```python
# proj/db.py

from sqlalchemy import create_engine
from sqlalchemy import sessionmaker, scoped_session
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
engine = create_engine('sqlite:////tmp/mysqlitedb.db')

session_factory = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Session = scoped_session(session_factory)

db = SQLAlchemy()

def init_db():
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)
    
```

The redesigned `db.py` required changes in the `__init__.py` file too:
```python
# __init__.py

from flask import Flask
from proj.db import init_db, Session

def create_app():
    app = Flask(__name__)
    init_db()  # This creates the database without needing to bind the db to the app
    
    with app.app_context():
        # db = Session()  # The db object provides the Flask app with access to the db
        ...
```
With this change implemented, the database logic was successfully split away from the Falsk application logic. I was finally done with all this annoying refactoring work and could get back to writing code! Or so I thought ...

#### OMG why am I getting all these errors from the Flask Development server?!
Earlier in this series, I devoted a full post to explaining [how and why I chose my technology components](./02-project-goals-and-design-considerations.md). When discussing the database solution, I noted that I opted to use SQLite despite its limitations regarding concurrent connections. This "limitation" began to cause trouble for me almost immediately (_admittedly due to my own programming stupidity_), and was due to two reasons:

1. The Flask development server initialization behaviour
2. Neglecting to close my database connections

##### What was causing the problem? (Hint: mostly me)
NOTE: GW TO TRY THE @app.before_first_request decorator to see if this solves the problem.
Flask is built upon the Werkzeug library. When Flask is started in development mode, Werkzeug will spawn a child process in order to [restart the main process each time the application code changes](https://stackoverflow.com/questions/25504149/why-does-running-the-flask-dev-server-run-itself-twice). This behaviour, however, also expresses itself when the application is first initialized.

Part of my `__init__.py: create_app()` function tries to seed the SQLite database with records. Given the double-execution behaviour of the Flask development server, this meant two processes were trying to connect to the database (each trying to write the same records). I was quite capable at opening database connections via the Session() object but I had neglected to write any code that _closed_ the database connection. This meant that the second process received a process conflict error (TO DO: GO FIND THE EXACT ERROR), but SQLite was still waiting for the first process to close its connection. Oops.

Before I figured out the root cause of the problem, I created a workaround procedure based on a StackOverflow post that discussed how to deal with a [locked database](https://stackoverflow.com/questions/26862809/operational-error-database-is-locked), and tied into my `__init__.py` file.

```python
# proj/db.py

from sqlalchemy import create_engine
from sqlalchemy import sessionmaker, scoped_session
from sqlalchemy.ext.declarative import declarative_base

import os
import random
import time

Base = declarative_base()
engine = create_engine('sqlite:////tmp/mysqlitedb.db')

session_factory = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Session = scoped_session(session_factory)

db = SQLAlchemy()


def init_db():
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)


def commit_or_rollback(entry):
    print(f'PID is: {os.getpid()}')
    # Note: 'Finally' not used because we break on success. but need to retry if processes clash.

    for attempt in range(3):
        db = Session()
        
        try:
            db.session.add_all(entry)
            db.session.commit()
            return
        except exc.IntegrityError as e:
            print(f'Record already exists in db.\n{e}')
            db.session.rollback()
            break
        except exc.OperationalError as e:
            print(f'PID {os.getpid()} encountered SQLALCH OperationalError(sqlite3.OperatioalError. sleeping')
            print(e)
            db.session.rollback()
            time.sleep(random.randint(1, 3))
```

```python
# __init__.py

from flask import Flask
from proj.db import init_db, Session
from proj.helpers import load_initial_data

def create_app():
    app = Flask(__name__)
    init_db()  # This creates the database without needing to bind the db to the app
    
    with app.app_context():
        # db = Session()  # The db object provides the Flask app with access to the db
        load_initial_data()
        ...
```

```python
# proj/helpers.py

from proj.db import commit_or_rollback
from proj.models import User  # User was a SQLAlchemy class that I created elsewhere in the application

u1 = User('user1')
u2 = User('user2')
users = [u1, u2]
commit_or_rollback(users)
...

```
The workaround prevented the Flask app from completely crashing on the initial double load, but was still causing chaos when I was trying to serve up dynamically populated Jinja2 templates (_not surprising given that I still hadn't bothered to close the database connection anywhere in the code!_).

##### How did I solve the problem? (Hint: Context Manager)
I know it comes as a shock, but closing a database connection is *kinda* important. I was also consistently forgetting to do so. Something had to change or else building out the project was going to be come a nightmare.
 
TO DO: ADD CONTEXT MANAGER LINK
I happily used Context Managers to facilitate file read/write operations, so could I do the same thing with my database connection? Article xxxxx said yes. I set out to refactor the code ... again.

The database connection was moved from the global scope of `proj/db.py` into a custom `SQLAlchemyDBConnection` object that had defined `__enter__` and `__exit__` dunders (thereby allowing me to use it as a context manager via the `with` keyword). I also modified the `commit_or_rollback()` procedure to use the `SQLAlchemyDBConnection` class. 

TO DO: MAKE IT MORE CLEAR WHAT THE IMPLICATIONS OF LOSING THE scoped_session DID. (NEED TO READ MORE).
TO DO: MAKE IT MORE CLEAR THAT i DID NOT IMPLEMENT BOB'S query HELPER SINCE I WANTED A CONSISTENT DB INVOCATION TECHNIQUE

```python
# proj/db.py

from sqlalchemy import create_engine
from sqlalchemy import sessionmaker
from sqlalchemy.ext.declarative import declarative_base

import os
import random
import time

Base = declarative_base()
engine = create_engine('sqlite:////tmp/mysqlitedb.db')


class SQLAlchemyDBConnection(object):
    """SQLAlchemy database connection"""

    def __init__(self, connection_string='sqlite:////tmp/mysqlitedb.db'):
        self.connection_string = connection_string
        self.session = None

    def __enter__(self):
        engine = create_engine(self.connection_string)
        Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
        self.session = Session()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.session.close()


def init_db():
    Base.metadata.drop_all(bind=engine)
    Base.metadata.create_all(bind=engine)


def commit_or_rollback(entry):
    print(f'PID is: {os.getpid()}')
    # Note: 'Finally' not used because we break on success. but need to retry if processes clash.

    for attempt in range(3):
        with SQLAlchemyDBConnection() as db:
        
            try:
                db.session.add_all(entry)
                db.session.commit()
                return
            except exc.IntegrityError as e:
                print(f'Record already exists in db.\n{e}')
                db.session.rollback()
                break
            except exc.OperationalError as e:
                print(f'PID {os.getpid()} encountered SQLALCH OperationalError(sqlite3.OperatioalError. sleeping')
                print(e)
                db.session.rollback()
                time.sleep(random.randint(1, 3))
```

The `__init__.py` and `proj/helpers.py` files did not require any modifications, because I can encapsulated all the database logic within the `proj/db.py` class and procedures.



Circular import problem


like:
* Todd Birchard's [Building a Python App in Flask](https://hackersandslackers.com/series/build-flask-apps/)
* Miguel Grinberg's [Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)

### Choosing Your Flask Application Model
Recognizing that I needed to do things better in order for this project to be of any use, I decided to go back to basics and assumed the most natural place to start was the official Flask tutorial series. As I followed the steps, however, I constantly questioning WHY the application was being built in a manner that felt either unexplained, or convoluted and unnatural. 

It was only when I began searching for alternative sources that I found Todd Birchard's excellent Flask series on [www.hackersandslackers.com](https://www.hackersandslackers.com). The writing style was entertaining, but more importantly I found that his explanations made *sense* and addressed many of the design questions I was struggling with in previous tutorials. If you are serious about learning Flask, I suggest you stop reading this and go read/implement his series first because I draw heavily upon his work.
