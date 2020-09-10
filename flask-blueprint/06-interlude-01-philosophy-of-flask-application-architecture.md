## Interlude 01 - The Philosophy of Flask Application Architecture
So we've picked our technology components and configured our tooling - time to get coding, right!? Not quite.

I specifically chose Flask over Django because: <br>
1. Flask is an unopinionated framework that grants me a greater degree of control over my implementation, and <br>
1. Flask's design makes it very easy to stand up a microservice with just a few lines of code in a single `.py` file.

These are amazing features which work very well with Rapid Application Development, an application development methodology that I favour. Being able to move quickly and delivery results promptly builds trust with your stakeholders and helps ensure you are actually building the solution that is needed rather than risk encountering a horrible surprise close to the end of a project.

This flexibility and ease-of-entry, however, come at a great cost: the internet abounds with reams of badly-designed Flask tutorials which work for the simple needs of the tutorial article/series but cannot hope to scale appropriately should the developer ever want to move beyond a small-scale proof-of-concept stage. Given how easy it is to write a 'me-too' Introduction to Flask article in the pursuit of eyeballs and clicks, the proliferation of junk examples is not the least bit surprising due to Python's immense growth in popularity [ADD SOURCE].

_"Way to be a hypocrite, Graham!"_ you may be saying. I'm writing my own series on Flask, have never deployed a Flask-based Python application into Production, and haven't had to keep the infrastructure running under heavy load; so what the hell do I know? To that I say "_Dude, it's a free blog that I'm mostly writing for myself - if you dont like the ten minutes or so of content, just stop reading!_".

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

I began my own learning journey with the official Flask tutorial series. I followed the steps but was constantly questioning WHY the application was being built in a manner that felt convoluted and unnatural. This prompted a search for dissenting opinions, whereupon I found Todd Birchard's excellent Flask series on [www.hackersandslackers.com](https://www.hackersandslackers.com). I found Birchard's writing style entertaining, but more importantly I found that his explanations __made sense__ and addressed many of the design questions I was struggling with in previous tutorials. If you are serious about learning Flask, I suggest you stop reading this and go read/implement his series first because I have drawn heavily upon his work to complete my own.




### Choosing Your Database Integration Model





Defining a proper design is crucial for me purposes.

    
<br><br>
Previous: [Install and Configure VS Code](./05-install-and-configure-vscode.md)<br>
Next:
