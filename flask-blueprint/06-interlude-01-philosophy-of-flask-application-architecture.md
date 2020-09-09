## Interlude 01 - The Philosophy of Flask Application Architecture
So we've picked our technology components and configured our tooling - time to get coding, right!? Not quite.

I specifically chose Flask over Django because: <br>
(1) Flask is an unopinionated framework that grants me a greater degree of control over my implementation, and <br>
(2) Flask's design makes it very easy to stand up a microservice with just a few lines of code in a single `.py` file.

These are amazing features which work very well with Rapid Application Development, an application development methodology that I favour. Being able to move quickly and delivery results promptly builds trust with your stakeholders and helps ensure you are actually building the solution that is needed rather than risk encountering a horrible surprise close to the end of a project.

This flexibility and ease-of-entry, however, come at a great cost: the internet abounds with reams of badly-designed Flask tutorials which work for the simple needs of the tutorial article/series but cannot hope to scale appropriately should the developer ever want to move beyond a small-scale proof-of-concept stage. Given how easy it is to write a 'me-too' Introduction to Flask article in the pursuit of eyeballs and clicks, the proliferation of junk examples is not the least bit surprising due to Python's immense growth in popularity [ADD SOURCE].

_"Way to be a hypocrite, Graham!"_ you may be saying. I'm writing my own series on Flask, have never deployed a Flask-based Python application into Production, and haven't had to keep the infrastructure running under heavy load; so what the hell do I know? To that I say "_Dude, it's a free blog that I'm mostly writing for myself - if you dont like the ten minutes or so of content, just stop reading!_".

I will be the first to admit that I don't have all the answers (to be fair, I dont think _anybody_ has all the answers). With that said, the output of this project is meant to help me quickly and easily spin up future projects, so I'm well-motivated to build something that is cohesive, scaleable, and adheres to defensible best practices. I'll share my opinions, provide links and background to the material I reviewed that led me to my conclusion, and let you decide for yourself. Eventually, once I figure out how to use GitHub Pages, I may even throw caution to the wind and enable  anonymous internet strangers to brutally comment on all the mistakes and philosophical decisions I've made (_but maybe not that soon!_). 

With that said, onto my opinions ...







Defining a proper design is crucial for me purposes.

    
<br><br>
Previous: [Install and Configure VS Code](./05-install-and-configure-vscode.md)<br>
Next:
