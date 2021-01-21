## Set Up Testing

I had finally managed to get my rudimentary application working again with the newly-implemented ORM classes. I immediately encountered another problem, however, as the verification and bug fixing of my "plumbing" code required me to repeatedly manually reload, UI click, and data enter. I realized I needed some basic automated testing to make this chore less onerous.

Automated testing seems to have a mixed reputation based on the articles and forum comments I've seen. Everyone agrees that some automated testing is essential in order to ensure your application isn't a giant dumpster fire, but the problem seems to be defining the right balance between the extra work required to create and maintain the test cases versus the value their results provide. 

I personally experienced this frustration in a previous project where it felt like any change I made to my application source code required at least three times as long to implement the requisite updates in the assocated test cases. I didn't mind doing the updates the first few times but I started to become enraged over time as I felt I was spending all my time updatign the stupid test cases rather than finishing off the functionality of the service (which is what I really cared about).

So ... I'm going to approach this topic with an open mind and see where we end up, but can definitely say there's no way in hell I'll be following a Test-Driven Design model.

### Creating the Basics
At the very least, I needed some automated tests to quickly ascertain that the Flask Application could actually instantiate (a failure I had already encountered numerous times as typos killed the interconnected series of imports that executed within the `__init__.py` file.

I needed to be mindful of a few items when deciding how I would approach the implementation:
* My application used the Flask Application Factory model (_which would impact how I executed instantiation in the test cases_)
* I was using SQlite right now but likely would use a different database later (_I could shortcut my database tests with an in-memory SQLite db but that option might not be avaiable later with its replacement).
* I was undecided whether I wanted to leverage the `Flask-Testing` module (which would simply life for me) versus directly using `PyTest` (which would require more work and problem solving but also give me a deeper understanding of the underlying mechanics).
* My Application and database models were still somewhat fluid, so I needed to make sure the testcases were not too tightly coupled to the existing state of the code (but clearly needed some level of adherence or the entire point of testing was moot).


Next: TBD
Previous: [Docstrings and Naming Conventions](./10-docstrings.md)<br>
