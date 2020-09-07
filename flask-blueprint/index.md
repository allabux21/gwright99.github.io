## Flask Blueprint Project
This is the main page for my Flask blueprint project.

### Why I've embarked on the project
I am a liberal arts graduate who is a self-taught programmer and ascended to a Solution Architecture role via a Business Analysis route rather than the more traditional developer path. While I believe that my background has generally equipped me well for the job (_something I may choose to explore in another article series_), I recognized that the lack of years as a hardcore developer needed to be compensated or else I risked being hoodwinked when discussing physical implementations and not properly identifying technical process implications for some architectural decisions.    

I regularly read and try interesting programming tutorials in an effort to expand and update my technical skills. As a result of growing up in the pre-cloud era, however, I still favour working in a local environment (_sometimes a Linux VM on a Windows host, sometimes directly on Windows_). Although this approach is effective for getting started quickly (_pip install a few packages, start coding!_), I have noticed that I continually encountered the same problems like:

1. How do I manage my code so that I'm not afraid to enhance/refactor it?
2. How do I get this code off my local machine and some place else where others can see it?
3. How am I supposed to Dockerize my application? I built it in a VirtualBox Ubuntu VM, but the Windows Docker installation requires Hyper-V (_which will kill VirtualBox_).
4. Oh great, a scientific programming package. Do I install Anaconda so I can get the package without going through Windows compiler dependency hell, or do I hope I can figure out how to get the pip installation working this time?
3. Omg, why is pytest failing to import my code modules _again_?!

Ultimately, the problems all centred around one simple fact: **I did not have a stable, reuseable project template that handled these problems from the outset**.
Theoretically, if I fix the template problem I can stop working about my infrastructure and focus on all the reasons my code was broken instead!

### Project Goals

By working on this project, I hope to accomplish the following:
1. Settle on a development environment that:
  i. Removes the aggravation of Python development in a Windows environment.
  ii. Allows me to work locally but can migrate to the cloud effortlessly.
    
2. Establish a project template that:
  i. Has its dependencies isolated from other projects on the system.
  2. Comes with a pre-configured linting solution.
  3. Comes with a pre-configured testing solution.
  4. Comes with a pre-configured database solution.
  5. Comes with a pre-configured containerization solution (Dockerfile and secret management).
  6. Comes with a pre-configured Github Actions integration (for CI/CD and automatic deployment to a cloud-hosting service).
  
3. Define the processes and supporting configuration scripts necessary to easily deploy the template into a newly-instantiated local project.


