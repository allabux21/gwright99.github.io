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
1) I WOULD use the ORM as my database connection solution.
2) I WOULD model database tables as ORM objects rather than raw SQL statements.
3) I WOULD use ORM syntax to for CRUD operations against the database.<br>
4) I WOULD NOT use relationships in my ORM objects.
5) I WOULD NOT use shorthand syntax (e.g. `Base.query = Session.query_property()`) to minimize typing.
6) I WOULD NOT use cascading operations (i.e. I want the ORM to only execute specific operations that I specifically order it to do, not extra stuff that it opaquely doing on my behalf).

My logic was thus:
* I had working code and it would be crazy to scrap it.
* Scrapping the ORM-based logic would require complete rework on the database-connectivity side but also break the already-working account login functionality.
* The ORM is capable of executing raw SQL but raw SQL can't provide ORM capabilities, so it was better to keep the ORM in the event it was discovered later that ORM functionality was desperately needed.
* I am not comfortable relying too much on opaque ORM functions, particuarly when I don't have a good grasp on the underlying process/mechanism the ORM is managing for me. As a result, its power should be used sparingly.
* Relationships (in this case ORM, but equally applicable to real-life) were a major and continual source of confusion for me. If I simply ignored the ability to use object relationships, I could still leverage the ORM parts that I wanted AND adhere more closely to how a pure SQL solution might look.

I may come to regret these decisions in future, but they were the decisions that were needed now. Assuming I continue to make forward projet, I'll revisit this post and provide an update on how which decisions good and which ones ended up being terrible.

Next: [Database Connection Pattern](./08-database-connection-pattern.md)<br>
Previous: 
