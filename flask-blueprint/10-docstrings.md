## Docstrings and Naming Conventions

The revisitation of ORM functionality ended up being a long (but necessary, I feel) diversion from the main project. Now that i had the rudimentary basics under my belt, it was time to get back to the project itself. I moved back to the working source code I had drafted 6 months ago and ... immediately started struggling with the ORM objects again. :|

My initial problem that I was trying to build a data model that was slightly more complex than the examples. My ORM relationship examples used "clean" examples, where tables intersected on a single ForeignKey. The mock example I had chosen to build in the sample project was a very crude messaging system where a Message object could be sent by one User to another User. This meant that any Association Object record that I created needed to have TWO ForeignKeys to the User table. I actually had already managed to get this working in July 2020, but I had had to modify the relationship slightly based on some answers I found on StackOverflow (_the `relationship` definition of the Association Object needed to include a `foreign_keys` paramater linking the relationship back to the specific Association Object attribute_).

There were three bigger problems at play, however, and these required more thought:
1. I was continually mixing up ORM object attributes that mapped to real SQL columns versus attributes that existing only as a software relationship.
2. I couldn't settle on the level of self-documentation my classes and variables should have.
3. I was neglecting my ORM object docstrings and this was impacting my implementation efforts.

Although I have listed the above issues as three separate problems, my remediation efforts led me to treat them as inter-related. I'll need to meander for a few minutes to explain my train of thought.

My original and immediate problem was to find an effective way to mentally keep track of the ORM attributes values that I needed to explicitly supply versus those that would be handled by the ORM when it was submitting records in a Session (e.g. dynamic population of foreign key values in the User-Message Association Object record as it was written to SQLite). 

My project had four ORM classes at this point:
* User
* Message
* MessageType
* UserMessage

The User class attributes initially looked like this:
```python
class User(TimestampMixin, UserMixin, Base):
    __tablename__ = "user"
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String, nullable=False, unique=True)
    email = Column(String, nullable=False, unique=True)
    password = Column(String(200), unique=False, nullable=False)

    messages_sent = relationship(
        "UserMessage",
        foreign_keys='UserMessage.sender_id',
        back_populates='message_sender')

    messages_received = relationship(
        "UserMessage",
        foreign_keys='UserMessage.recipient_id',
        back_populates='message_recipient')
...
```
While I had been clever enough to add a super helpful blank line between the SQL column-linked attributes and the relationship-linked attributes, I had neglected to include anything in the names that could serve as a self-documenting hint when actually trying to create these objects elsewhere in the program. This meant that I had to constantly refer back to User class code for reminders as I was trying to create an initial set of test data. I run a 3-column VSCode setup on a 40-inch 4K monitor so this was not the absolute end of the world, but this still limited which files I could have actively displaying at any given time.

I decided to solve my problem by appending a `rel-` prefix to all relationship-linked attributes. This would differentitate the relationship attributes from the column-linked attributes without introducing too much additional verbosity and visual clutter (I was starting to pay more attention to this given my previous decision to use more verbose Session dot-notation habits which I began to regret immediately once I started actively developing).

The prefix change made things easier but then begged another question: "Should the relationship variable also include a self-documenting hint to the Class it is linked to?". My example was quite simple and already had objects with 3 relationship - what was it going to be like if/when I had far more complex objects?

I started implementing a change where my relationship variables looked like `<PREFIX>-<OTHER_OBJECT>-variablename`. This was non-ideal, however, because I had some long class names (more on that later) and it was causing the variable to become quite long. Furthermore, because I wasn't fully committed to my data model yet, classes and their associated attributes were still somewhat fluid. If I started baking class names into my variables, I risked heavier refactoring and troubleshooting if I decided that Object modifications were necessary. Granted, a global find-replace execution would get most of the work done, but these classes were all using common generic words like "id" and "message". Find-replace was not going to be able to do 100% of the job.

As I mulled the problem, I became more convinced that baking the object name into variable name was majorly stupid not only because of the points already raised, but also because I ALREADY had the target object defined within the relationship definition! If I could find a way to surface this information in the Intellisense hover tooltip, I could keep the variable more abstract and less verbose while still having access to the information I needed. 

This, however, was not as easy as I thought it would be. I was now stumbling into the horrible world of ... DOCSTRINGS. 

#### Problems Problems Everywhere
I have a confession to make: I have never used docstrings. I understood them conceptually, and that they were a best practice because ... somehow ... they helped others read and use your code. But almost all of the Python source code I've written has only been seen by one person: me. 

I've obviously done code reviews with peers when a solution I've written is about to be unleashed in Production, but these always happened via a line-by-line review of the code on my machine. Given the reality that none of my employers have been Python shops, I've relied on PyInstaller to create an .exe file whenever a program I've written needed to be run by someone else. As a result, there's never been a need to make my source code easily accessible to another human being (and has allowed me to avoid spending time on docstrings). Unfortunatey, this lack of experience was now impacting a VIP: me! A fix was necessary.

Conceptually, docstrings are quite easy: provide a description, list the parameters/attributes/functions, identify what response is returned, and call out important caveats. In practice, however, I found myself grappling with the following problems:
1. Writing clear and useful documentation in my ORM classes without it resulting in a 10:1 ratio of explanation to actual code.
2. Choosing the appropriate documentation formatting style.
3. Mitigating the display quirks of VSCode
4. Balancing my immediate needs as a learning programmer versus the more abstract desire to adhere to best pratices.

##### Writing Clear and useful documentation
I like comprehensive documentation that provides good descriptions and plenty of examples but find that I react badly to needing to scroll the if I want to jump between reading the documentation and examining the that is being described. Unfortuately, after description text reaches a certain critical mass, there is just no way to avoid the need to scroll. Or was there?! 

As I mentioned earlier, I run a 3-column view in VSCode. This has allowed me to put interrelated objects side-by-side in the different panes and follow a call flow from one file to the next. Furthermore, I was already subdividing my project into functional areas like 'app', 'blueprints', 'database', and 'configuration' so I wondered if it would make sense to create my docstrings as constants in a 'documentation' folder and simply import these variables into its associationed class. This way I could ensure there was still documentation, but it would be organized in a way that felt organizationally cleaner and allowed me to more quickly find class code.

In theory, the execution should have been simple: just modify the class's `__doc__` property with the text I wanted and voila, you are done! The problem was that I couldn't find any examples that showed how to set this value inside a class BEFORE it was instantiated! The limited number of articles returned by Google either had the change being made after instantiation, or invovled a hinky function-in-a-function decorator pattern that I didn't want to have to figure out. To make this even more difficult, I was determining my success based on whether VSCode would display my externalized doctstring when I hovered the mouse over a reference to the class in a different file. I later found out VSCode had several Python docstring display problems, so it is possible that I DID get the externalization working but ... I feel the lack of useful articles on the topic makes it more likely that it is just not possible.  

After a few hours of struggling, I decided that externalizing the docstring was not worthwhile and chose instead to implement the docstring in a standard manner: as a deluge of text at the start of the class.

##### Choosing the appropriate documentation formatting style
I had chosen WHERE I would document the class but I still hadn't decided HOW, and this became my next challenge. My Google search had identified several different docstring formatting systems and plenty of unresolved diagreements between various authors.

One very opinion article said to only use Sphinx style because that was the official Python method. Unfortunately the article was several years old and I found found Sphinx-style to be the least visually accessible method, so that was ignored. Another article suggested either Google or NumPy, suggesting that Google was good but Numpy was better if your text was longer. Unfortunately his examples were super short so there wasn't any noticeably compelling difference between the two formats other tha some minor differences on how headers were identified and where carrige returns were placed.

Normally I would have said "to hell with these purity priests!" and just implemented a documentation style that met my needs. I was being a little cautious, however, because I wanted my code to be able to integrate with the Python ecosystems's documentation generation packages and also be able to leverge the functionality of `doctest` [TODO: find link] - a package which crawls a code package's docsrings and executes any of the code examples contained within to see if they actually work.

Ultimately, I choose to use LSST as my reference implementation because I liked that they were opinionated and had very comprehensive implementation descriptions. Unfortunately, I also did a half-assed job of it because:
1. Full compliance broke VSCode (see next section), and
2. Writing fully flesh documentation take major time and effort, and I wasn't convinced my data model was solid yet so it was possible I was going to have to make changes that invalidated the hours of writing I was doing to document the AS IS.

##### Mitigating the Display Quirks of VSCode




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


DOCSTRING CONVENTIONS (VSCODE, useing '\t' causes display errors in Pylance).
Use of Docstring help to avoid needing extra long variable names (eg. rel_message_sender instead of rel_User_message_sender)
https://realpython.com/documenting-python-code/
Some display issues with Pylance. CHoice - write docstrings that looks good in VSCode (Markdown) or write docstrings that are compliant to the Python tooling (Sphinx, Google, etc). https://stackoverflow.com/questions/6046263/how-to-indent-a-few-lines-in-markdown-markup
https://github.com/sublimelsp/LSP-pyright/issues/42

Decision - follow Numpy style but make accommodations for VScode to make it usable (keep `\n and \t and code blocks`).
Did not find way to move docstrings out the class itself.

Next: TBD<br>
Previous: [Beef With ORMs](./09-orm-beef.md)<br>
