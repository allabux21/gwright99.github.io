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
I wanted y tests in a separate folder that was a peer to the application code, rather than nestled within it. Manipulating the folder strucutre was easy, but it quickly became a nightmare when it came to:
1. Getting VSCode to discover the tests
2. Defining the proper import paths for the test cases to execute successfully. 

It's all a little blurry now (thank god) but I remember struggling for days on this. The solution that eventually worked was jamming `sys.path.insert(0, os.path.abspath(os.path.join(os.path.dirname(__file__), '..')))` near the top of each test file. This inserted the parent directory of the `tests` folder into my system path, and meant I could use the exact same import notation (e.g. `from blueprint.db.database import Base`) that I used in my application code. 

I wasn't thrilled about having to add this shim in each test file, but it seemed like the only solution that could adequately meet the needs of two different solution components:
1. Configuring VSCode testing required the definition of a root to begin from. Because I have my application code and tests as peers, I had to pick one or the other.
2. Meanwhile, the Make file testing mechanism directly called Pytest and pointed to the test module. 

Using relative import paths not only felt unclean (too many "../../" and "./"), it also always seemed to break either the VSCode side or the Make Pytest side. Once I tried the `sys.path.insert` solution, everything worked and I decided the kludge was 100% acceptable.

#### Flask Environment Definition
TO DO: Fill this in later.

#### Fixtures (specifically the test_client fixture)


Thankfully - although I had completely forgotten how I managed it before - I had actually implemented some testing functionality during my previous development run.




<TO DO: Project test strucutre>

Next: TBD
Previous: [Docstrings and Naming Conventions](./10-docstrings.md)<br>
