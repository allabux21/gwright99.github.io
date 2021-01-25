## Testing Setup

I had finally managed to get my rudimentary application working again with the newly-implemented ORM classes. I immediately encountered another problem, however, as the verification and bug fixing of my "plumbing" code required me to repeatedly manually reload, UI click, and data enter. I realized I needed some basic automated testing to make this chore less onerous.

Automated testing seems to have a mixed reputation based on the articles and forum comments I've seen. Everyone agrees that some automated testing is essential in order to ensure your application isn't a giant dumpster fire, but the problem seems to be defining the right balance between the extra work required to create and maintain the test cases versus the value their results provide. 

I personally experienced this frustration in a previous project where it felt like any change I made to my application source code required at least three times as long to implement the requisite updates in the assocated test cases. I didn't mind doing the updates the first few times but I started to become enraged over time as I felt I was spending all my time updatign the stupid test cases rather than finishing off the functionality of the service (which is what I really cared about).

So ... I'm going to approach this topic with an open mind and see where we end up, but can definitely say there's no way in hell I'll be following a Test-Driven Design model. At the very least, I needed some automated tests to quickly ascertain that the Flask Application could actually instantiate and load test data into the database (a code failure I had already experienced far too many times as I was making changes).

### Implementation Considerations

I needed to be mindful of a few items when deciding how I would approach the implementation:
* My application used the Flask Application Factory model (_which would impact how I executed instantiation in the test cases_)
* I was using SQlite right now but likely would use a different database later (_I could shortcut my database tests with an in-memory SQLite db but that option might not be avaiable later with its replacement).
* I was undecided whether I wanted to leverage the `Flask-Testing` module (which would simply life for me) versus directly using `PyTest` (which would require more work and problem solving but also give me a deeper understanding of the underlying mechanics).
* I needed to be mindful that my testing solution worked with the Make file and CI/CD logic I had already built based on the Martin Heinz article that I was modifying. These were based on vanilla Pytest so it could limit my ability to leverage another testing package like Flask-Testing.
* My Application and database models were still somewhat fluid, so I needed to make sure the testcases were not too tightly coupled to the existing state of the code (but clearly needed some level of adherence or the entire point of testing was moot).

### Testing Refresher
Before I began coding, however, I needed a refresher because I *TOTALLY* didn't remember how I set any of this stuff up before. I consulted Google and began filtering the results to find something that was relatively close to my set up and written in a tolerable style. Patrick Kennedy's [Testing Flask Applications with Pytest](https://testdriven.io/blog/flask-pytest/) fit the bill:
* It was new-ish (October 2020)
* It used Pytest (and deliberately avoided other libraries built on top, for demonstration purposes).
* It used a folder structure where the tests were in a peer folder to the main application.
* It used the Flask Appication Factory Pattern.
* It covered Fixtures.
* It had some suggestions for documentation (and acknowledged that test code quality is normally crap)

My first task was to remind myself what my existing `conftest.py` file logic was doing. It looked like:
```python
# conftest.py
# All custom fixtures should reside here - this is automatically discovered by Pytest. No import needed.
import logging
import os
import sys
import pytest
from dotenv import load_dotenv

sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))

from blueprint import create_app                    # noqa: E402 #pylint: disable=wrong-import-position
from blueprint.db.database import init_db           # noqa: E402 #pylint: disable=wrong-import-position

load_dotenv('.flaskenv-test', override=True)  # Set FLASK_ENV

# with open(os.path.join(os.path.dirname(__file__), 'data.sql'), 'rb') as f:
#    _data_sql = f.read().decode('utf8')

LOGGER = logging.getLogger(__name__)


@pytest.fixture(name='test_app', scope='function')
def fixture_test_app():
    # Main function has been build to set its config based on the FLASK_ENV
    # This is set above by load_dotenv, so I should be able to reuse the
    # function, and then just tack on the db drop/create.
    app = create_app()

    # Clear everything in the testdb and create
    init_db()

    return app

# tutorial has
# yield app
# os.close
# os.unlink(db_path)


@pytest.fixture(name="client")
def fixture_client(test_app):
    return test_app.test_client()


# Simple fixture. Logs message, allows test to excute, then logs final message.
# @pytest.fixture(scope='function')
# def example_fixture():
#    LOGGER.info("Setting up Example Fixture...")
#    yield
#    LOGGER.info("Tearing Down Example Fixture...")

```
Five major things happened when I reread this code:
1. I had a panicky flashback to the struggle required to get VSCode to detect my tests.
2. I had a vague recollection of the struggle to sort out the relationship between FLASK_ENV and other settings (_which I think I documented in an earlier post_).
3. I remember that I meant to figure out the whole `test_app.test_client()` use case.
4. I uttered a small curse when I saw that this code was based on my old database implementation pattern and would need to be updated. Dammit.
5. July 2020 Graham was a complete jerk for not leaving any useful comments.

Let's start unpacking.

#### Getting VSCode To Detect My Tests
I wanted my tests in a separate folder that was a peer to the application code, rather than nestled within it. Manipulating the folder strucutre was easy, but it quickly became a nightmare when it came to:
1. Getting VSCode to discover the tests
2. Defining the proper import paths for the test cases to execute successfully. 

It's all a little blurry now (thank god) but I remember struggling for days on this. The solution that eventually worked was jamming `sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))` near the top of each test file. This inserted the parent directory of the `tests` folder into my system path, and meant I could use the exact same import notation (e.g. `from blueprint.db.database import Base`) that I used in my application code. 

I wasn't thrilled about having to add this shim in each test file, but it seemed like the only solution that could adequately meet the needs of two different solution components:
1. Configuring VSCode testing required the definition of a root to begin from. Because I have my application code and tests as peers, I had to pick one or the other.
2. Meanwhile, the Make file testing mechanism directly called Pytest and pointed to the test module. 

Using relative import paths not only felt unclean (too many "../../" and "./"), it also always seemed to break either the VSCode side or the Make Pytest side. Once I tried the `sys.path.insert` solution, everything worked and I decided the kludge was 100% acceptable.

#### Flask Environment Definition
TO DO: Fill this in later.

#### Fixtures
As a reminder, fixtures are functions which can be configured to run at various points in the Setup/Teardown parts of a classic test case. As per Kennedy' article, fixtures had the following scopes:
* `function` - run once per test function (default scope)
* `class` - run once per test class
* `module` - run once per test file
* `session` - run once per session

I suspect that I would use the module scope to create common database objects (that are used by multiple test cases, e.g. User) rather than having to recreate the data every time, and I might use the function scope to invoke the database context manager. I continued with the article to see if I was right.

The next paragraph confirmed my suspicions. Modifying his code to reflect my own project's structure, we might go about confirming that our User object is behaving properly as follows:
```python
# tests/conftest.py
import pytest
from blueprint.models.User import User

@pytest.fixture(scope='module')
def new_user():
    user = User(username="tester", email="tester@gmail.com", password="testerpassword")
    return user

```

```python
# test/unit/test_models.py
import pytest

def test_new_user(new_user):  # Use the `new_user` fixture to supply this testcase with the fixture's `user` object.
    """
    GIVEN a User model
    WHEN a new User is created
    THEN check the username, email, and hashed_password
    """
    assert new_user.username == ' 'tester'
    assert new_user.email == 'tester@gmail.com'
    assert new_user.password != 'testerpassword'
    
```
I liked how Kennedy included a short but clear docstring in his testcases and have decided I will include the same in my own. Given the previous post on my struggles with getting docstring 'right', I'll remain mindful of how I try to balance clarity with terseness.

##### Special Fixture For Testing Views: test_client()
While testing objects is easy, we need a different way to test that the routes are working. Thankfully Flask has solved this problem via the `test_client()` method. This can be used to invoke calls to our endpoints and examine the response data. Returning to Kennedy's article, we can access the test_client like so:
```python
# test/conftest.py

import pytest
from blueprint import create_app

@pytest.fixture(scope='module')
def test_client():
    flask_app = create_app('flask_test.cfg')
    
    with flask_app.test_client() as testing_client:
        with flask_app.app_context():
            yield testing_client
            
```
```python
# tests/functional/test_endpoints.py
import pytest


def test_home_page_with_get(test_client):
    """
    GIVEN a Flask application configured for testing
    WHEN the '/' page is requested via GET
    THEN check that the response is valid
    """
    response = test_client.get("/")
    assert response.status_code == 200
    assert b"<SOME_TEXT_THAT_SHOULD_BE_IN_RESPONSE>" in response.data
    
    
def test_home_page_with_post(test_client):
    """
    GIVEN a Flask application configured for testing
    WHEN the '/' page is requested via POST
    THEN check that a '405' status code is returned
    """
    response = test_client('/')
    assert response.status_code == 405
    assert b"<SOME_TEXT_THAT_SHOULD_NOT_BE_IN_RESPONSE>" not in response.data
    
```

### Remediating My Existing Code
During the original July 2020 development stint, I had started working on getting a testing framework implemented. My `conftest.py` looked like this before I stopped:

```python
# conftest.py
# All custom fixtures should reside here - this is automatically discovered by Pytest. No import needed.
import logging
import os
import pytest
import sys

from dotenv import load_dotenv

sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))

from blueprint import create_app                    # noqa: E402 #pylint: disable=wrong-import-position
from blueprint.db.database import init_db           # noqa: E402 #pylint: disable=wrong-import-position

load_dotenv('.flaskenv-test', override=True)  # Set FLASK_ENV

# with open(os.path.join(os.path.dirname(__file__), 'data.sql'), 'rb') as f:
#    _data_sql = f.read().decode('utf8')

LOGGER = logging.getLogger(__name__)


@pytest.fixture(name='test_app', scope='function')
def fixture_test_app():
    # Main function has been build to set its config based on the FLASK_ENV
    # This is set above by load_dotenv, so I should be able to reuse the
    # function, and then just tack on the db drop/create.
    app = create_app()

    # Clear everything in the testdb and create
    init_db()

    return app

# tutorial has
# yield app
# os.close
# os.unlink(db_path)


@pytest.fixture(name="client")
def fixture_client(test_app):
    return test_app.test_client()


# Simple fixture. Logs message, allows test to excute, then logs final message.
# @pytest.fixture(scope='function')
# def example_fixture():
#    LOGGER.info("Setting up Example Fixture...")
#    yield
#    LOGGER.info("Tearing Down Example Fixture...")

```

Before deciding what to fix, let's take stock of the AS IS state:
* I needed to add the `system.path.insert` shim to successfully import my project-specific modules. This line had to be executed before the import statements.
* Unfortunately, the placement of the shim caused my linters to throw an import exception. This exception prevented the Make file test automation from successfully completing. 
* I didn't want to silence the linter import errors throughout the IDE, so I locally silenced it with the `# noqa: E402 #pylint disable=wrong-import-position` inline comment. 
* I used the `dotenv` python library to load environment variables that the Flask application needed, like `FLASK_APP` and `FLASK_ENV`.
* I was trying to do ... something ... with a `data.sql` file. I dont remember the exact details but assume I was trying to load some startup database records.
* I created a primitive logger object that I never subsequently used, other than for a commented fixture that demonstrated a simple log-yield-log pattern.

Onto the pre-existing fixtures:
* The core fixture `def fixture_test_app()` had two items of note: (1) the alias 'test_app' was defined in the fixture decorator (and used by downstream fixtures); (2) the function contained an `init_db()` step - something which was required based on the previous database integration pattern I used but no longer relevant now that I had implemented a newer, more decoupled pattern.
* The `def fixture_client(test_app)` was pretty straight forward: get a properly configured Flask app from the base fixture and then return a `test_client` for that app for API behaviour-testing purposes.

#### Onto the Remediation
The import shim and dotenv logic could stay. Everything else needed to change, and some of these changes meant I probably needed to change the project's `__init__.py` file too. I decided to start with the base fixture and follow various modification paths as they revealed themselves.

##### Fixing the base fixture
I identified six things that needed to change in the pre-existing base fixture:
1. The decorator scope should change from `function` to `module`<br>The Flask application only needed to be created and configured once for the entire testing cycle.
1. The decorator name should be removed<br>Rather than simplyfying my code, I found the alias made it more challenging to trace calls by downstream fixtures. I would remove the alias and directly refer to the function name itself.
1. The function name shoud be prefixed with `fixture_`<br>I didn't want a repeat of struggle I had keeping track of ORM attributes and relationship attributes, so i decided to prefix all my fixture functions with a hintword.
1. I needed to change the database integration model<br>The old integration wasn't going to work anymore. I needed to implement the new model and make sure this worked for my dev, testing, and production instances.
1. The two existing fixtures should be streamlined into one<br>I felt there was no justifiable reason for separating the Flask app creation from the test_client creation. I would combine the two and simplify my code at the same time.
1. I need to rough in logging capabilities<br>I was primarily using `print` statements to debug right now. A better solution would be to create logger that would write to file AND stdout.

The first three changes were easy, but the database change required a bit more thought. As it turned out, it required a very deep dive into my configuration set up and ... I had to change the damn database logic again.

##### Join Me For A Convoluted Journey
This whole thing was sparked by the presence of `load_dotenv('.flaskenv-test', override=True)` in my `conftest.py` file. I mentioned that my 'tests' folder was a peer to the 'blueprint' folder (where all the application code is), but I had neglected to mention that I had several other files that were peers to 'test' and 'blueprint'. Amongst them were:
* .flaskenv
* .flaskenv-test
* .flaskenv-prod
* dev.Dockerfile
* prod.Dockerfile

The Dockerfiles were important to my remote deployment plans but not relevant over the short term due to my decision to develop directly within WSL2 rather than on Docker/Kubernetes. I thought of them as a quasi-silver bullets: any configuration item I couldn't get working on my local machine/Make implementation could be easily solved later due to my ability to extensively set environment variables and modify/replace source code copied into the container. I relegated them to a "do later when you are getting closer to do fully-fledged remote deployments" and more to my more pressing problem: the flaskenv files.

The `.flaskenv*` files were structurally the same, and had only minor content differences; each file contained two key-values pairs: FLASK_APP and FLASK_ENV. The FLASK_APP was alwasy 'wsgi.py', whereas the FLASK_ENV varied (.flaskenv had 'development', .flaskenv-test had 'testing', and .flaskenv-prod had 'production'). There was nothing wrong with defining the environment variables like this, the problem was that I was loading everything differently:
* The development instance was setting FLASK_ENV and FLASK_APP via two different methods:
** Via my local ~/.profile
** Via a top-level export in the Makefile I created as part of Martin Heinz automation project setup.
* The testing instance was setting FLASK_ENV and FLASK_APP via `.load_dotenv('.flaskenv-test', override=True)`.
* The production instance did not have any method of setting the FLASK_ENV and FLASK_APP values defined in `.flaskenv-prod`. This was primarily due to:
** I had assumed I would not test the production configuration in a non-production environment.
** Because of my first assumption, I had not built my code in a way to deliberately instantiate a production instance.
** I assumed any instantiation of a production instance (in Production) would occur via the prod.Dockerfile (where I easily could set environment variables with ENV)
** I had given up trying to modify the Make file by inserting a 'production' target that could dynamically modify environment variables.

In addition to all the files I had spawned to set these two environment variables, I had OTHER configuration files in the `blueprint.config` folder, which handled the defintion of Flask application configuration values and sensitive data like SECRET_KEY and database connection strings.

This configuration rat nest was a becoming a big problem because it was directly limiting my ability to build a more flexible database context manager that could:
* Juggle three different sets of database connection strings: those that served development, testing, and production.
* Support multiple databases concurrently (e.g. instantiate a sqlite context manager as well as a separate Mongodb instance). I didn't know if I really needed this, but it seemed important to get working.

The FLASK_ENV environment variable seemed like the perfect environment identification because: 
** It was always populated
** Its potential values were perfect (development, testing, production)
** It was easily accessed with `os.environ`

I could make the non-streamlined dev and testing configuration work, but didn't know how I was going to be able to test the production instance locally without having to remember to manually set FLASK_ENV on the CLI every time I wanted to test. I gathered my courage and returned to the Make file.

##### Ugh Make Files
During my previous effort, I had tried to avoid modifying the Make file because I found the tool to be finicky and majority of the help posts to be less than clear. I couldn't think of another (non-Docker) way to standardize the FLASK_ENV definition though, so I dove back in. 

The 'as is' file had the following logic for the development and testing instances. 
```make
BLUE='\033[1;34m'
TAG := $(shell git describe --tags --always --dirty)
...
# Setting default values to development.
export FLASK_ENV = development
export FLASK_APP := wsgi

run:
	@flask run
    
test: 
	@pytest
	@echo "\nDirty git tag: $(TAG)\n"
....
```

I was ok with the Make file using development as the default FLASK_ENV but needed a way to overwrite this (preferrably in the Make file itself so I didn't need to continue the `dotenv` solution I had in the test fixture. Google searching ensued and I eventually found a few articles that helped me puzzle out what to do. [This Stack Overflow](https://stackoverflow.com/questions/35948154/how-to-set-environment-variables-in-makefile) post was particularly useful because it helped me understand why my previous attempts had failed:
* The Make file is invoked in one process, but spawns a new child process for each line of a recipe. Using the code above as an example:
** Process A is spawned when I type `make test` on the command line.
** Process B is spawned to execute the first step of the recipe (`@pytest`)
** Process C is spawned to execute the second step of the recipe (`@echo ....`)
* In my previous efforts, I did not realize these lines were atomic and was defining FLASK_ENV within the recipe in a way that guaranteed it would be overwritten by the parent thread's value.

The Stack Overflow article provided a solution: chain the lines together. Several troubleshooting hours later I landed on a final state for my Makefile entries:
```make
...
# Setting default values to development.
export FLASK_ENV = development
export FLASK_APP := wsgi

.DEFAULT_GOAL := run

run:
	@flask run

test: 
	@export FLASK_ENV=testing; pytest

production: 
	export FLASK_ENV=production; flask run
...
```
I didn't particularly like hardcoding the FLASK_ENV in here, but I couldn't figure out a better way to do it and was tired of fighting the code.

#### The Dreaded PyTest ImportError Returns
While verifying that the Make file's 'test' command worked, I noticed something weird - somehow the FLASK_ENV was being set to 'testing' automagically. This had me puzzled so I investigated, being with the test_client fixture which serves as the base of my pytest setup.

Print statements didn't seem to want to appear on stdout when I invoked pytest, so I used a rudimentary logger `LOGGER = logging.getLogger(__name__)` that I created in conftest.py. I inserted several environment variable checks in the function that creates the test_client(), thinking that I'd see the point where 'development' changed to 'testing'. The value was consistently 'developent', however, so this let me to believe that the value was somehow changing in the test_case that relied upon the fixture as a pre-requisite.

Print statements didn't work in this test case either, so I needed a Logger. I didn't want to create a new logger in the test_case file, so I tried to import the Logger that I had created in conftest.py and ... set off several hours worth of rage, frustration, and a terrible flashback to trying to fix this bloody problem six months before. ANGER!!!!!!

Let me tell you how I fixed it first, before getting into the unclear reasons for why it failed:
My initial folder setup looked like
```tree
+-- blueprint
|   +-- (various files)
+-- tests
|   +-- conftest.py
|   +-- (various testcases)
```








<TO DO: Project test strucutre>

Next: TBD
Previous: [Docstrings and Naming Conventions](./10-docstrings.md)<br>
