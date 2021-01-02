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

This is just a tiny, easy example and we can already see extra steps and work piling up. Or, despite my distate for it, I can use the ORM functionality that was designed to specifically make this process easier for me: `relationship`. With one extra line added to the `user` class, I can leverage the ORM to coordinate all the inserts for me.

### Digging Into Relationships
To go through this methodically, I'm going to remove the ForeignKey entry in my `user_blog_post.user_id` definition. The full code that I will try to execute now looks like:
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

Base = declarative_base()
engine = create_engine('sqlite:////tmp/relationshiptest.db', echo=True)


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
    # Adding relationship linkage
    blog_posts = relationship("user_blog_post", backref="user")


class user_blog_post(Base):
    __tablename__ = 'user_blog_post'
    id = Column(Integer, primary_key=True)
    title = Column(String)
    body = Column(String)
    user_id = Column(Integer)  #, ForeignKey('user.id'))


if __name__ == '__main__':
    print('Creating relationshiptest.db')
    Base.metadata.create_all(bind=engine)

    newUser = user(username='Moshe', email="moshe@4degrees.ai")
    newUser.blog_posts = [
        user_blog_post(title="first", body="first post body"),
        user_blog_post(title="second", body="second post body")
    ]

    with SQLAlchemyDBConnection() as db:
        db.session.add(newUser)
        db.session.commit()
```



```python
class user(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    username = Column(String)
    email = Column(String)
    # Adding relationship linkage
    blog_posts = relationship("user_blog_post", backref="user")
```

Unfortunately, this code fails with the interpreter returning a `sqlalchemy.exc.NoForeignKeysError: Can't find any foreign key relationships between 'user' and 'user_blog_post'` error. From this we can conclude that we must provide foreign key definitions in our object models if we want to leverage the ORM's ability to streamline inserts for us.

I changed `user_id = Column(Integer)  #, ForeignKey('user.id'))` back to `user_id = Column(Integer, ForeignKey('user.id'))` and ran the code again. I then checked the `user` and `user_blog_post` tables created in my Sqlite3 database.
```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.

sqlite> select * from user;
1|Moshe|moshe@4degrees.ai

sqlite> select * from user_blog_post;
1|first|first post body|1
2|second|second post body|1
sqlite>
```
As we can see from the results of of `user_blog_post` query, the ORM automatically got the `user.id` value generated by the database when the user was created and added it as the `user_blog_post.user_id` parameter when inserting records into the `user_blog_post` table. This is further demonstrated by adding a few more records in the same transaction:
```python
if __name__ == '__main__':
    print('Creating relationshiptest.db')
    Base.metadata.create_all(bind=engine)

    newUser = user(username='Moshe', email="moshe@4degrees.ai")
    newUser.blog_posts = [
        user_blog_post(title="first", body="first post body"),
        user_blog_post(title="second", body="second post body")
    ]

    newUser2 = user(username='Person2', email='person2@gmail.com')
    newUser3 = user(username='Person3', email='person3@gmail.com')
    newUser3.blog_posts = [
        user_blog_post(title="Person3's Blog", body="The first blog body for Person3")
    ]

    with SQLAlchemyDBConnection() as db:
        db.session.add(newUser)
        db.session.add(newUser2)
        db.session.add(newUser3)
        db.session.commit()
```

Results in:
```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.

sqlite> select * from user;
1|Moshe|moshe@4degrees.ai
2|Person2|person2@gmail.com
3|Person3|person3@gmail.com

sqlite> select * from user_blog_post;
1|first|first post body|1
2|second|second post body|1
3|Person3's Blog|The first blog body for Person3|3
```
I set the SQLAlchemy engine's echo flag to True in order to see the SQL statements that SQLAlchemy was emitting to turn my Python commands into actual SQL records. The following commands appeared on stdout:
```sql
2021-01-02 10:36:42,234 INFO sqlalchemy.engine.base.Engine SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
2021-01-02 10:36:42,236 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 10:36:42,238 INFO sqlalchemy.engine.base.Engine SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
2021-01-02 10:36:42,241 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 10:36:42,243 INFO sqlalchemy.engine.base.Engine PRAGMA main.table_info("user")
2021-01-02 10:36:42,244 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 10:36:42,247 INFO sqlalchemy.engine.base.Engine PRAGMA main.table_info("user_blog_post")
2021-01-02 10:36:42,247 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 10:36:42,256 INFO sqlalchemy.engine.base.Engine SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
2021-01-02 10:36:42,257 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 10:36:42,260 INFO sqlalchemy.engine.base.Engine SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
2021-01-02 10:36:42,261 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 10:36:42,262 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-02 10:36:42,264 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 10:36:42,265 INFO sqlalchemy.engine.base.Engine ('Moshe', 'moshe@4degrees.ai')
2021-01-02 10:36:42,268 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 10:36:42,269 INFO sqlalchemy.engine.base.Engine ('Person2', 'person2@gmail.com')
2021-01-02 10:36:42,270 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 10:36:42,273 INFO sqlalchemy.engine.base.Engine ('Person3', 'person3@gmail.com')
2021-01-02 10:36:42,275 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 10:36:42,277 INFO sqlalchemy.engine.base.Engine ('first', 'first post body', 1)
2021-01-02 10:36:42,279 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 10:36:42,280 INFO sqlalchemy.engine.base.Engine ('second', 'second post body', 1)
2021-01-02 10:36:42,281 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 10:36:42,284 INFO sqlalchemy.engine.base.Engine ("Person3's Blog", 'The first blog body for Person3', 3)
2021-01-02 10:36:42,286 INFO sqlalchemy.engine.base.Engine COMMIT
```

SQLAlchmey was somehow retrieving the row ids of newly-created `user` records, but I didn't see where this was happening in the emitted SQL statements. The data didn't seem to be coming from the COMMIT step, because we see different `user.id` values being used in the preceding statements. I figured either:
* Sqlite must be returning some sort of response object containing the value.
* SQLAlchemy must be doing some sort of behind the scenes `refresh`, which queried the database immediately after the insert of the user object.

This sent me down a rabbithole of SQLAlchemy documentation and old StackOverflow posts that I didn't trust. After hours of searching, I was unable to pinpoint the exact mechanism at play here but I was able to get a rough idea of how the mechanism worked by halting my program at regular intervals and directly checking the Sqlite database. Here's what I did:

I rewrote the __main__ code block, inserting `input` commands between the various database commands. By causing the program to wait until I provided input, I could step through the program and check the database to see what effect the commands were having on records within:
```python
if __name__ == '__main__':
    print('Creating relationshiptest.db')
    Base.metadata.create_all(bind=engine)

    newUser = user(username='Moshe', email="moshe@4degrees.ai")
    # Comment this out so we only create the user, not related blog_posts
    #newUser.blog_posts = [
    #    user_blog_post(title="first", body="first post body"),
    #    user_blog_post(title="second", body="second post body")
    #]

    newUser2 = user(username='Person2', email='person2@gmail.com')
    newUser3 = user(username='Person3', email='person3@gmail.com')
    newUser3.blog_posts = [
        user_blog_post(title="Person3's Blog", body="The first blog body for Person3")
    ]

    with SQLAlchemyDBConnection() as db:
        print('Adding newUser')
        db.session.add(newUser)
        input('Press any key1')
        db.session.flush()  # Not needed. SQLAlchemy has autoflush=True by default
        input('Press any key2')
        print('Check if we have access to user.id via SQLA and then check db')
        print('User.id is: ', newUser.id)
        input('Press any key3')
        # Now create the blog_posts.
        newUser.blog_posts = [
            user_blog_post(title="first", body="first post body"),
            user_blog_post(title="second", body="second post body")
        ]
        input('press any key')
        db.session.add(newUser2)
        db.session.add(newUser3)
        db.session.commit()
```

The first pause occurred after I added `newUser`. No SQL was emitted by the command, nor were there any records return by SELECT statments:
```stdout
Creating relationshiptest.db
2021-01-02 13:51:12,218 INFO sqlalchemy.engine.base.Engine SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
2021-01-02 13:51:12,218 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:51:12,220 INFO sqlalchemy.engine.base.Engine SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
2021-01-02 13:51:12,220 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:51:12,222 INFO sqlalchemy.engine.base.Engine PRAGMA main.table_info("user")
2021-01-02 13:51:12,222 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:51:12,224 INFO sqlalchemy.engine.base.Engine PRAGMA temp.table_info("user")
2021-01-02 13:51:12,225 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:51:12,226 INFO sqlalchemy.engine.base.Engine PRAGMA main.table_info("user_blog_post")
2021-01-02 13:51:12,226 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:51:12,227 INFO sqlalchemy.engine.base.Engine PRAGMA temp.table_info("user_blog_post")
2021-01-02 13:51:12,228 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:51:12,230 INFO sqlalchemy.engine.base.Engine
CREATE TABLE user (
        id INTEGER NOT NULL,
        username VARCHAR,
        email VARCHAR,
        PRIMARY KEY (id)
)


2021-01-02 13:51:12,237 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:51:12,245 INFO sqlalchemy.engine.base.Engine COMMIT
2021-01-02 13:51:12,246 INFO sqlalchemy.engine.base.Engine
CREATE TABLE user_blog_post (
        id INTEGER NOT NULL,
        title VARCHAR,
        body VARCHAR,
        user_id INTEGER,
        PRIMARY KEY (id),
        FOREIGN KEY(user_id) REFERENCES user (id)
)


2021-01-02 13:51:12,254 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:51:12,260 INFO sqlalchemy.engine.base.Engine COMMIT
Adding newUser
Press any key1
```

```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> .tables
user            user_blog_post
sqlite> select * from user;
sqlite> select * from user_blog_post;

```

The second pause occured after I flushed the session (_Note: I'm not sure I needed to do this since SQLAlchemy is supposedly using autoflush=True by default_). This resulted in INSERT statement being emitted by the ORM but still nothing returned by the SELECT.
```stdout
Press any key1
2021-01-02 13:53:53,248 INFO sqlalchemy.engine.base.Engine SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
2021-01-02 13:53:53,248 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:53:53,249 INFO sqlalchemy.engine.base.Engine SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
2021-01-02 13:53:53,250 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 13:53:53,251 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-02 13:53:53,255 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 13:53:53,257 INFO sqlalchemy.engine.base.Engine ('Moshe', 'moshe@4degrees.ai')
Press any key2
```
```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> select * from user;
sqlite> select * from user_blog_post;
```

The third pause occured after I printed `newUser.id`. The only way I can have this value is if SQLalchemy was somehow able to retrieve the value from the Sqlite AFTER the record was created. The idea was successfully printed but still nothing when using SELECT.
```stdout
Press any key2
Check if we have access to user.id via SQLA and then check db
User.id is:  1
Press any key3
```
```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> select * from user;
sqlite> select * from user_blog_post;
```

The fourth pause was after the creation of the `newUser` blog post entries via the relationship mechanism. The ORM emitted SQL but nothing seemed to be present when I checked with SELECT. More confusingly, the ORM seemed to be issuing its own SELECT query here (which is strange because I thought I was only creating Python objects?)
```stdout
Press any key3
2021-01-02 14:01:42,699 INFO sqlalchemy.engine.base.Engine SELECT user_blog_post.id AS user_blog_post_id, user_blog_post.title AS user_blog_post_title, user_blog_post.body AS user_blog_post_body, user_blog_post.user_id AS user_blog_post_user_id
FROM user_blog_post
WHERE ? = user_blog_post.user_id
2021-01-02 14:01:42,703 INFO sqlalchemy.engine.base.Engine (1,)
press any key
```
```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> select * from user;
sqlite> select * from user_blog_post;
sqlite>
```
Finally, we commit the records. This results in more users and blog posts being created AND these records now showing up when I SELECT directly from the database:
```stdout
press any key
2021-01-02 14:04:36,406 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 14:04:36,406 INFO sqlalchemy.engine.base.Engine ('Person2', 'person2@gmail.com')
2021-01-02 14:04:36,410 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 14:04:36,410 INFO sqlalchemy.engine.base.Engine ('Person3', 'person3@gmail.com')
2021-01-02 14:04:36,413 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:04:36,414 INFO sqlalchemy.engine.base.Engine ('first', 'first post body', 1)
2021-01-02 14:04:36,416 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:04:36,418 INFO sqlalchemy.engine.base.Engine ('second', 'second post body', 1)
2021-01-02 14:04:36,420 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:04:36,426 INFO sqlalchemy.engine.base.Engine ("Person3's Blog", 'The first blog body for Person3', 3)
2021-01-02 14:04:36,427 INFO sqlalchemy.engine.base.Engine COMMIT
```
```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.

sqlite> select * from user;
1|Moshe|moshe@4degrees.ai
2|Person2|person2@gmail.com
3|Person3|person3@gmail.com

sqlite> select * from user_blog_post;
1|first|first post body|1
2|second|second post body|1
3|Person3's Blog|The first blog body for Person3|3
```

I ran one more test, after executing `db.session.flush()`, I inserted a record directly from the Sqlite CLI and then resumed the pause Python code. The code continued executing thinking it was using rowid 1 until it errored out on the commit step, throwing a http://sqlalche.me/e/13/e3q8 error. 
```sql
sqlite> INSERT INTO user(username,email) VALUES ('Thiswill', 'BreakMyCode');
sqlite> SELECT * from user;
1|ThisWill|BreakMyCode
```

```stdout
press any key
2021-01-02 14:12:46,530 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 14:12:46,530 INFO sqlalchemy.engine.base.Engine ('Person2', 'person2@gmail.com')
2021-01-02 14:12:46,534 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 14:12:46,535 INFO sqlalchemy.engine.base.Engine ('Person3', 'person3@gmail.com')
2021-01-02 14:12:46,539 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:12:46,544 INFO sqlalchemy.engine.base.Engine ('first', 'first post body', 1)
2021-01-02 14:12:46,546 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:12:46,547 INFO sqlalchemy.engine.base.Engine ('second', 'second post body', 1)
2021-01-02 14:12:46,548 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:12:46,550 INFO sqlalchemy.engine.base.Engine ("Person3's Blog", 'The first blog body for Person3', 3)
2021-01-02 14:12:46,551 INFO sqlalchemy.engine.base.Engine COMMIT
Traceback (most recent call last):
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/engine/base.py", line 774, in _commit_impl
    self.engine.dialect.do_commit(self.connection)
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/engine/default.py", line 546, in do_commit
    dbapi_connection.commit()
sqlite3.OperationalError: disk I/O error

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "relationshiptest.py", line 81, in <module>
    db.session.commit()
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/orm/session.py", line 1042, in commit
    self.transaction.commit()
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/orm/session.py", line 508, in commit
    t[1].commit()
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/engine/base.py", line 1765, in commit
    self._do_commit()
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/engine/base.py", line 1796, in _do_commit
    self.connection._commit_impl()
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/engine/base.py", line 776, in _commit_impl
    self._handle_dbapi_exception(e, None, None, None, None)
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/engine/base.py", line 1511, in _handle_dbapi_exception
    util.raise_(
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/util/compat.py", line 178, in raise_
    raise exception
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/engine/base.py", line 774, in _commit_impl
    self.engine.dialect.do_commit(self.connection)
  File "/home/deeplearning/python-virtual-environments/flask_poc/flask_poc_venv/lib/python3.8/site-packages/sqlalchemy/engine/default.py", line 546, in do_commit
    dbapi_connection.commit()
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) disk I/O error
(Background on this error at: http://sqlalche.me/e/13/e3q8)
```


Sent me down a rabbit hole of SQLAlchemy documentation and old stackoverflow posts I didn't trust.
Why do I care? Performance and cost - computationally expensive, can impact perforamnce, and cloud providers charge by the READ/WRITE.


Essentially, the relationship entry means "

Dont like backref. Lazy. Be explicit on both sides.
Does the implicit join only work because there is a single Foreign Key?

Next: [Database Connection Pattern](./08-database-connection-pattern.md)<br>
Previous: 
