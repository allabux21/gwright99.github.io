## Why the Beef with ORMs?

I initially wrote this blog series in July 2020 and only picked it back up again at the end of December 2020. Part of the reason I stopped was to spend more time on work commitments (including the now-abandoned OpenShift4 Training blog series) but, honestly, part of it was also because I was growing tired of fighting the database (and trying to learn the SQLAlchemy ORM).

Just before I stopped, I listed the following four items as reasons I found myself disliking the use of an ORM:
1. Registering the ORM when using the Flask Application Factory (tight coupling)
2. Intermingling SQL abstractions with object relationships
3. Obfuscation of underlying database interactions
4. Learning the ORM rather than learning SQL

Within a day of restarting the project, I (mostly) had these nagging feelings again. 

Originally, it felt like I was fixating too strongly on this issue and I started to wonder if I was subconsciously using this as an excuse to not continue with the rest of the project. Instead, I could just refactor the database code section over and over again ad nauseaum until I gave up for a second time and go watch videogames on Youtube instead. 

However, my opinion began to change as I started reviewing the project code again. I was fixating on this problem because it was a __foundational__ decision that had rammifications across the design and architecture of the whole application: 

* My code was littered with commented blocks I was afraid to delete, reference links, confused explanations, notes to further investigate later, and other notes that said "I did investigate more and still dont understand what the documentation is trying to explain but I *think* it works this way."

* Instantiation and syntax patterns had repeatedly changed as I moved from Flask-SQLAlchemy to SQLAlchemy, and then needed to change again to accommodate the creation of a context manager for database connections (granted partially driven by the limitations of Sqlite to only access requests from one process at a time). 
Model.query
db.session

* I had to find, research, and eventually rewrite a series of functions (Bob Waycott) to be able to find and load multiple models spread over multiple files. While the search logic was reusable, the model creation was tied to the SQLAlchemy Base class, thereby requiring me to rewrite the database module to separate the logic the instantiates db object (Base, engine, ContenxtManager) from poplation functions (due to circular imports).  

* The population of test data began to cause me trouble. It was simple enough to populate records in an isolated table, but how was I supposed to populate the assocation tables? My Models had relationships attributes that were tied to Association tables in the database, but associations were a programmatic relationship that the ORM would turn into the requisite SQL behind the scenes. Was I cheating myself of the opportunity to truly understand how one is supposed to update a series of records in a database by using this confusing shortcut?


### The Case For Ditching The ORM
I obviously was struggling with the ORM. Maybe I was better off without it? 

Ripping it out would mean:
* I would need to use raw SQL statements.<br> 
Getting better at SQL is always good given the global entrenchment of RDBMS. If I had to choose between becoming a master of SQLAlchemy vs SQL, I'd pick SQL every time.

* I could simplify the project strucuture.<br>
Many of the circular import problems I had encountered thus far were caused by the code I needed to initialize and seed the database via the ORM.

* I would (theoretically) be better positioned for more advanced queries later.<br>
Many articles berating ORMs call out the fact that advanced uses (e.g. a query which contains a subquery) are very inefficient via ORM, and I'd probably have to end up writing raw SQL anyways. If I ditched the ORM immediately, I would be better prepared for this eventuality when/if encountered.

* I could remove a large rabbit hole that I was frequetly falling into: trying to understand the inner mechanics of SQLAlchemy.<br>
Several times throughout the project thus far, I'd find a browser instance with 20+ open tabs from a session where I was desperately trying to understand how some mechanic worked (e.g. relationships, lazy loads). If I avoided using the ORM, I could avoid the rabbit hole.

* I could learn to use the native tooling offered by the database itself, rather than the ORM's abstractions.<br>


### The Case For Keeping The ORM
While there were good reasons for tossing the ORM, there were also good reasons for keeping it:

* I expected to change databases soon.<br>
Using Sqlite3 made absolute sense for ease of local development, but my intention to eventually host this solution in Kubernetes meant that it would be relatively simple to switch to a more robust database solution (e.g. MariaDB or Postgres). Normally I'd discount that the argument that an ORM helps prevent database lock-in (how often are you REALLY going to change your Production database solution?) but in this case it was a valid argument.

* I would need to manage the database connections myself.<br>
SQLAlchemy was doing most of the lifting to connect and interact with the database. If I jettisoned this component, I would need to replace it with another database-specific package. The learning curve might be easier with the new package, but it wasn't guaranteed.

* I would need to learn the DB functionlity.<br>
I listed this as a reason for ditching the ORM, but it is also a reason for keeping it. I would still have to expend effort to learn the pecularities of the database. While this might be better for me long-term, it did not lessen the short-term burden.

* I would likely lose the ability to leverage other helper packages.<br>
This issue was of major concern. The ORM itself was just __one__ part of my project. Recognizing that many business ideas would require the ability to manage and display user-specific content, I was going to need other solution components like Account Login and Session Security, much of which was already avaialable OOTB via the Flask ecosystem. 
I had already managed to build basic user login capabilities using the Flask-Login package. This integration, however, was the result of object inheritance (with my SQLAlchmey database table objects inheriting Flask-Login object capabilities). If I jettisoned the ORM (and its database table object definitions), how was I going to be able to leverage these other packages? 
I was either going to lose the functionality entirely, or would need to conduct more research for an alternate solution. This seemed wasteful given that I already had a viable solution that was simple to implement. 

* I did not want to deviate from industry norms.<br>
Regardless of my personal feelings, SQLAlchemy appears a defacto norm in Flask-based application development. I could assume that many of the tutorials and StackOverflow answers I found would have a SQLAlchemy component to them. Furthermore, I could assume that a subset of companies that I interviewed with would be using an ORM of some sort. Better to maintain at least a modicum of familiarity with the technology rather than dismissing it outright.

* I did not want to have to refactor my code again.<br>
I had already executed 4+ refactorings to get the ORM-based solution to a state that seemed to decouple database interaction from the webapp itself, and capable of scaling as the project grew. This was also the effort of several weeks of painful research and trial-and-error. Did I *really* want to scrap it all and restart? 


### Decision: (Compromise) Keep the ORM But Limit How It Is Used
I needed to make a decision so that I could get on with the rest of the damn project. I ultimately decided on a compromise solution: _Keep the ORM in the project, but deliberately limit how its capabilities are used._

Going forward, I would govern my interaction with the ORM as follows:
1) I WILL use the ORM as my database connection solution.
2) I WILL model database tables as ORM objects rather than raw SQL statements.
3) I WILL use ORM syntax to for CRUD operations against the database.<br>
4) I WON'T use relationships in my ORM objects.
5) I WON'T use shorthand syntax (e.g. `Base.query = Session.query_property()`) to minimize typing.
6) I WON'T use cascading operations (i.e. I want the ORM to only execute specific operations that I specifically order it to do, not extra stuff that it opaquely doing on my behalf).

My logic was thus:
* I had working code and it would be crazy to scrap it.
* Scrapping the ORM-based logic would require complete rework on the database-connectivity side but also break the already-working account login functionality.
* The ORM is capable of executing raw SQL but raw SQL can't provide ORM capabilities, so it was better to keep the ORM in the event it was discovered later that ORM functionality was desperately needed.
* I am not comfortable relying too much on opaque ORM functions, particuarly when I don't have a good grasp on the underlying process/mechanism the ORM is managing for me. As a result, its power should be used sparingly.
* Relationships (in this case ORM, but equally applicable to real-life) were a major and continual source of confusion for me. If I simply ignored the ability to use object relationships, I could still leverage the ORM parts that I wanted AND adhere more closely to how a pure SQL solution might look.

I may come to regret these decisions in future, but they were the decisions that were needed now. Assuming I continue to make forward projet, I'll revisit this post and provide an update on how which decisions good and which ones ended up being terrible.

### Update (Roughly 2 Hours After I Said I Had Definitively Decided)
Although I really want to forgo ORM relationships, I had a sneaking suspicion this was a bad idea. What few posts I could find on the discussion all agree that - while you didn't NEED to use relationship functionality alongside your models - it was weird not to. So I made one final research push to see if I should reconsider my decision. 

I decided to start with the [simple example here](https://dev.to/4degrees/sqlalchemy-table-relationships-25od). 
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

Base = declarative_base()
engine = create_engine('sqlite:////tmp/relationshiptest.db')


class SQLAlchemyDBConnection(object):
    """SQLAlchemy database connection"""

    def __init__(self, connection_string='sqlite:////tmp/relationshiptest.db'):
        self.connection_string = connection_string
        self.session = None

    def __enter__(self):
        engine = create_engine(self.connection_string)
        Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
        self.session = Session()  # (bind=engine)
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.session.close()


class user(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    username = Column(String)
    email = Column(String)


class user_blog_post(Base):
    __tablename__ = 'user_blog_post'
    id = Column(Integer, primary_key=True)
    title = Column(String)
    body = Column(String)
    user_id = Column(Integer, ForeignKey('user.id'))


if __name__ == '__main__':
    print('Creating relationshiptest.db')
    Base.metadata.create_all(bind=engine)
```
The Python code above resulted in the Sqlite3 tables below. So far, so good.
```sql
sqlite> .schema user
CREATE TABLE user (
        id INTEGER NOT NULL,
        username VARCHAR,
        email VARCHAR,
        PRIMARY KEY (id)
);
sqlite> .schema user_blog_post
CREATE TABLE user_blog_post (
        id INTEGER NOT NULL,
        title VARCHAR,
        body VARCHAR,
        user_id INTEGER,
        PRIMARY KEY (id),
        FOREIGN KEY(user_id) REFERENCES user (id)
);
sqlite>
```
Sticking with the example, let's imagine that we have a user who created 2 blog posts. Logically, we must:
1) Create a `user` table entry
2) Get the `user.id` of the newly created `user` record
3) Create a `user_blog_post`, making sure to populate the `user_blog_post.user_id` foreign key field with the `user.id` we retrieved in Step 2.
4) Create another `user_blog_post`, once again populating the `user_blog_post.user_id` foreign key field with the `user.id` we retrieved in Step 2.

The hurdle here is that I won't be able to populate the `user_blog_post` records' foreign key column until I've created the user. So this means I need to somehow get the database to return the `user.id` value during the INSERT transaction (so I don't have to invoke a second transaction to query the value of `user.id`) or I need to group together the user INSERT with the subsequent user_blog_post INSERTs.

A random Google query offers hints on various methods like:
* [using RETURNING](https://stackoverflow.com/questions/8589674/sqlalchemy-getting-the-id-of-the-last-record-inserted/8590301) 
Example:
```python
result = conn.execute("INSERT INTO user(username, email) VALUES ('Moshe', 'moshe@4degrees.ai') RETURNING id")
user_id = result.fetchone()

## Populate blog posts here
```

* [grouping statements](https://stackoverflow.com/questions/175066/sql-server-is-it-possible-to-insert-into-two-tables-at-the-same-time)
Example:
```sql
BEGIN TRANSACTION
   DECLARE @DataID int;
   INSERT INTO DataTable (Column1 ...) VALUES (....);
   SELECT @DataID = scope_identity();
   INSERT INTO LinkTable VALUES (@ObjectID, @DataID);
COMMIT
```

I'm sure I could replicate something akin to this functionality using the ORM model, but it would require research. And I would need to figure out if Sqlite3 supported it (one comment said it did not, but the post was from 2011 so it's possible it is supported now). The first method seems less efficient (3 transactions instead of 1) but I might be able to bulk insert the two blog posts to bring the first one down to two (something else I would need to research)...

This is just a tiny, easy example and we can already see extra steps and work piling up. OR, despite my distate for it, I can use the ORM functionality that was designed to specifically make this process easier for me: `relationships`.




Next: [Database Connection Pattern](./08-database-connection-pattern.md)<br>
Previous: 
