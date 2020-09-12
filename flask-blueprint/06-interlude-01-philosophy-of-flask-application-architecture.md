## Interlude 01 - The Philosophy of Flask Tutorials
So we've picked our technology components and configured our tooling - time to get coding, right!? Not quite.

I specifically chose Flask over Django because:

1. Flask is an unopinionated framework that grants me a greater degree of control over my implementation, and <br>

1. Flask's design makes it very easy to stand up a microservice with just a few lines of code in a single `app.py` file.

These are amazing benefits which work very well with Rapid Application Development (the development methodology that I favour). Being able to move quickly and deliver results promptly builds trust with your stakeholders and helps ensure you are actually building the solution that is needed rather than risk encountering a horrible surprise close to the end of a project.

This flexibility and ease-of-entry, however, come at a great cost: the internet abounds with reams of badly-designed Flask tutorials which work for the simple needs of the tutorial article/series but cannot hope to scale appropriately should the developer ever want to move beyond a small-scale proof-of-concept stage. Many of the simplest Flask tutorials I've encountered look something like this:

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

This makes perfect sense for teaching a beginner: it requires minimal code, will work immediately, and encourages the reader to continue their learning journey. Unfortunately, it also begins engraining [bad development habits](https://hackersandslackers.com/flask-application-factory/) from the outset, which the reader will need to unlearn later when/if they are exposed to more fulsome tutorials.

I was/am equally guilty of these bad habits. Roughly a year ago, I built a Flask application for a job-related POC, and the end result was a single enormous `app.py` file. Granted, I built it quickly and proved my point, but the way the application was architected meant it could have never gone into Production without a complete overhaul (_thankfully my employer is a Java shop, so there was never any question that we would take the next bazillion years to rewrite it in a 'proper' language /s).  

### Great, another noob - why should I listen to you?
_"Way to be a hypocrite, Graham!"_ you may be saying. I've never deployed a Flask application into Production and never had to keep application infrastructure running under heavy load; so what the hell do I know? 

First, I would say "_Dude, it's a free blog that I'm mostly writing for myself - just stop reading if you haven't found the content useful so far!_".

For anyone remaining after the first response, I say this: I don't have all the answers. I will never claim to have all the answers. But I am motivated to build out my technical portfolio, and to do that I need to have a resuseable project nucleus that enables me quickly spin up future projects that I trust to be scaleable. I'll share my opinions, provide links and background to the material I reviewed that underpin my conclusions, and let you decide for yourself. Eventually, once I become more proficient at this blogging platform, I may even throw caution to the wind and enable anonymous internet strangers to brutally comment on all the mistakes and design decisions I've made (_but maybe not that soon!_). 

### Fine, I'll bite - shower us with your opinion
IMHO, three major application design decisions needed to be made at the start of the project due to their impact on all the subsequent work:

1. How to initialize the Flask application object
1. How to structure your project
1. How to structure the database connection

To avoid things from getting overly long, I'll cover each item in its own separate page.


<br><br>
Previous: [Install and Configure VS Code](./05-install-and-configure-vscode.md)<br>
Next: [Flask Development Server Invocation Model](./07-flask-dev-server-invocation-model.md)
