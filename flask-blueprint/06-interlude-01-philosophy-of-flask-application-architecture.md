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
from flask import Flask

app = Flask(__name__)


@app.route('/', methods=['GET'])
def hello_world():
    return "Hello world!"
 
 
app.run()
```
If the tutorial is especially bold, it will also include a basic database connection as well:
```python
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

### Choosing Your Flask Application Invocation Model
Not gonna lie, i found this WAAAAY more challenging than it had any right to be. 

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

To keep my life simple over the short term, I opted to:
1. Add `export FLASK_ENV=development` to my WSL2 instances's `~/.profile`
1. Modify the Makefile run command to use `flask run`
1. Add `app.run()` to the `__main__` dunder of my `wsgi.py` file.

TO DOS: 
(1) FIGURE OUT IF THE DEVELOPMENT DOCKER SHOULD BE BUILT WTIH A PROPER APPLICATION SERVER OR USE THE BUILT-IN WEB SERVER.
(2) CONFIRM THAT SETTING development AS THE FLASK_ENV VALUE IN ~/.profile DOES NOT IMPACT INVOCATION OF TESTING INSTANCE.

Don't worry if you are a little confused as to the exact position of these files in the project hierarchy, I'll be sure to call these changes out again when we get to the 
actual code.


### Choosing Your Database Integration Model





Defining a proper design is crucial for me purposes.
Circular import problem

    
<br><br>
Previous: [Install and Configure VS Code](./05-install-and-configure-vscode.md)<br>
Next:
