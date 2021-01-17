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

* Instantiation and syntax patterns had repeatedly changed as I moved from Flask-SQLAlchemy to SQLAlchemy, and then needed to change again to accommodate the creation of a context manager for database connections (granted partially driven by the limitations of Sqlite to only accept requests from one process at a time). 

* I had to find, research, and eventually rewrite a series of functions (Bob Waycott) to be able to find and load multiple models spread over multiple files. While the search logic was reusable, the model creation was tied to the SQLAlchemy Base class, thereby requiring me to rewrite the database module to separate the logic the instantiates db object (Base, engine, ContenxtManager) from poplation functions (due to circular imports).  

* The population of test data began to cause me trouble. It was simple enough to populate records in an isolated table, but how was I supposed to populate the assocation tables? My Models had relationships attributes that were tied to Association tables in the database, but associations were a programmatic relationship that the ORM would turn into the requisite SQL behind the scenes. Was I cheating myself of the opportunity to truly understand how one is supposed to update a series of records in a database by using this confusing shortcut?


### The Case For Ditching The ORM
I obviously was struggling with the ORM. Maybe I was better off without it? Ripping it out would mean:

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

SQLAlchemy was somehow retrieving the row ids of newly-created `user` records, but I didn't see where this was happening in the emitted SQL statements. The data didn't seem to be coming from the COMMIT step, because we see different `user.id` values being used in the preceding statements. I figured either:
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
        print('Committing')
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
Committing
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
Committing
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

This noticed behaviour aligns with some of the responses provided in a very old StackOverflow post [sqlalchemy flush() and get inserted id?](https://stackoverflow.com/questions/1316952/sqlalchemy-flush-and-get-inserted-id). Assuming the information can be believed, one answer said that 'primary-key attributes are populated immediately with the flush() process as they are generated, and no call to commit() should be required', with a second commenter writing '... a refresh is not necessary. Once you flush, you can access the id field, sqlalchemy automatically refreshes the id which is auto-generated at the backend." 

I believe the bit about the primary key being generated (we could clearly see the value available for insertion on the subsequent blog post creation) but I'm not so sure about the `refresh` comment. As per the official [refresh](https://docs.sqlalchemy.org/en/13/orm/session_api.html#sqlalchemy.orm.session.Session.refresh) documentation, a refresh will cause 'a query to be issued to the database and all attributes will be refreshed with their current database value.' Given that I couldn't retrieve the `user` record via the DB CLI query, why would it suddenly work for a SQLAlchemy-based query? 

Something was still fishy re the explanation of this behaviour but - short of setting up a proxy to capture all the traffic between SQLAlchemy and Sqlite (which would require more research and work) - I had taken this investigation as far as I could justifiably invest the time. I decided that I would just accept the hazy behaviour for what it was and issue myself a new project maxim "Make sure the ORM is the only component that speaks to the database". It was time to get on to other things.

#### So What's Going on Here?
I'm still not 100 sure whether the row id is coming from a response object, being refreshed by SQLAlchemy behind the scenes, or something else (and a little bit frustrated this isn't clearly described in the documentation or source code). With that said, after reading ["SQLAlchemy commit(), flush(), expire(), refresh(), merge() - what's the difference?"](https://www.michaelcho.me/article/sqlalchemy-commit-flush-expire-refresh-merge-whats-the-difference), by Michael Cho, I have some suspicions regarding the flow:
1) INSERT statements were only emitted by the ORM on the `.flush()` and `.commit()` commands.
2) Using `.flush()` or `.commit()` (which calls flush behind-the-scenes) results in the objects in memory being written to the __Database Transaction Buffer__. This is essentially a "soft write" where a record row has been allocated to each record (hence why we subsequently have access to the database's record id), but will not be finalized until we do a COMMIT (at which point the record will become available to queries).

I confirmed my suspicions by further modifying my test code. I commented out the `.flush()` step and try to print the `newUser.id` before the soft write to the database. Any attempts to write this value return a value of None until the softwrite occurred:
```python
with SQLAlchemyDBConnection() as db:
        print('Adding newUser')
        db.session.add(newUser)
        print('User.id is: ', newUser.id)
        input('Press any key1')
        # db.session.flush()  # Not needed. SQLAlchemy has autoflush=True by default
        input('Press any key2')
        print('Check if we have access to user.id via SQLA and then check db')
        print('User.id is: ', newUser.id)
        input('Press any key3')
```
```stdout
Creating relationshiptest.db
2021-01-02 14:32:07,218 INFO sqlalchemy.engine.base.Engine SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
2021-01-02 14:32:07,218 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 14:32:07,220 INFO sqlalchemy.engine.base.Engine SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
2021-01-02 14:32:07,220 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 14:32:07,222 INFO sqlalchemy.engine.base.Engine PRAGMA main.table_info("user")
2021-01-02 14:32:07,222 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 14:32:07,225 INFO sqlalchemy.engine.base.Engine PRAGMA main.table_info("user_blog_post")
2021-01-02 14:32:07,226 INFO sqlalchemy.engine.base.Engine ()
Adding newUser
User.id is:  None
Press any key1
Press any key2
Check if we have access to user.id via SQLA and then check db
User.id is:  None
Press any key3
press any key
Committing
2021-01-02 14:37:34,826 INFO sqlalchemy.engine.base.Engine SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
2021-01-02 14:37:34,826 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 14:37:34,828 INFO sqlalchemy.engine.base.Engine SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
2021-01-02 14:37:34,828 INFO sqlalchemy.engine.base.Engine ()
2021-01-02 14:37:34,829 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-02 14:37:34,831 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 14:37:34,831 INFO sqlalchemy.engine.base.Engine ('Moshe', 'moshe@4degrees.ai')
2021-01-02 14:37:34,834 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 14:37:34,835 INFO sqlalchemy.engine.base.Engine ('Person2', 'person2@gmail.com')
2021-01-02 14:37:34,837 INFO sqlalchemy.engine.base.Engine INSERT INTO user (username, email) VALUES (?, ?)
2021-01-02 14:37:34,838 INFO sqlalchemy.engine.base.Engine ('Person3', 'person3@gmail.com')
2021-01-02 14:37:34,844 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:37:34,845 INFO sqlalchemy.engine.base.Engine ('first', 'first post body', 10)
2021-01-02 14:37:34,846 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:37:34,847 INFO sqlalchemy.engine.base.Engine ('second', 'second post body', 10)
2021-01-02 14:37:34,848 INFO sqlalchemy.engine.base.Engine INSERT INTO user_blog_post (title, body, user_id) VALUES (?, ?, ?)
2021-01-02 14:37:34,849 INFO sqlalchemy.engine.base.Engine ("Person3's Blog", 'The first blog body for Person3', 12)
2021-01-02 14:37:34,850 INFO sqlalchemy.engine.base.Engine COMMIT
```

#### Super, Why Did you Spend Hours of Your Life Trying to Figure This Out?
In addition to simply not liking not knowing how the mechanics of my tooling work, I feel there are larger rammifications at play - particularly when it comes to additional database transactions.

In a small project on a local machine, it's mostly irrelevant if your application needs to make an additional call to the database. However, if you are trying to create a performant application that uses microservice architecture, one should try to limit the computational and network latency costs as much a possible. Furthermore, I understand that some cloud databases (e.g. DynamoDB) incur a charge for every Read & Write made to the database. As a result, I feel it is well worth havinga solid understanding of the Application-Database interaction model, as this can provide a better knowledge base upon which to build future decisions. 

For now, I'll just accept the behaviour for what it is and revisit at a later point once I've made more progress on the rest of the work.


### That Was A Fun Diversion - Could You Please Get Back to Describing How to Use Relationships?
One article on SQLAlchemy relationships defined them as a way to "navigate foreign key relationships through attributes on your model". I prefer to think about it as telling SQLAlchemy "I want you to worry about managing the foreign keys as we interact with database records".

Think back to our logical sequencing of steps when we wanted to link a `user` to a `blog_post`. We needed to:
1. Create `user` record
2. Get userid
3. Create blog_post providing `user.id` as a Foreign Key on the `blog_post` record.
4. Repeat Step 3 as needed.

From a SQL sequencing perspective, we can't write `blog_post` records to the database until after the `user` is created. But that doesn't mean we are limited by this constraint when writing our code. Using `relationship` definitions, we can stay at the logical level where we create and define the connection between ALL the objects (i.e. a user and two user_blog_post objects), and leave it to SQLAlchmey to ensure that all the entries have their foreign keys populated properly when the objects are flushed to the database. 

```python
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
    # Note: ForeignKey MUST be defined or relationship insert wont work.
    user_id = Column(Integer, ForeignKey('user.id'))
    
if __name__ == "__main__":
    # CREATE THE OBJECTS AND LINK THEM VIA THE blog_posts SHORTHAND.
    newUser = user(username='Moshe', email="moshe@4degrees.ai")
    newUser.blog_posts = [
        user_blog_post(title="first", body="first post body"),
        user_blog_post(title="second", body="second post body")
        
    # FLUSH AND COMMIT HAPPENS HERE, AT WHICH POINT 3 RECORDS WILL BE CREATED AND EACH user_blog_post.user_id 
    # WILL BE POPULATED WITH THE AUTO-GENERATED PRIMARY KEY ASSIGNED TO THE user.id FIELD OF THE CREATED user RECORD
    db.session.add(newUser)
    db.session.commit()
```
When we check on the records create in the Sqlite database (via the Sqlite CLI), we see:
```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.

sqlite> .tables
user            user_blog_post

sqlite> select * from user;
1|Moshe|moshe@4degrees.ai

sqlite> select * from user_blog_post;
1|first|first post body|1
2|second|second post body|1
```

#### What Is Written Must Be Retrieved
We are eventually going to want to retrieve records that we've written to the database. Whereas raw SQL would require use to craft a proper JOIN query, we can once again leverage the `relationship` mechanism to retrive the data in a programmatic not-dotation manner.

Imagine we want to retrieve the `user` and `user_blog_post` entries that we just created. This simple Python snippet is enough to complete the task:
```python
# Using the db connection context manager I created earlier
with SQLAlchemyDBConnection() as db:
    person1 = db.session.query(user).first()
    print(f"{person1.blog_posts[0].title}")
```
Which results in the follow SQL being emitted by SQLALCHEMY and resulting stdout:
```stdout
2021-01-04 16:31:49,900 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-04 16:31:49,902 INFO sqlalchemy.engine.base.Engine SELECT user.id AS user_id, user.username AS user_username, user.email AS user_email
FROM user
 LIMIT ? OFFSET ?
2021-01-04 16:31:49,908 INFO sqlalchemy.engine.base.Engine (1, 0)
2021-01-04 16:31:49,911 INFO sqlalchemy.engine.base.Engine SELECT user_blog_post.id AS user_blog_post_id, user_blog_post.title AS user_blog_post_title, user_blog_post.body AS user_blog_post_body, user_blog_post.user_id AS user_blog_post_user_id
FROM user_blog_post
WHERE ? = user_blog_post.user_id
2021-01-04 16:31:49,916 INFO sqlalchemy.engine.base.Engine (1,)
first
```
Easy-peasy, right? Yes, so long as you remain aware of a few caveats:
TO DO: REORDER THESE
1) Relationship Loading Technique and Join Type 
2) Session Object Mapping and Lazy Load
3) Session Creation
4) Query syntax
5) Backref vs Back_Populates
6) Extending the object model with properties

###### Relationship Loading Technique and Join Type
SQLAlchemy has [reams of documentation](https://docs.sqlalchemy.org/en/14/orm/loading_relationships.html) about the mechanics of relationship loading. It is important to note that the default "lazy loading" technique __does not__ query the records stored in tables other than the object we supplied as part of our ORM query. This technique will issue a second SELECT statement to retrieve these records in the event we try to access any of these attributes via the original object's relationship.

Looking at the `stdout` of the example above, you'll notice there are TWO SELECT statements:
* The first SELECT was issued in reponse to the ```python person1 = db.session.query(user).first() ``` invocation.
* The second SELECT was issued in response to the ```python print(f"{person1.blog_posts[0].title}")``` invocation. In this case, we are using the `blog_post` relationship as a way to jump from the `user` object to a related `user_blog_post` object, but the ORM doesn't have this data because its lazyload only grabbed the `user` table data! The second query was automatically invoked to retrieve the `user_blog_post` object data needed to successfully complete the print command.

I mentioned earlier in this post that one of the design considerations I was minding was the number of database transactions the application must invoke in order to complete a request. Using lazyload carelessly can significantly increase this number, but conversely it also limits the amount of data returned in a response. Depending on buiness need, it may be necessary to replace the the default behaviour with a more [eager SELECT option](https://docs.sqlalchemy.org/en/14/orm/loading_relationships.html#relationship-loading-techniques) to ensure a multitable dataset is returned via a single query.
```python
# EXAMPLE 1 - LOAD DEFINITION IN CLASS DEFINITION
class user(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    username = Column(String)
    email = Column(String)
    # Adding relationship linkage
    blog_posts = relationship("user_blog_post", backref="user", lazy="joined")
    
# EXAMPLE 2 - LOAD DEFINITION IN QUERY SPECIFICATION
user_and_blogposts = db.session.query(user).options(joinedload(user.blog_posts)).all()
```

CAVEAT: SQLAlchemy eager loads use a LEFT OUTER JOIN by default (see [here](https://stackoverflow.com/questions/38549/what-is-the-difference-between-inner-join-and-outer-join#:~:text=You%20use%20INNER%20JOIN%20to,and%20columns%20will%20have%20values.&text=LEFT%20OUTER%20JOIN%20returns%20all,matches%20in%20the%20second%20table.) for a refresher on SQL JOINs if you suddenly feel a little hazy on the types). An [INNER JOIN can be used instead](https://docs.sqlalchemy.org/en/14/orm/loading_relationships.html#joined-eager-loading), defined either at design time or within the query itself.

See [What Kind of Loading to Use?](https://docs.sqlalchemy.org/en/14/orm/loading_relationships.html#relationship-loading-techniques) for more guidance by the SQLAlchemy folks on the trade-offs between calls, complexity, and performacne.

```python
# EXAMPLE 1 - INNER JOIN DEFINED IN CLASS DEFINITION
class user(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    username = Column(String)
    email = Column(String)
    # Adding relationship linkage
    blog_posts = relationship("user_blog_post", backref="user", lazy="joined", innerjoin=True)
    
# EXAMPLE 2 - INNER JOIN DEFINITION IN QUERY SPECIFICATION
user_and_blogposts = db.session.query(user).options(joinedload(user.blog_posts, innerjoin=True)).all()
```

##### Session Object Mapping and Lazy Load
As noted above, using a Lazy Load relationship loading technique will cause SQLAlchemy to emit a second SELECT query if we try to access data on another table via a retrieved record's relationship attribute. The documentation identifies [one exception to the rule](https://docs.sqlalchemy.org/en/14/orm/loading_relationships.html#lazy-loading): "The one case where SQL is not emitted is for a simple many-to-one relationship, when the related object can be identified by its primary key alone and that object is already present in the current Session. For this reason, while lazy loading can be expensive for related collections, in the case that one is loading lots of objects with simple many-to-ones against a relatively small set of possible target objects, lazy loading may be able to refer to these objects locally without emitting as many SELECT statements as there are parent objects."

I didn't fully grasp the implications of this until I ran the following test:
```python
### SAME CODE HERE THAT I USED IN MY PREVIOUS EXAMPLES PRIOR TO THE __main__ DUNDER
if __name__ == '__main__':

    Base.metadata.create_all(bind=engine)

    newUser = user(username='Moshe', email="moshe@4degrees.ai")
    newUser.blog_posts = [
        user_blog_post(title="first", body="first post body"),
        user_blog_post(title="second", body="second post body")
    ]

    with SQLAlchemyDBConnection() as db:
        # ADD user AND RELATED user_blog_posts TO DATABASE
        db.session.add(newUser)
        db.session.commit()

        # RETRIEVE user AND RELATED BLOG POSTS
        print("\n\n1 -----------------------------")
        person1 = db.session.query(user).first()
        print(f"{person1.blog_posts[0].title}")

        print("\n\n1A ----------------------------")
        person1A = db.session.query(user).first()
        print(f"{person1A.blog_posts[0].title}")
        
        input('COMPARE SELECT EMISSION FOR person1 VS person1A')
```

```stdout
1 -----------------------------
2021-01-05 09:28:53,084 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-05 09:28:53,086 INFO sqlalchemy.engine.base.Engine SELECT user.id AS user_id, user.username AS user_username, user.email AS user_email
FROM user
 LIMIT ? OFFSET ?
2021-01-05 09:28:53,091 INFO sqlalchemy.engine.base.Engine (1, 0)
2021-01-05 09:28:53,097 INFO sqlalchemy.engine.base.Engine SELECT user_blog_post.id AS user_blog_post_id, user_blog_post.title AS user_blog_post_title, user_blog_post.body AS user_blog_post_body, user_blog_post.user_id AS user_blog_post_user_id
FROM user_blog_post
WHERE ? = user_blog_post.user_id
2021-01-05 09:28:53,101 INFO sqlalchemy.engine.base.Engine (1,)
first


1A ----------------------------
2021-01-05 09:28:53,108 INFO sqlalchemy.engine.base.Engine SELECT user.id AS user_id, user.username AS user_username, user.email AS user_email
FROM user
 LIMIT ? OFFSET ?
2021-01-05 09:28:53,112 INFO sqlalchemy.engine.base.Engine (1, 0)
first

COMPARE SELECT EMISSION FOR person1 VS person1A
```
The Python code creates a single `user` record and two related `user_blog_post` records in the database. It then runs two  successive transactions to retrieve the records:
* The first transaction retrieves the first record found in the `user` table and then tries to access the title of first `user_blog_post` record (accessed via the `user_blog` relationship we had defined in our models). Because I didn't change the default relationship loading technique, SQLAlchemy used the lazy load method - issuing one query to retrieve the `user` and then issuing a follow-up query in response to the attempt to access the `user_blog_post` title via the relationship. This is why we see two SELECT statements being echoed in the stdout of the transaction.

* The second transaction repeats this process: retrieve the first record in the `user` table and then retrieve the associated blog post title via relationship. This time, only a _single_ SELECT statement is issued before the code is able to finish execution. What gives?

This appears to be the exception that SQLAlchemy was calling out. We had already retrieved the blog post in the previous query and were still working within the same database session. As a result of the way SQLALchemy converted the blog_post_record back into memory, it was able to tell that it already had the record and thus had to no need to issue the follow-up query.

I confirmed this behaviour by modifying the code slightly, so that each retrieval transaction took place with its own Session object. This resulted in both transactions executing both an initial query for the `user` record, with a follow-up query for the user_blog_post object when we tried to access it via the relationship.
```python
...
if __name__ == '__main__':

    Base.metadata.create_all(bind=engine)

    newUser = user(username='Moshe', email="moshe@4degrees.ai")
    newUser.blog_posts = [
        user_blog_post(title="first", body="first post body"),
        user_blog_post(title="second", body="second post body")
    ]

    with SQLAlchemyDBConnection() as db:
        # ADD user AND RELATED user_blog_posts TO DATABASE
        db.session.add(newUser)
        db.session.commit()

        # RETRIEVE user AND RELATED BLOG POSTS
        print("\n\n1 -----------------------------")
        person1 = db.session.query(user).first()
        print(f"{person1.blog_posts[0].title}")
    
    # INSTANTIATE A NEW SESSION OBJECT
    with SQLAlchemyDBConnection() as db:
        print("\n\n1A ----------------------------")
        person1A = db.session.query(user).first()
        print(f"{person1A.blog_posts[0].title}")
        
        input('COMPARE SELECT EMISSION FOR person1 VS person1A')
```

```stdout
...
1 -----------------------------
2021-01-05 12:58:12,666 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-05 12:58:12,667 INFO sqlalchemy.engine.base.Engine SELECT user.id AS user_id, user.username AS user_username, user.email AS user_email
FROM user
 LIMIT ? OFFSET ?
2021-01-05 12:58:12,674 INFO sqlalchemy.engine.base.Engine (1, 0)
2021-01-05 12:58:12,678 INFO sqlalchemy.engine.base.Engine SELECT user_blog_post.id AS user_blog_post_id, user_blog_post.title AS user_blog_post_title, user_blog_post.body AS user_blog_post_body, user_blog_post.user_id AS user_blog_post_user_id
FROM user_blog_post
WHERE ? = user_blog_post.user_id
2021-01-05 12:58:12,684 INFO sqlalchemy.engine.base.Engine (1,)
first
2021-01-05 12:58:12,686 INFO sqlalchemy.engine.base.Engine ROLLBACK


1A ----------------------------
2021-01-05 12:58:12,692 INFO sqlalchemy.engine.base.Engine SELECT CAST('test plain returns' AS VARCHAR(60)) AS anon_1
2021-01-05 12:58:12,692 INFO sqlalchemy.engine.base.Engine ()
2021-01-05 12:58:12,695 INFO sqlalchemy.engine.base.Engine SELECT CAST('test unicode returns' AS VARCHAR(60)) AS anon_1
2021-01-05 12:58:12,696 INFO sqlalchemy.engine.base.Engine ()
2021-01-05 12:58:12,699 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-05 12:58:12,700 INFO sqlalchemy.engine.base.Engine SELECT user.id AS user_id, user.username AS user_username, user.email AS user_email
FROM user
 LIMIT ? OFFSET ?
2021-01-05 12:58:12,703 INFO sqlalchemy.engine.base.Engine (1, 0)
2021-01-05 12:58:12,706 INFO sqlalchemy.engine.base.Engine SELECT user_blog_post.id AS user_blog_post_id, user_blog_post.title AS user_blog_post_title, user_blog_post.body AS user_blog_post_body, user_blog_post.user_id AS user_blog_post_user_id
FROM user_blog_post
WHERE ? = user_blog_post.user_id
2021-01-05 12:58:12,708 INFO sqlalchemy.engine.base.Engine (1,)
first
COMPARE SELECT EMISSION FOR person1 VS person1A
```

I wondered what would happen if I kept both transactions in a single Session but also emulated updates coming into the database from a source other than the ORM. Would it be clever enough to realize that the object it had retrieved and stored in memory was actually out-of-data-now? I modified the code again to run one final test.

I added an input pause between the first and second retrieval transactions. After the first two SELECT queries were complete, I logged into the database and modified the title of the now-retrieved `user_blog_post` object. I then let the second retrieval query execute. SQLAlchemy did not appear to be aware that the record value had changed, because it returned the stored value of the previously-retrieved user_blog_post object.
```python
...
if __name__ == '__main__':

    Base.metadata.create_all(bind=engine)

    newUser = user(username='Moshe', email="moshe@4degrees.ai")
    newUser.blog_posts = [
        user_blog_post(title="first", body="first post body"),
        user_blog_post(title="second", body="second post body")
    ]

    with SQLAlchemyDBConnection() as db:
        # ADD user AND RELATED user_blog_posts TO DATABASE
        db.session.add(newUser)
        db.session.commit()
        
        print("\n\n1 -----------------------------")
        person1 = db.session.query(user).first()
        print(f"{person1.blog_posts[0].title}")

        input("Via SQL CLI, modify the title of the first blog post.")

        print("\n\n1A ----------------------------")
        person1A = db.session.query(user).first()
        print(f"{person1A.blog_posts[0].title}")

        input('COMPARE SELECT EMISSION FOR person1 VS person1A')
```

```sql
sqlite> UPDATE user_blog_post SET title = "ModifiedFirstTime" WHERE id = 1;
sqlite> select * from user_blog_post;
1|ModifiedFirstTime|first post body|1
2|second|second post body|1
3|Person3's Blog|The first blog body for Person3|3
```

```stdout
1 -----------------------------
2021-01-05 15:01:07,432 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-05 15:01:07,433 INFO sqlalchemy.engine.base.Engine SELECT user.id AS user_id, user.username AS user_username, user.email AS user_email
FROM user
 LIMIT ? OFFSET ?
2021-01-05 15:01:07,438 INFO sqlalchemy.engine.base.Engine (1, 0)
2021-01-05 15:01:07,441 INFO sqlalchemy.engine.base.Engine SELECT user_blog_post.id AS user_blog_post_id, user_blog_post.title AS user_blog_post_title, user_blog_post.body AS user_blog_post_body, user_blog_post.user_id AS user_blog_post_user_id
FROM user_blog_post
WHERE ? = user_blog_post.user_id
2021-01-05 15:01:07,447 INFO sqlalchemy.engine.base.Engine (1,)
first
Via SQL CLI, modify the title of the first blog post.


1A ----------------------------
2021-01-05 15:02:54,234 INFO sqlalchemy.engine.base.Engine SELECT user.id AS user_id, user.username AS user_username, user.email AS user_email
FROM user
 LIMIT ? OFFSET ?
2021-01-05 15:02:54,241 INFO sqlalchemy.engine.base.Engine (1, 0)
first
COMPARE SELECT EMISSION FOR person1 VS person1A
```

CONCLUSION: Be aware of your relationship loading techniques and remember that discrepancies can occur if you let solutions other than the ORM interact with your database records (without building in more robust data integrity protections like `expire` and `refresh`. For more advanced query creation techniques, refer to [this article](https://medium.com/python-in-plain-english/relationships-with-sqlalchemy-958b7358e16).

This seems to align with the general best practice that every request a webapp API receives should use its own Session only the for the lifetime of that single http request, but goes further because we've seen how data _within_ that limited lifetime could still become misaligned under the right circumstances.

##### Session Creation
I must talk about Session creation before covering query syntax because it turns out that the way you create a Session affects what query syntax you can use.

As was demonstrated in my previous Database Connection Model [TO DO: ADD LINK] post, I had to refactor the code several times as I struggled to split database functionality (connectivity and object models) from a hard dependency on the initialization of a webapp. Even after had broken the hard dependency, I was still often forgetting to close the database connection once I had completed my submission. 

The problem was resolved by creating a __Context Manager__ that would automatically close the connection for me. I happened to find the method described in Joel Ramos's article["Python Context Managers and the 'with' Statement](https://blog.ramosly.com/python-context-managers-and-the-with-statement-8f53d4d9f87) early in my search and found that it worked well for my mental model, so I opted to implement it. As result, much of the Session creation logic moved from the top-level of the module to within the class that would act as a context manager.

```python
# BEFORE
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine('sqlite:////tmp/declarative_db.db')
session_factory = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Session = scoped_session(session_factory)

# Interact with database via the Session object
...
```
To:
```python
# AFTER
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker


class SQLAlchemyDBConnection(object):
    """SQLAlchemy database connection"""

    def __init__(self, connection_string='sqlite:////tmp/relationshiptest.db'):
        self.connection_string = connection_string
        self.session = None

    def __enter__(self):
        engine = create_engine(self.connection_string, echo=True)
        Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
        self.session = Session()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.session.close()


# INTERACT WITH THE DATABASE WITHIN THE CONTEXT MANAGER
with SQLAlchemyDBConnetion() as db:
    db.session.query(...)
```

Subsequent investigation of SQLAlchemy Sessions revealed alternative methods of context management, so I felt it was important to:
1) Provide links to these other methods for reference purposes
2) Explain the design considerations I chose to adhere by

Other notable sources:
1) [SQLAlchemy Session Basics](https://docs.sqlalchemy.org/en/13/orm/session_basics.html#when-do-i-construct-a-session-when-do-i-commit-it-and-when-do-i-close-it)<br>The SQLAlchemy documentation offer a few different option suggestions, one of which is a turning ather function that handles commits and rollback into context manager via Python's `contextlib` standard library.

2) [Dan Bader's "Context Managers and the 'with' Statement in Python](https://dbader.org/blog/python-context-managers-and-with-statement)<br>This succint article covers different ways to create a context manager including both the technique I followed from Ramos's article and the contextlib method described by SQLAlchemy. 

3) [Oliver Beattie's Github Gist](https://gist.github.com/obeattie/210032)<br>A pure code example, Beattie uses the contextlib method but also has a more complicated setup where he differentiates between a throwaway temporary session for SELECT and a more robust transaction session for INSERT/UPDATE/etc. Furthermore, there is another control function that uses the `functools.wraps` method to wrap a function in a function in a function.

Bader notes in his article _"Both the class-based implementations and the generator-based are practically equivalent. Depending on which one you find more readable you might prefer one over the other. A downside of the @contextmanager-based implementation might be that it requires understanding of advanced Python concepts, like decorators and generators. Once again, making the right choice here comes down to what you and your team are comfortable using and find the most readable."_. This describes exactly why I chose to follow the Ramos method:
* I wanted code that I could read that explicitly described the `__init__`, `__enter__`, and `__exit__` dunders.
* I wanted a context manager that ONLY handled my connection, whereas I would be responsible for flushes/commits/rollbacks separately
* I did not want nested complexity
* I did not want to divert my attention to refresh my knowledge on [decorator 'inside baseball'](https://stackoverflow.com/questions/308999/what-does-functools-wraps-do); I recall looking into `functools.wrap` in the past and struggling to grok the nested function calls (I was already struggling to fully grasp the proper SQLAlchemy patterns and didnt need to make my problems even more numerous!).
* I wasn't convinced I needed Beattie's extra complexity because I thought SQLAlchemy was already handling the transaction management part for me?

I did have one final decision to make with the Ramos approach: would I instantiate a new instance of the class every time I needed a session, or would I instantiate only once and reuse it across the application?
```python
# EXAMPLE OF INSTANTIATION MODELS

class SQLAlchemyDBConnection(object):
    """SQLAlchemy database connection"""

    def __init__(self, connection_string='sqlite:////tmp/relationshiptest.db'):
        # OVERRIDE THE DEFAULT connection_string IF NEEDED
        self.connection_string = connection_string
        self.session = None

    def __enter__(self):
        engine = create_engine(self.connection_string, echo=True)
        Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
        self.session = Session()  # (bind=engine)
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.session.close()
        
# OPTION 1
with SQLAlchemyDBConnection() as db:
    # SQLAlchemyDBConnection.__init__ and .__enter__ executed by this point
    result = db.session.query(someClass).first()
    ....
    
with SQLAlchemyDBConnection() as db:
    # SQLAlchemyDBConnection.__init__ executed again before .__enter__
    result = db.session.query(someClass).first()
    ....
    
# OPTION 2
db = SQLAlchmeyDBConnection()  # CLASS IS INITIALIZED HERE
with db:
    # SQLAlchemyDBConnetion.__enter__ ONLY EXECUTED. DID NOT NEED TO INITIALIZE
    result = db.session.query(someClass).first()
    ...
```

Instantiating the `SQLAlchemyDBConnection` each time meant I could change the `connection_string` as needed, but the downside was a potential loss of efficiency from the extra `__init__` functions. I thought it was unlikely that I would change the connection_string often and - even if I did - Ramos had an easy workaround: just create a different instance for each database! If I needed to interact with two different SQLite files, I could instantiate two instances of SQLAlchemyDbConnection class and provide a different connection string for each. If I needed to interact with SQLite and MongoDB, I could either modify the SQLAlchemyDConnection to accommodate addition mandatory connection information like Port, or I could just quickly create another Context Manager class that was specific to MongoDB.

In the end, I was convinced by the [SQLAlchemy documentation](https://docs.sqlalchemy.org/en/13/orm/session_basics.html#when-do-i-make-a-sessionmaker), which said that a `sessionmaker` should only be made once in the application global scope. Unfortunately, this meant I had to rejig my SQLAlchemyDBConnection class (to move the sessionmaker logic from `__enter__` to `__init__` and also modify how I invoked the class when I wanted to interact with the database. Given that I was reorganizing this class, I also took the opportunity to reorganize how I create `declarative_base` objects as well

```python
# FULL CODE, WITH MODIFICATIONS
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, joinedload

from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

# CREATE THE ROOT OBJECT FOR OUR ORM OBJECTS
Base = declarative_base()

# DEFINE DB CONNECTION STRINGS. ADD ANOTHER STRING FOR EACH TARGET DB.
sqlite_connection_string = 'sqlite:////tmp/relationshiptest.db'


# GENERIC FUNCTION TO CREATE INITIAL TABLES
# WILL NEED TO BE EXPANDED AS OTHER VALUES ARE REQUIRED (E.G. port)
def createBaseTables(base=None, connection_string=None, echo=False):
    # Uses `Base` object that was assigned sqlalchemy.ext.declarative.declarative_base
    engine = create_engine(connection_string, echo=echo)
    base.metadata.create_all(engine)


# GENERIC CONTEXT MANAGER CLASS.
# WILL NEED TO BE EXPANDED AS OTHER VALUES ARE REQUIRED (e.g. port)
class SQLAlchemyDBConnection(object):
    """SQLAlchemy database connection"""

    def __init__(self, connection_string=None, echo=False):
        engine = create_engine(connection_string, echo=echo)
        self.session_factory = sessionmaker(autocommit=False, autoflush=False, bind=engine)

    def __enter__(self):
        self.session = self.session_factory()
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
    # Note: ForeignKey MUST be defined or relationship insert wont work.
    user_id = Column(Integer, ForeignKey('user.id'))

# CREATE THE CONTEXT MANAGERS TO BE USED IN THE REST OF THE APPLICATION. CREATE ONE PER CONNECTION STRING
sqlitedb = SQLAlchemyDBConnection(connection_string=sqlite_connection_string, echo=True)

if __name__ == '__main__':

    # CREATE THE TABLES IN THE SQLITE DATABASE
    createBaseTables(base=Base, connection_string=sqlite_connection_string, echo=True)

    # CREATE TEST RECORD DATA
    newUser = user(username='Moshe', email="moshe@4degrees.ai")
    newUser.blog_posts = [
        user_blog_post(title="first", body="first post body"),
        user_blog_post(title="second", body="second post body")
    ]

    # POPULATE DB WITH TEST DATA
    with sqlitedb as db:
        print('Adding newUser')
        db.session.add(newUser)
        db.session.commit()

    # QUERY DATA
    with sqlitedb as db:
        result = db.session.query(user_blog_post).first()
        user_of_blog = result.user.username
        print(f'user_of_blog is: {user_of_blog}')

        print('accessing user_blog_post data via user')
        result = db.session.query(user).first()
        print('first blog title of first user is: ', result.blog_posts[0].title)
```

I think the changes make the code better organized and more flexible. However, the decision directly impacted my options in the next topic I'll cover: the ORM query syntax.




##### Query Syntax
During my readings, I noticed examples using two simple - but different - query notations: `Model.query(...)` vs `Session.query(Model)...`. Not wanting to devote headspace to remember both syntaxes, I sought an explicit stance on the one-and-only way to execute an ORM query. This ended up being quite easy once I understood the underlying mechanic. (__ANSWER__: Use `Session.query(Model)...`)

The Model query technique appears to simply be a shorthand, and requires one to bind a Session object to your declarative_base object:
```python
# EXAMPLE: MODEL QUERY SYNTAX
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# CREATE SESSION OBJECT
engine = create_engine('sqlite:////tmp/declarative_db.db')
session_factory = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Session = scoped_session(session_factory)

# CREATE DECLARATIVE BASE
Base = declarative_base()

# TIE DECLARATIVE BASE QUERY TO SESSION OBJECT
Base.query = Session.query_property()

# DEFINE SOME TABLE OBJECT HERE THAT INHERITS FROM declarative_base
class someClass(Base):
...

# YOU COULD THEN QUERY VIA:
someClass.query(...)
```
To save a few characters of typing at the start of a database query definition, we require the global declaration of a `Base` and `Session` object, and then build a hard dependency between them. I had just spent alot of effort to *BREAK* hard dependencies and implement factory pattern in my web application, so I wasn't super enthused about reintroducing the pattern if this was the only benefit. Worst of all, I wasn't sure how this was going to impact the Context Manager class I had created to manage my database Session objects - would I need to pass in the `declarative_base` variable on every instantiation (or hardcode it in the `__init__` method?

The alternative approach was to use the slightly more verbose `Session.query(SOME_CLASS)...` method that was already available in the Context Manager class I had created. I didn't need to make any changes to use this technique, and I happened to like it better anyways since it was explicit and consistent across any object I wanted to query for.

I opted to stick with the `Session.query(SOME_CLASS)` approach. I may end up regretting needing to type an additional few characters each time I'm interacting with the database, but I think IDE autocomplete will minimize the pain and the simpler headspace makes this worthwhile.



##### Backref vs Back_Populates
This is yet another issue where "less typing" is pitted versus "clear, explicit definition" to achieve the _same_ ultimate behaviour. 

Let's return to the `user` and `user_blog_post` classes from earlier to illustrate this point:
```python
...
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
    # Note: ForeignKey MUST be defined or relationship insert wont work.
    user_id = Column(Integer, ForeignKey('user.id'))
```
The `blog_posts = relationship("user_blog_post", backref="user")` definition in the `user` class establishes an ORM-based relationship to the `user_blog_post` class, meaning we can use the relationship attribute to:
* Reach from `user` to `user_blog_post`, via `user.blog_posts`
* Reach from `user_blog_post` to `user`, via `user_blog_post.user`.

While there is nothing *technically* wrong with this relationship definition techique, I feel it's an anti-pattern because the relationship is defined entirely within the `user` object; no hint exists in the `user_blog_post` object. For the sake of saving an extra line at design-time, we complicate the effort of later readers/maintainers to understand what's happening in the code (particularly if working with a project with several hundred ORM object definitions).

Rather than use `backref`, I'm making a decisio to always use `back_populates` instead. This establishes the same exact relationship as was established by the `backref` techique, but requires me to explicitly define the relationship *in both* objects. Using the `back_populates` technique, our classes would now look like:
```python
```

Example:
```python
class user(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    username = Column(String)
    email = Column(String)
    
    # Adding relationship linkage
    blog_posts = relationship("user_blog_post", back_populates="user")


class user_blog_post(Base):
    __tablename__ = 'user_blog_post'
    id = Column(Integer, primary_key=True)
    title = Column(String)
    body = Column(String)
    user_id = Column(Integer, ForeignKey('user.id'))
    
    # Adding relationship linkage
    # Note: ForeignKey MUST be defined or relationship insert wont work
    user = relationship("user", back_populates="blog_posts")

```
This technique is a tad more verbose, but explicitly surfaces the relationship in BOTH objects. Just as I argued re: the query technique, I think a bit of extra typing is well worth the extract design clarity.

It is worth further noting a few more issues related to SQLAlchemy relationship management: 
1. The placement of a Foreign Key affects methods SQLAlchemy offers.<br>In our example above, we have a one-to-many relationship between `user` and `user_blog_post`. This means we can use `.append` when attaching blog entries to a user via the `user.blog_posts` relationship attribute, but will see an error if we try to use `user_blog_post.user.append(USER_OBJECT)` (instead we must use `user_blog_post.user = USER_OBJECT`.

1. Orphan Records vs Cascading Deletes.<br>SQLAlchemy will balk if the deletion of a record results in a null value in another table (thereby creating an orphan record). 
Example: We have a Student table, an Activity table, and an Enrollment table which uses the Student.id and Activity.id as a compound key. If we delete a Student record whose Student.id is present in the Enrollment table, SQLAlchemy will return a `AssertionError: Dependency rule tried to blank-out primary key column` error (and refuse to delete the Student record).

We can tell SQLAlchemy to automatically delete any orphans it may generate by adding a `cascade='all, delete-orphan'` option on the relationship. (Further note, as per SQLAlchemy itself `'delete-orphan cascade` is normally configured on the 'one' side of the 'one-to-many' relationship). This is yet again another good reason to use `back_populates` over `backref` as I'm not sure how you would specify the relationship on the 'one' if the backref definition was placed on the 'many'. See more details [here](https://docs.sqlalchemy.org/en/13/orm/session_basics.html#deleting-objects-referenced-from-collections-and-scalar-relationships).

##### Extending the Object Model with Properties
I found this good idea while reading Mike Patterson's [Relationships with SQLAlchemy](https://medium.com/python-in-plain-english/relationships-with-sqlalchemy-958b7358e16_ blog post.

Patterson presents a scenario where a student wants to enroll in various activities offered by their school. We model this as:
```python
class Student(Base):
    __tablename__ = 'student'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    enrollments = relationship(
        "Enrollment", back_populates="student",
        cascade='all, delete-orphan')  # Set cascade to remove orphaned enrollments.


class Activity(Base):
    __tablename__ = 'activity'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    enrollments = relationship("Enrollment", back_populates="activity")


class Enrollment(Base):
    __tablename__ = 'enrollment'
    id_student = Column(ForeignKey(Student.id), primary_key=True)
    id_class = Column(ForeignKey(Activity.id), primary_key=True)
    start_date = Column(Date, nullable=False)
    end_date = Column(Date)
    
    # Because the foreign key is on this table, SQLAlchemy knows
    # these are one to many relationships with Enrollment being the many
    student = relationship(Student, back_populates='enrollments')
    activity = relationship(Activity, back_populates='enrollments')

```
Extending our scenario, let's assume our student has had multiple activities over the years, that those activities have changed as the student's tastes evolve, and that we are a meticulous recordkeeper. This means that our Activity table will be populated with activites that the student used to do, as well as the student still does. Is there a way to easily leverage the already-existing `relationship` but cleanly limit the results without having to specify it in the query each time?

Patterson says yes: **use a property decorator to wrap a function**. Example:
```python
class Student(Base):
    __tablename__ = 'student'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    
    # RELATIONSHIP(s)
    enrollments = relationship(
        "Enrollment", back_populates="student",
        cascade='all, delete-orphan')  # Set cascade to remove orphaned enrollments.
        
   # PROPERTY FUNCTIONS - CREATE TWO FUNCTIONS THAT WILL RETURN ACTIVE AND LEGACY ACTIVITIES
   @property
   def current_activities(self):
        for enrollment in self.enrollments:
            if not enrollment.end_date:
                yield enrollment.activity
            
   @property
   def past_activities(self):
        for enrollment in self.enrollments:
            if enrollment.end_date:
                yield enrollment.activity
                
  # OTHER STUFF HERE LIKE CONTEXT MANAGER AND OBJECT LINKAGE
  ....
  
  # ACCESS DATA THROUGH NEW @property FUNCTIONS
  all_tables = sqlitedb.session.query(Student) \
            .options(
                joinedload(Student.enrollments)
                .joinedload(Enrollment.activity)
            )
            
  for student in all_tables:
        print(f"{student.name} activity: {student.enrollments[0].activity.name}")
        
        # RESULTS RETURNED THROUGH A GENERATOR. MUST COMPENSATE FOR THIS WHEN PRINTING.
        print(f"Former activities are: {[x.name for x in student.past_activities]}")
        print(f"Current activities are: {[x.name for x in student.current_activities]}")
```

I've documented the `@property` method for reference purposes. But I'm not sure how useful it will be in day-to-day use. I originally wondered if it would just be easier to move this filtering logic, but I think that means I would need to either:
1. Do a generic query and post process (akin to what's happening in the `@property` logic), or
2. Run two separate queries, each with its own filtering logic.

I think the benefit here is that we can run the query once, retrieving all the physical results from the SQL database and then use the ORM objects to slice and dice further without needing to run more queries. Time will tell.

##### Multithreading-safe Session
SQLAlchemy warns that a [global Session object IS NOT thread-safe by default](https://docs.sqlalchemy.org/en/13/orm/contextual.html#unitofwork-contextual). This can be mitigate, however, by using `scoped_session()` functionality provided by the package. 

Given that I already built a context manager (which creates a new session each time we enter it), I wasn't sure if I needed to modify my code to include `scoped_session`. A Google search returned poor result set when I searched for "SQLAlchemy context manager vs scoped session" but I did find an article by Xiang Zhu ["Using Python SQLAlchemy session in multithreading"](https://copdips.com/2019/05/using-python-sqlalchemy-session-in-multithreading.html) which suggested that my existing context manager approach was sufficient.

It remains to be seen if I ever need to run a session within another session. In such a case, I think the global scoped_session approach might work, but am not sure about the context manager in a context manager scenario. This seems very edge-case-y to me, however, so I'm going to stick with my existing pattern on the educated guess that it is probably ok.

##### Association Tables
This is the last item to consider before I get cracking. In many-to-many relationships, an association table must be used to track what is linked to what. It is important to note that this can be done in two ways: 
* Use a Table<br>Use this method when you need to link foreign keys only
* Define a new class object<br>Use this method when you need to link foreign keys AND keep track of additional data via columns unique to this table.

The SQLAlchemy docs cover both the [Asssociation Table option](https://docs.sqlalchemy.org/en/14/orm/basic_relationships.html#many-to-many) and [Association Object option](https://docs.sqlalchemy.org/en/14/orm/basic_relationships.html#association-object). For my own benefit, I'm listing their examples here with my own additional descriptions and modifications for clarity:

```python
# ASSOCIATION TABLE EXAMPLE (BIDIRECTIONAL RELATIONSHIP VIA back_populates)
from sqlalchemy import Column, Integer, ForeignKey, Table
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

association_table = Table('association', Base.metadata,
    Column('parent_table_id', Integer, ForeignKey('parent_table.id')),
    Column('children_table_id', Integer, ForeignKey('children_table.id'))
)


class Parent(Base):
    __tablename__ = 'parent_table'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    children = relationship(
        "Child",
        secondary=association_table,
        back_populates="parents")


class Child(Base):
    __tablename__ = 'children_table'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    parents = relationship(
        "Parent",
        secondary=association_table,
        back_populates="children")
```
Now lets create some records and add them to the database:
```python
...
with sqlitedb:
    p1 = Parent(name="Parent1")
    p2 = Parent(name="Parent2")
    c1 = Child(name="Child1")
    c2 = Child(name="Child2")
    
    p1.children.add(c1)
    p1.children.add(c2)
    p2.children.add(c1)
    
    sqlitedb.session.add_all[p1, p2])
    sqlitedb.session.commit()
```
This results in the creation of the following SQLite tables and records, showing that the association table is being automatically created:
```sql
SQLite version 3.31.1 2020-01-27 19:55:54
Enter ".help" for usage hints.
sqlite> .tables
association  parent_table  children_table

sqlite> select * from parent_table;
1|Parent1
2|Parent2

sqlite> select * from children_table;
1|Child1
2|Child2

sqlite> select * from association;
1|1
1|2
2|1

```
If we were to try to retrieve the children of Parent1 via ```python children_of_p1 = p1.children```, we can see the assocation table is used to limit the records that are retrieved from the children table:
```sql
2021-01-11 11:02:21,459 INFO sqlalchemy.engine.base.Engine BEGIN (implicit)
2021-01-11 11:02:21,460 INFO sqlalchemy.engine.base.Engine SELECT parent_table.id AS parent_table_id, parent_table.name AS parent_table_name
FROM parent_table
WHERE parent_table.id = ?
2021-01-11 11:02:21,462 INFO sqlalchemy.engine.base.Engine (1,)

2021-01-11 11:02:21,465 INFO sqlalchemy.engine.base.Engine SELECT children_table.id AS children_table_id, children_table.name AS children_table_name
FROM children_table, association
WHERE ? = association.parent_table_id AND children_table.id = association.children_table_id
2021-01-11 11:02:21,468 INFO sqlalchemy.engine.base.Engine (1,)
```

Now lets assume that we made a data entry mistake and it turns out Parent1 isn't actually related to Child2. We would fix this in the Python code by breaking the relationship with ```python p1.children.remove(c2)``` which translates to the following SQL being emitted by the ORM:
```sql
2021-01-11 11:06:47,835 INFO sqlalchemy.engine.base.Engine DELETE FROM association WHERE association.parent_table_id = ? AND association.children_table_id = ?
2021-01-11 11:06:47,836 INFO sqlalchemy.engine.base.Engine (1, 2)
2021-01-11 11:06:47,838 INFO sqlalchemy.engine.base.Engine COMMIT
```
The documentation covers further scenarios like "what if i wanted to delete the child object directly?". I appears that my decision to use `back_populates` everywhere makes this possible (since relationships are comprehensively defined). I'll investigate more once I actually encounter this situation in the while, because I want to move on to the Association Object now.

Returning to our example, lets assume that we need to collect some additional data like whether a Parent is the biological parent or a step parent. Since an individual can be the biological parent to one child, but a step-parent to another, it makes the most sense to capture this element in the Association table between Parent and Child. Unfortunately, the need for extra columns means we can't use the Table method, but must now create a full-fledged Association Object and modify the relationship definitions so that the Parent and Children tables point directly to the Association Object, rather than to each other.

```python
# ASSOCIATION TABLE EXAMPLE (BIDIRECTIONAL RELATIONSHIP VIA back_populates)
from sqlalchemy import Column, Integer, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Association(Base):
    __tablename__ = 'association2',
    parent_table_id = Column(Integer, ForeignKey('parent_table.id'), primary_key=True)
    children_table_id = Column(Integer, ForeignKey('children_table.id'), primary_key=True)
    extra_data = Column(String(50))
    # ESTABLISH RELATIONSHIPS
    child = relationship("Child", back_populates="parents")
    parent = relationship("Parent", back_populates="children")


class Parent(Base):
    __tablename__ = 'parent_table'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    children = relationship("Association", back_populates="parent")


class Child(Base):
    __tablename__ = 'children_table'
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
    parents = relationship("Association", back_populates="child")
```

With the redefinition of the classes and relationships complete, we can now start creating objects and linking them together. The SQLAlchemy example continues with a code snippet:
```python
p = Parent()
a = Association(extra_data="step-parent")
a.child = Child()
p.children.append(a)

for assoc in p.children():
    print(assoc.extra_data)
    print(assoc.child)
```

I believe the example is attempting to demonstrate that:
1. Because these are pure Python objects at this point, you can slowly populate the attributes over time (unlike with writing the MYSQL record where you must have all the mandtories at write-time).
2. You can chain together shorter relationships into a longer relationship chain. 

While the SQLAlchemy flow *does* work, I found the style to be counter-intuitive and spent longer than I would care to admit trying to wrap my head around the inter-linkage sequence. I found I preferred using a slightly more verbose style that made the Assocation the focal point of the relatinoship definition:
```python
# EXAMPLE
a = Association(extra_data="BiologicalParent")
p = Parent(name="ThisParent")
c = Child(name="ThatChild")

# LINK OBJECTS USING ASSOCIATION AS FOCUS
a.parent = p
a.child = c

# ASSUME WE WANT THE LAST PARENT IN THE TABLE AND WANT TO KNOW WHO THEIR CHILDREN ARE AND THE RELATIONSHIP TO THEM
with sqlitedb:
    # TO GET LAST ENTRY, order_by DESCENDING THEN USE first().
    p = sqlitedb.session.query(Parent).order_by(Parent.id.desc()).first()
    
    for assoc in p.children:
        print(f"{p.name} is a {assoc.extra_data} to {assoc.child.name}")
```
```stdout
> ThisParent is a BiologicalParent to ThatChild
```

##### Feature to be Mindful of: Association Proxy
I documented the object linking method that I intuitively prefer, but must admit that the method is a bit verbose when trying to access attributes via relationships (e.g. `objA.relationship.relationship.objB.attribute`). 

SQLAlchemy offers a shortcut feature called the [Association Proxy](https://docs.sqlalchemy.org/en/13/orm/extensions/associationproxy.html). The documentation describes it as, _"It essentially conceals the usage of a 'middle' attribute between two endpoints, and can be used to cherry-pick fields from a collection of related objects or to reduce the verbosity of using the association object pattern."_ Basically, in addition to defining a relationship from one object to another, I can also define other rules like "when I call this attribute, I want you to pull it from the object that is linked via the relationship that is also defined in this object".

I can see the value of less typing (I'm alreadying beginning to question my decision to use `sqlitedb.session...` instead of just `session...` BUT I feel that these shortcuts place a higher demand on my headspace resources. Explicit may be longer, but I can clearly follow it from start to finish. Shortcuts like the association proxy may make my code less verbose but I also need to devote headspace to remember that I had built shortcuts in the object model (which I am executing elsewhere). As a result, I've noted the feature as a reminder to myself to revisit once I'm more comfortable with SQLAlchemy object interactions, but will refrain from using right now to minimize the amount of potential debugging I will need to do in my beginning code base.

##### Warning: Beware Style Clashes
I had another `relationship` example open in my browser, so I decided to reinforce the concept by trying this one as well. I found some stylistic differences that I didn't gel with the official SQLAlchemy documentation (but still worked), so I thought it was worth calling out. 

You can find the example [here](https://www.pythoncentral.io/sqlalchemy-association-tables/). The code in question is:
```python
from sqlalchemy import Column, DateTime, String, Integer, ForeignKey, func
from sqlalchemy.orm import relationship, backref
from sqlalchemy.ext.declarative import declarative_base
 
Base = declarative_base()
 

class Department(Base):
    __tablename__ = 'department'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    employees = relationship(
        'Employee',
        secondary='department_employee_link'
    )
 
 
class Employee(Base):
    __tablename__ = 'employee'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    hired_on = Column(DateTime, default=func.now())
    departments = relationship(
        Department,
        secondary='department_employee_link'
    )
 
 
class DepartmentEmployeeLink(Base):
    __tablename__ = 'department_employee_link'
    department_id = Column(Integer, ForeignKey('department.id'), primary_key=True)
    employee_id = Column(Integer, ForeignKey('employee.id'), primary_key=True)
    extra_data = Column(String(256))
    department = relationship(Department, backref=backref("employee_assoc"))
    employee = relationship(Employee, backref=backref("department_assoc"))
    
```
The Department-Employee example differs from the official SQLAlchemy Object association in two ways:
1. It uses the `secondary` parameter in the relationship definitions tying the Department and Employee classes to the Association Object class;
2. It retains a direct relationship between the Department and Employee classes.

The result is a solution which more closely resembles an Assocation Table / Association Object hybrid rather than pure solution. Given that the example references a previous post which had the reader create an Association Table (with the post I was following providing direction on how to change the Association Table into an Association Object), my first thought was that the authors had missed a few necessary changes. However, the documented code DID work when I executed it, so this was a bit puzzling. 

It's possible that I'm missing a minor behavioural nuance here, but I don't understand what the hybrid does that makes it better (if anything) than the pure Association Object example from SQLAlchemy. My takeway here is that I stick with the non-hybrid approach, but need to remain aware of this other possible implementation when I'm reading 3rd party code.

#### Naming Convention Practices
Always use Association Object. Always name Association Object as composite of the two objects' full names plus "assocation"

Object A : User
Object B : Message
Object C : UserMessageAssociation

Lookups always have Lookup at the end.
all relationship variables prefixed with "rel_"

# Actual database columns must reference database table names (foreign keys)
# relationships reference Python class objects
# When I link the objects in the Python code, I MUST do it on the RELATIONSHIP attributes,
# NOT the database columns!!!!!
# eg:

#   msg_type_global = MessageTypeLookup(type='Global')
#   msg_type_personnal = MessageTypeLookup(type='Personal')

#   msg1 = Message(rel_message_type=msg_type_global, text="This is message #1")
#   msg2 = Message(rel_message_type=msg_type_global, text="This is message #2")


Next: [Database Connection Pattern](./08-database-connection-pattern.md)<br>
Previous: 
