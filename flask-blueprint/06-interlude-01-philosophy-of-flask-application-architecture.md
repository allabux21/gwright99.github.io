## Interlude 01 - The Philosophy of Flask Application Architecture
So we've picked our technology components and configured our tooling - time to get coding, right!? Not quite.

I specifically chose Flask over Django because: <br>
1. Flask is an unopinionated framework that grants me a greater degree of control over my implementation, and <br>
1. Flask's design makes it very easy to stand up a microservice with just a few lines of code in a single `.py` file.

These are amazing features which work very well with Rapid Application Development, an application development methodology that I favour. Being able to move quickly and delivery results promptly builds trust with your stakeholders and helps ensure you are actually building the solution that is needed rather than risk encountering a horrible surprise close to the end of a project.

This flexibility and ease-of-entry, however, come at a great cost: the internet abounds with reams of badly-designed Flask tutorials which work for the simple needs of the tutorial article/series but cannot hope to scale appropriately should the developer ever want to move beyond a small-scale proof-of-concept stage. Given how easy it is to write a 'me-too' Introduction to Flask article in the pursuit of eyeballs and clicks, the proliferation of junk examples is not the least bit surprising due to Python's immense growth in popularity [ADD SOURCE].

_"Way to be a hypocrite, Graham!"_ you may be saying. I'm writing my own series on Flask, have never deployed a Flask application into Production, and never had to keep application infrastructure running under heavy load; so what the hell do I know? 

To that I say "_Dude, it's a free blog that I'm mostly writing for myself - if you dont like the ten minutes or so of content, just stop reading!_".

I will be the first to admit that I don't have all the answers (to be fair, I dont think _anybody_ has all the answers). With that said, the output of this project is meant to help me quickly and easily spin up future projects, so I'm well-motivated to build something that is cohesive, scaleable, and adheres to defensible best practices. I'll share my opinions, provide links and background to the material I reviewed that led me to my conclusion, and let you decide for yourself. Eventually, once I figure out how to use GitHub Pages, I may even throw caution to the wind and enable  anonymous internet strangers to brutally comment on all the mistakes and philosophical decisions I've made (_but maybe not that soon!_). 

With that said, onto my opinions ...

### Choosing Your Flask Application Model
Many of the simplest Flask tutorials I've encountered look something like this:
```python
# app.py

from flask import Flask

app = Flask(__name__)


@app.route('/', methods=['GET'])
def hello_world():
    return "Hello world!"
 
 
app.run()
```
If the tutorial is especially bold, it will also include a basic database connection as well:
```python
# app.py

from flask import Flask
import sqlite3

app = Flask(__name__)


@app.route('/', methods=['GET'])
def hello_world():
    return "Hello world!"


def connect_to_db():
    conn = sqlite3.connect('somedb.db')
    cur = conn.cursor()
    db_results = cur.execute('SELECT * FROM some_table;').fetchall()


app.run()
```

This makes perfect sense for teaching a beginner: it requires minimal code, will work immediately, and encourages the reader to continue their learning journey. 

Unfortunately, it also begins engraining [bad development habits](https://hackersandslackers.com/flask-application-factory/) immediately, which the reader will need to unlearn later as they are exposed to more fulsome tutorials like:
* Todd Birchard's [Building a Python App in Flask](https://hackersandslackers.com/series/build-flask-apps/)
* Miguel Grinberg's [Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)
* The official [Flask Tutorial](https://flask.palletsprojects.com/en/1.1.x/tutorial/) (_which i still have reservations about, more on this later in the database commentary_)

I began my own learning journey with the official Flask tutorial series. I followed the steps but was constantly questioning WHY the application was being built in a manner that felt convoluted and unnatural. This prompted a search for dissenting opinions, whereupon I found Todd Birchard's excellent Flask series on [www.hackersandslackers.com](https://www.hackersandslackers.com). I found Birchard's writing style entertaining, but more importantly I found that his explanations made sense and addressed many of the design questions I was struggling with in previous tutorials. If you are serious about learning Flask, I suggest you stop reading this and go read/implement his series first because I draw heavily upon his work.

#### Decision: Use the Flask Application Factory Model
The **Application Factory Model** was of particular interest to me because it offered:
* A clean entrypoint for WSGI-based application servers
* A way to decouple application configuration from application instantiation (configuration values are passed in as parameters at runtime, thereby facilitating testing).
* A modular structure that mostly resolves the [circular import problem](https://itnext.io/flask-factory-pattern-to-setup-your-project-8fe7d6b23247) 

These capabilities meant I could:
1. Implement Flask Blueprints to logically separate different functional areas.
2. Build the application knowing that I could eventually drop-in an industrial-grade application server when I was ready to go to Production.
3. Facilitate the integration of a testing framework.

Plenty has been written about the Flask Application Factory Model so I will not spend more time parroting ideas already written more clearly by others. However, it is crucial to point out that the decision to implement this model then sparked two other design problems:

1. How to invoke the Flask application via the `wsgi.py` entrypoint
2. How to supply the Flask application with a database access solution

### Choosing Your Flask Development Server Invocation Model
Not gonna lie, i found this more challenging than it had any right to be. 

Before you continue, go read [How to Run a Flask Application](https://www.twilio.com/blog/how-run-flask-application) by Miguel Grinberg. It succinctly describes how the invocation of the Flask development web server has changed over time (`app.run()` vs. `flask run`), and provides the foundation to decide our own preferred invocation model. (The Production deployemnt model remains unchanged, with a dedicated application server [directly calling the Flask Application Factory](https://flask.palletsprojects.com/en/1.1.x/tutorial/deploy/)).

TLDR:
1. `flask run` is the newest invocation method and is recommended by the Flask project (_not surprising, they obviously created this functionality for reason_).
<br><br>This method offers multiple ways to identify the target application and gives fine-grain control over your development server behaviour. It is essential to note, however, that you CANNOT control debug mode via this command - you must do it via the FLASK_ENV environment variable (e.g. `export FLASK_ENV=development`). 
<br><br>This is also where we start to see knock-on effects: certain `flask run` options like `--reload` and `--debugger` draw their default values based on whether debug is enabled (with debug itself enabled via `FLASK_ENV=development`). To add to the fun, other settings like `--eager-loading` draw their default values from the `--reload` value.
<br><br>Confused yet? Yep, I was too.
 
1. `app.run()` is less robust when it comes to reloading and has no CLI, but you also avoid spaghetti dependencies.

At the end of the article, Grinberg gives us an easy out: just use both! 
* Set your environment variable `export FLASK_APP=main_application_file.py`, and
* Make sure your main application file contains: 
```python
if __name__ == "__main__":
    app.run()
```

Seems pretty straightforward, so why am I claiming it's actually more complicated than this? The challenge is less about which invocation command to use and more about how to properly account for the decision when integrating it into other areas of the solution that we will be building, including:
* `__init__.py` verification logic
* dotenv configuration logic
* Dockerfile command definition and sequencing (both development & production instances)
* Makefile command invocation 

#### Decision: Use flask run AND app.run()
To keep my life simple over the short term, I opted to:
1. Add `export FLASK_ENV=development` to my WSL2 instances's `~/.profile`
1. Modify the Makefile run command to use `flask run`
1. Add `app.run()` to the `__main__` dunder of my `wsgi.py` file.

TO DOS: 
(1) FIGURE OUT IF THE DEVELOPMENT DOCKER SHOULD BE BUILT WTIH A PROPER APPLICATION SERVER OR USE THE BUILT-IN WEB SERVER.
(2) CONFIRM THAT SETTING development AS THE FLASK_ENV VALUE IN ~/.profile DOES NOT IMPACT INVOCATION OF TESTING INSTANCE.

Don't worry if you aren't totally clear on the exact position of these files in the project hierarchy, I'll be sure to call these changes out again when we get to the 
actual code.


### Choosing Your Database Integration Model
I'll just come out and say it: I really don't like using an ORM.

I understand that using an ORM is supposed to make my life easier by abstracting away the mechanics of interacting with a database and allowing me to retrieve/manipulate records using Pythonic syntax. I understand that an application that uses an ORM can be more easily forklifted to a different database solution, requiring only minor changes to the database connection settings. I also understand that the ORM offers coding shortcuts to access record data (e.g. relationship definitions and `Object.query.filter_by()...` rather than `db.session.query(Object).filter_by()...`. But I don't find these helpful enough to compensate for the headspace I need to devote to keeping track of what the ORM is doing for me and how I'm supposed to use it.

#### Aside: Graham admits how little he knows about this topic (but will compensate for that with Truthiness)
Before I continue, I need to make it very clear that this is a topic I'm not 100% about. There are tons of opinions on the web supporting the pro- and anti- ORM cause. Both sides appear to be well-informed and can back up their cases with plenty of technical facts. The comment sections are then filled with anecdotes like _"sure you can get away without using an ORM when you are building a small project, but GOOD LUCK trying to maintain a multi-million line entreprise code base without one!"_, _"We built our application using an ORM but now the ORM isn't doing what we need it to do and have spent the last six months struggling to rip it out of our codebase"_, and _"Won't somebody PLEASE think of performance!!"_ (this one is used by both sides).

I don't know who the authors or commenting practitioners are. I don't know what solutions they built. I don't know how those who chose to use an ORM architected their integration. I don't know what solutions those who chose not to use an ORM used to avoid having to write bespoke SQL for every transaction. I don't know if it's realistic to expect to the availability of a complete, up-to-date Logical Data Model if one is brought in as a developer on a large legacy codebase. I don't know who or what to trust, and this makes it exceedingly hard to make an informed decision regarding my own project.

I CAN comment on my own experience trying to get an ORM (`flask-sqlalchemy` and the `SQLAlchemy`) integrated my own small project, and the thoughts I had while observing myself trying to get the initial solution working as well as wondering "How would I get this to scale if this was more than a one-man operation?".

With my digressing out of the way, onto the main points.

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

##### Flask development server initialization behaviour
NOTE: GW TO TRY THE @app.before_first_request decorator to see if this solves the problem.
Flask is built upon the Werkzeug library. When Flask is started in development mode, Werkzeug will spawn a child process in order to [restart the main process each time the application code changes](https://stackoverflow.com/questions/25504149/why-does-running-the-flask-dev-server-run-itself-twice). This behaviour, however, also expresses itself when the application is first initialized.

Part of my `__init__.py: create_app()` function tries to seed the SQLite database with records. Given the double-execution behaviour of the Flask development server, this meant two processes were trying to connect to the database (each trying to write the same records). I was quite capable at opening database connections via the Session() object but I had neglected to write any code that _closed_ the database connection. This meant that the second process received a process conflict error (TO DO: GO FIND THE EXACT ERRRO), but SQLite was still waiting for the first process to close its connection. Oops.





* If you observe the Flask development server logs on startup, the same code is executed twice. 



Defining a proper design is crucial for me purposes.
Circular import problem

    
<br><br>
Previous: [Install and Configure VS Code](./05-install-and-configure-vscode.md)<br>
Next:
