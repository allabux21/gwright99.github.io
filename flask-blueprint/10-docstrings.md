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

Normally I would have said "to hell with these purity priests!" and just implemented a documentation style that met my own needs. I was being a little cautious, however, because I wanted my code to be able to integrate with the Python ecosystem's documentation generation packages and also be able to leverge the functionality of `doctest` [TODO: find link] - a package which crawls a code package's docstrings and executes any of the code examples contained within to see if they actually work.

Ultimately, I chose to use LSST as my reference implementation because I liked that they were opinionated and had very comprehensive examples, and because they chose to implement the Numpy system (which I happened to find personally appealing). However, I also did a half-assed job implementing this documentation style into my code because:
1. Full compliance broke VSCode (see next section), and
2. Writing fully-fleshed documentation takes *major* time and effort. I wasn't convinced my data model was solid yet, so it was possible additional changes would invalidate large swathes of what I had already written.

##### Mitigating the Display Quirks of VSCode
The [Google](https://realpython.com/documenting-python-code/#google-docstrings-example)- and [Numpy](https://realpython.com/documenting-python-code/#numpyscipy-docstrings-example)-style docstrings make heavy use of whitespace and tabs to create clear, clean documentation blocks (a major reason why I prefer them to the more compact but IMHO less legible reStructured and Epytext styles).

The whitespace made reading easier when directly examining the source code, but displayed as a unformatted wall of text when displayed in an Intellisense pop-up window by VSCode. After another round of investigation, I discovered that [VSCode renders docstrings as Markdown](https://github.com/microsoft/pylance-release/issues/48) (with VSCode itself driven by some setting in the Pyright package), and there appeared to be no way to tweak settings to easily fix the problem.

TODO: INSERT SIDE-BY-SIDE OF CLEAN SOURCE CODE VS VSCODE DISPLAY.

So now I had another problem decision to make:
1. *I could change nothing.*<br> This would keep me compliant to Python documentation standards at the expense of usable Intellisense popups. I considered this a bad choice because I was going to be the only user of the documentation for the foreseeable future, so why was I going to make my immediate life more difficult at the expense of an abstract best practice and *potential* future integration with docstring testing tools?

1. *I could change documentation styles*<br>I could avoid the documentation bloat and (maybe) VScode whitespace display problem by abandoning my preferred Google/Numpy documentation style in favour of the more compact reStructured/Epytext style. I did not favour this choice because it didn't guarantee a display fix but definitely meant I was going to have to use a documentation style which I clearly disliked.

1. *I could manually insert whitespace indicators*<br>By manually inserting `\n` and `\t` in my docstrings, I could force the VSCode rendering engine to display the text with greater fidelity to how it was formatted in my classes. This was non-ideal however, as adding the whitespace indicators cause the VSCode Intellisense popup to have _other_ display issues, plus it meant my docstrings were now littered with escaped whitespace characters (causing further size bloat and being less visually appealing).

None of my options were ideal. In the end, I used a hybrid that touched upon options 2 and 3:
* I used Numpy-style headers to visually break up the sections, but Google-style Attribute notation for great compaction.
* I avoided using `\t` to minimize visual clutter, but still added `\n` to force carriage returns.

##### Balancing immediate vs long-term absract needs 
I have no illusions that this is a perfect solution. I made the decision to implement this way with the resigned understanding that I will likely need to revisit the problem at a later point (either when VSCode solves the display problem, I want to build in automated documentation testing, or if I ever decide to release this project publicly).

This decision is ok, however, because the major problems are LATER. Obviously I would prefer to get things right on the first try and minimize the refactoring debt that I accumulate, but it was clear that there was no solution that would work both technically and culturally (i.e. serve my individual needs). This is still a learning project devoted 100% to enhancing my own skillset and understanding. If that means I have to deliberately break a few Python conventions over the short-term in order to achieve greater results, the trade-off seems fair based on the knowledge I have right now.


### Naming Convention Practices
With the docstrings semi-resolved, I could then turn to the original question that spawned this whole problem in the first place: how should I name my ORM classes and their variables?

As noted at the beginning of this post, I found that adding `_rel` as a prefix to any relationship variable was a useful self-documented reminder as I interacted with the class. Ensuring my docstrings had a good example at very top meant I had removed the need to add further details into the variable name itself - I could just hover the mouse and reference the Intellisense documentation instead. 

I still needed to figure out a clear system for how to name the objects themselves though. Based on my initial rudimentary coding efforts, I had already identified the following considerations:
* Singular vs Plural
* Abbreviations vs Full Text
* Table Purpose

_NOTE: I'm absolutely sure best practices have been documented a million times over on this topic. I was just so tired of solving other problems that I couldn't bear to go down another rabbit hole of stylistic argumentation, so I decided I would just rely on my own organic experience with the understanding that any mistake made would be useful for future learning and growth._

_SECOND NOTE: I am not super well-versed in the jargon of SQL, so expect barbarian-like language as I discuss concepts that likely have a specific and accurate term that I am unaware of._

##### Singular vs Plural (Decision: Singular)
I dithered about this for awhile. Mentally, I was constantly thinking about the SQL tables as plural (e.g. "Users", "Messages"). But from a code perspective, I was creating and linking individual object instances. Furthermore, the plural didn't work well once you add association and lookup tables into the mix.(e.g. "MessageType" seems more correct than "MessagesType"). For these reasons, I settled on using the singular to naming my classes and tables.

##### Abbreviation vs Full Text (Decision: Full Text ... I think ...)
I've repeatedly stated my preference for longer, more explicit names despite the additional typing and memory footprint costs. I'm still holding to this maxim, but must admit that I'm starting to become a little less certain as I go through the act of writing source code. Auto-complete is a godsend, but one still needs to find/select the right variable and there is no way to avoid the increased need for screen real estate as longer variables are used. I've decided to stick with full text for now (understanding that I can back out this decision with aggressive find-replace action later). I may change my mind later.

##### Table Purpose
This was easy in some senses: "Lets use a `User` table to store users!". 

I found this became less clear when I needed to create support tables as I tried to normalize my data. For example, was calling a table `MessageType` clear enough, or should I further qualify it by specifically identifying it as `MessageTypeLookup`? I personally found that the 'Lookup' part made it immediately clear (to me at least) that this was a small table that only existed to extract and normalize some data elements that previously lived on the Message table, but maybe the mere presence of "Type" would have been enough for a more experienced database user? What if I needed to change my data model in the future and this Lookup table was no longer a true lookup table - would I need to go back and change the name and all associated references? 



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
