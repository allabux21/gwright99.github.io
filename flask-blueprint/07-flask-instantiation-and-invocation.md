## Flask Application Instantiation & Invocation Models

Choosing your Flask application's _instantiation_ & _invocation_ models is an essential first decision because it will have ramifications that ripple throughout the rest of the development project including:

* The project structure
* Makefile configuration
* Dockerfile configuration
* Dotenv configuration
* Flask application start-up verification logic
* Flask application plugin registration logic
* Flask application database integration logic

### Selected Models
The project will use the **Flask Application Factory Model** as its application instantiation model.
The project will use **flask run** as its development server invocation model.


#### Why use the Flask Application Factory Model?
The Flask Application Factory Model is of interest because it offers:

* A way to decouple application configuration from application instantiation (configuration values are passed in as parameters at runtime, thereby facilitating testing).
* A modular structure that resolves the circular import problem
* The capability to concurrently run multiple application instances.

The downside of this approach is that the application instantiation logic becomes a bit more convoluted to compensate for the fact that the `app` object will not exist until runtime. This represents a one-time design and implementation cost, thereby making it a completely acceptable cost in order to access the benefits it provides.

To learn more about the mechanisms underlying the Flask Factory Application Model:
* See the [offical Flask documentation](https://flask.palletsprojects.com/en/1.1.x/patterns/appfactories/) page for a basic explanation.
* See Todd Birchard's [Demystifying Flask's Application Factory](https://hackersandslackers.com/flask-application-factory/) article for a better (IMHO) explanation.
* See Felipe Florencio Garcia's [Flask Factory Pattern to set up your project](https://itnext.io/flask-factory-pattern-to-setup-your-project-8fe7d6b23247) article for deeper explanation of the circular import problem.


#### Why use the `flask run` Invocation Model?
Not gonna lie, i found this more challenging than it had any right to be. 

It is vital to remember that what we are deciding is how to invoke the inbuilt Werkzeug **DEVELOPMENT** web server. This decision does not have any impact on the Production deployment model (which calls for a dedicated application server to [directly call the Flask Application Factory](https://flask.palletsprojects.com/en/1.1.x/tutorial/deploy/)). I've not yet tried it, but I assume this would also apply to any non-Production instance where the Flask application code is built on a dedicated WSGI application server (something I may experiment with in the future given that the project has a Docker component).

Before you continue, go read Miguel Grinberg's [How to Run a Flask Application](https://www.twilio.com/blog/how-run-flask-application). It succinctly describes how the invocation of the Flask development web server has changed over time (`app.run()` vs. `flask run`), and provides the foundation upon which I decided my own preferred invocation model. 

As per Grinberg, it basically boils down this:

1. `flask run` is the newest invocation method and is recommended by the Flask project. It boasts several advanced capabilities, one of which is a method to invoke a Flask application factory pattern directly from the CLI.

1. `app.run()` is less robust when it comes to reloading and has no CLI, but is less complex to implement.

##### Seems simple enough, what's so challenging about it?
For reasons I don't fully understand, the `flask run` method relies upon a combination of CLI options AND environment variable values. The only way to activate Flask's debug mode is by setting `FLASK_ENV=development` in the environment. Options like `--reload` and `--debugger` draw their default values based on whether debug is enabled. Other settings like `--eager-loading` draw their default values from the `--reload` value. Furthermore, the `FLASK_APP` environment variable needed to be set to identify the actual application to invoke. Confused yet? Yep, I was too.

Despite Grinberg giving us an easy way out (just use both!), I declined because it felt hacky. The creation of the new method clearly indicated that the Flask team felt that the old way was insufficient, and I had a sneaking (unproven) suspicion that using both methods would eventually cause some bizarre bug. Given my desire to build a modern project, use the application factory pattern, and the fact that I would be Dockerizing the Production instance, I decided that the `flask run` method was more appropriate.

This decision, however, rippled to other components of the project that demanded a reckoning:
1. I had to ensure that my WSL2 environment had these values at all times.
1. I had to ensure that the Makefile automation component was not impacted.
1. I had to ensure that the Dockerfile components specified these environment variables and had the correct values.

It was easy to hardcode the environment variables in the WSL2 instance, by adding them to my profile:
```bash
# ~./profile
export FLASK_ENV=development
export FLASK_APP=wsgi.py
```

The downside of this action is that I now have to remember that I hardcoded them and that they will always reset to this value when the terminal is restarted. I determined this was acceptable, however, because: 

* My local environment is meant for development, during which I assume I will always want debug and auto-reloader capabilities. 
* I avoid process scope problems that I would encounter if I tried setting the variables as part of the Makefile-based invocation.
* Given that the project is meant to be reusable, any subsequent application should also use `wsgi.py` as its entrypoint.
* The Production instance of this application is generated via a Dockerfile, in which I can easily set the `FLASK_ENV=production`

The only thing I wasn't certain about was the impact to spinning up a testing instance.
* ?? TO DO: IMPLICATION TO TESTING INSTANCE?? TBD. 

 
<br><br>
Next: [Database Connection Pattern](./08-database-connection.md)
Previous: [Interlude 01 - The Philosophy of Flask Tutorials](./06-interlude-01-philosophy-of-flask-application-architecture.md)
