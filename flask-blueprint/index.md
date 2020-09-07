## Flask Blueprint Project
This is the main page for my Flask blueprint project.

### Why I've embarked on the project
I am a liberal arts graduate (and self-taught programmer) who ascended to a Solution Architecture role via business system analysis rather than the more traditional developer path. My background has generally equipped me well for the job but I constantly try to improve my technical skills and knowledge to compensate for the lack of official developer experience.    

One of my favoured education methods is to reproduce interesting programming tutorials in my local development environment. Although this approach is easy to initiate and has minimal entry barriers, I've noticed that I repeatedly encounter the same kinds of problems once I progress far enough, like:

1. _How do I manage my code so that I'm not afraid to enhance/refactor it?_
1. _How do I get this code off my local machine and some place else where others can see it?_
1. _How am I supposed to Dockerize my application when VirtualBox and Docker For Windows dont play nicely?_
1. _How should I acquire scientific programming libraries? Do I proliferate Python versions and use Anaconda, or use pip and endure Windows compiler hell?_
1. _Omg why is pytest failing to import my code modules **again**?!_

Ultimately, the problems all centred around one simple fact: 
**I did not have a stable, reuseable project template that handled these problems from the outset**.

Theoretically, if I fix the template problem, I stop worrying about the infrastructure and focus exclusively on my code.

### Project Goals
By working on this project, I hope to accomplish the following:
1. Settle on a development environment that:
    1. Removes the aggravation of Python development in a Windows environment.
    1. Allows me to work locally but can migrate to the cloud effortlessly.
    
2. Establish a project template that:
    1. Has its dependencies isolated from other projects on the system.
    1. Comes with a pre-configured linting solution.
    1. Comes with a pre-configured testing solution.
    1. Comes with a pre-configured database solution.
    1. Comes with a pre-configured containerization solution (Dockerfile and secret management).
    1. Comes with a pre-configured Github Actions integration (for CI/CD and automatic deployment to a cloud-hosting service).
  
3. Define the processes and supporting configuration scripts necessary to easily deploy the template into a newly-instantiated local project.


