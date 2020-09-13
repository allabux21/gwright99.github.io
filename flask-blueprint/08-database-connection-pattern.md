## Database Connection Pattern
Choosing your database pattern is an early project decision that is critical to settle quickly. Making the wrong decision can result in costly and frustrating rework throughout the entire application should you ever need to change mid-stride. While I'm unable to comment on support issues that may come up in Production, I can comment on the process I had to go through to settle on the database connection pattern that I'll be using in the project

### Decision: Use the SQLAlchemy ORM ... Sort of
As per my [technology component decisions](./02-project-goals-and-design-considerations.md) I said I would be using the SQLAlchemy ORM. How I reached that decision was an evolutionary process based on development efforts I conducted before I began to write this material. 

To be honest, I'm still not convinced it even makes sense to use SQLAlchemy (_I'll explain why once I detail my steps_), but at this point in time I don't want to have to retrofit yet again so ... I'll stick with the ORM for now.

### Caveat: Take what I say with a grain of salt
Before I begin, I should make it clear that I'm not 100% in my mastery of this topic. I have tried to learn the SQLAlchemy ORM before and consistently found it to feel unnatural and confusing. With that said, I have not spent huge amounts of time reading through the large body of documentation and - when I did read portions of it - more often than not my reaction was "err, wut?". 

What I'm trying to say is: I *think* I don't like using an ORM.

I understand that using an ORM is supposed to facilitate life by abstracting the mechanics of database interaction and allowing me to stay in a Pythonic syntax world. I understand that an application using an ORM can be more easily forklifted to a different database solution, requiring only minor changes to the database connection settings. I also understand that the ORM offers coding shortcuts to access record data (e.g. relationship definitions and `Object.query.filter_by()...` rather than `db.session.query(Object).filter_by()...`. 

The problem is that I don't find these reasons to be good enough to counterbalance the headspace I must devote to keeping track of what the ORM is doing for me and how I'm supposed to use it.

There are tons of opinions on the web supporting the pro- and anti- ORM cause. Both sides appear to be well-informed and can back up their cases with plenty of technical facts. The comment sections are then filled with anecdotes like _"sure you can get away without using an ORM when you are building a small project, but GOOD LUCK trying to maintain a multi-million line entreprise code base without one!"_, _"We built our application using an ORM but now the ORM isn't doing what we need it to do and have spent the last six months struggling to rip it out of our codebase"_, and _"Won't somebody PLEASE think of performance!!"_ (this one is used by both sides).

I don't know who the authors or commenting practitioners are. I don't know what solutions they built. I don't know how those who chose to use an ORM architected their integration. I don't know what solutions those who chose not to use an ORM used to avoid having to write bespoke SQL for every transaction. I don't know if it's realistic to expect to the availability of a complete, up-to-date Logical Data Model if one is brought in as a developer on a large legacy codebase. I don't know who or what to trust, and this makes it exceedingly hard to make an informed decision regarding my own project.

I CAN comment on my own experience trying to get an ORM (`flask-sqlalchemy` and the `SQLAlchemy`) integrated my own small project, and the thoughts I had while I was struggling and cursing to get things working. 

I'll write more about my ORM thoughts in a separate post, partially because I want to focus on how I reached the Database Connection Pattern that I did but also because these efforts directly informed my ORM-use opinion.

### DB Connection Pattern #1: flask-sqlalchemy instantiated directly within __init__.py
I am a big fan of Todd Birchard's Flask article series on www.hackersandslackers.com. My first DB connection pattern attempt followed the example in his [Connect Flask to a Database with Flask-SQLAlchemy](https://hackersandslackers.com/flask-sqlalchemy-database-models/) article.

I've omitted a few lines for brevity, but it is basically:
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
The article provides a clear step-by-step breakdown of how the database is registered with the Flask application, so I won't rehash the details here. I'll take a moment, however, to reiterate the two most crucial items to remember:

1. The database connection is not available for use until it has been registered with the application (_at runtime_).
1. The database is not created until the first time the application is executed.

This design meets the immediate need of providing the application with stateful storage. Unfortunately, it tightly couples database instantiation with application instantiation and this precludes the reuse of our connection code should we wish to interact with the database independently of the application. 

I didn't like this and started looking for an alternative design.

### DB Connection Pattern #2: flask-sqlalchemy directly instantiated in a separate database file
Searching for a way to split the database instantiation from the application instantiation, I found a gem of an article in Bob Waycott's [Organizing Flask Models with Automatic Discovery](https://bobwaycott.com/blog/how-i-use-flask/organizing-flask-models-with-automatic-discovery/#sqlalchemy-database-setup) (I was actually looking for a way to define ORM table classes over multiple files, but more on that later).

Waycott creates the database connection object in a separate `db.py` file. He then provides a function that can be called by the Flask Application Factory in `__init__.py` that will bind the flask_sqlalchemy object to the Flask application. (_Note: I've omitted some of his code for brevity_):
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
Not quite. Waycott's solution gives us an independent entrypoint to the database, but we still have to register the flask_sqlalchemy plugin with the Flask application. This probably isn't terrible but I still didn't like it for a few reasons:

1. I had to write an (albeit small) function to tie the objects together.
1. It made me dependent on the flask-sqlalchemy package (which wraps the underlying SQLAlchemy library).
1. flask-sqlalchemy has a slightly different syntax for defining database objects than using SQLAlchemy directly.

Even though the SQLAlchemy documentation itself suggests using a helper library like flask-sqlalchemy !ADD REFERENCE!, something about the design felt off. Furthermore, whenever I searched for database syntax help, I always had to to check if the answer was written for SQLAlchemy or flask-sqlalchemy. After finding too many SQLAlchemy solutions and not enough flask-sqlalchemy answers, I assumed that it was better to use SQLAlchmey directly to maximize the chances of finding answers to the problems I encountered. This opinion was subsequently reinforced by the arguments made in Edward Krueger's [Use Flask and SQLAlchemy, not Flask-SQLAlchemy!](https://towardsdatascience.com/use-flask-and-sqlalchemy-not-flask-sqlalchemy-5a64fafe22a4?gi=6c73d7f74e07) article.

As a result, I decided to abandon flask-sqlalchemy and rework the code to use SQLAlchemy directly. 

### DB Connection Pattern #3: SQLAlchemy directly instantiated in a separate database file
Using Kreuger's example as inspiration, I refactored my database file by replacing the flask-sqlalchemy with the base SQLAlchemy. 

The new `db.py` file looked like:
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

This redesign also required changes in the `__init__.py` file too:
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
With the change implemented, the database logic was successfully split away from the Flask application logic. I was finally done with all this annoying refactoring work and could get back to writing code! Or so I thought ...

#### OMG why am I getting all these errors from the Flask development server?!
Earlier in this series, I devoted a full post to explaining [how and why I chose my technology components](./02-project-goals-and-design-considerations.md). When discussing the database solution, I noted that I opted to use SQLite despite its limitations regarding concurrent connections. 

This "limitation" began to cause trouble for me almost immediately (_admittedly due to my own programming stupidity_), and was due to two reasons:

1. The Flask development server initialization behaviour
2. Neglecting to close my database connections

##### What was causing the problem? (Hint: mostly me)
NOTE: GW TO TRY THE @app.before_first_request decorator to see if this solves the problem.
Flask is built upon the Werkzeug library. When Flask is started in development mode, Werkzeug will spawn a child process in order to [restart the main process each time the application code changes](https://stackoverflow.com/questions/25504149/why-does-running-the-flask-dev-server-run-itself-twice). This behaviour, however, also expresses itself when the application is first initialized.

Part of my `__init__.py: create_app()` function tries to seed the SQLite database with records. Given the double-execution behaviour of the Flask development server, this meant two processes were trying to connect to the database (each trying to write the same records). I was quite capable at opening database connections via the Session() object but I had neglected to write any code that **_closed_** the database connection. This meant that the second process received a process conflict error (TO DO: GO FIND THE EXACT ERROR), but SQLite was still waiting for the first process to close its connection. Oops.

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

### DB Connection Pattern #4: SQLAlchemy instantiated via a Context Manager (in a separate database file)
Implementing a database connection Context Manager required the database connection to move from the global scope of `proj/db.py` into a custom `SQLAlchemyDBConnection` object that had defined `__enter__` and `__exit__` dunders (thereby allowing me to use it as a context manager via the `with` keyword). I also modified the `commit_or_rollback()` procedure to use the `SQLAlchemyDBConnection` class. 

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

The `__init__.py` and `proj/helpers.py` files did not require any modifications, because I encapsulated all the database logic within the `proj/db.py` class and procedures.

### DB Connection Pattern #5: ???
So what Database Connection Pattern did I try next? Given my already-stated distaste for ORM usage, it would have probably made sense to try using a SQL querybuilder instead of a full-fledged ORM. With that said, I was tired of analysis paralysis that was causing continually refactoring of existing code rather than building the rest of the project. 

As a result, I made an executive decision to stick with the SQLALchemy Context Manager solution and get on with the real work.

Next: [What's the beef with ORMs](./09-ORM-beef.md)<br>
Previous: [Flask Instantiation & Invocation Patterns](./07-flask-instantiation-and-invocation.md)
