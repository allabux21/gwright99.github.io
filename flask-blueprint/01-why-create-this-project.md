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

Theoretically, if I could fix the template problem, I could stop worrying about the infrastructure and focus exclusively on my code.

#### Reuse, Don't Reinvent!
Martin Heinz, an author regularly featured in the Hacker News newsletter, wrote an excellent set of articles on the [Ultimate Setup for Your Next Python Project](https://martinheinz.dev/blog/14) and [Automating Every Aspect of Your Python Project](https://martinheinz.dev/blog/17). I highly recommend that readers of this blog series check them out, because I've heavily reused his material within my own efforts (_and made some design decision based on his explanations_).

Despite the excellence of the series, there was a problem that caused it to be insufficient for fulfilling my needs: the resulting Docker container returned a 'Hello World' response to the CLI. 

From a plumbing perspective, a CLI response was perfectly acceptable. One could expect a technically-saavy individual to download the Docker image from the Github Packages Registry, spin up the image via their local Docker installation, and witness the container's CLI response before it terminated. 

I, on the otherhand, did not want to have to rely on an individual to be technically-saavy in order to see the result. Whether fair or not, visuals - particularly those easily-accessible to a layperson via a web browser - earn kudos from the stakeholders far more easily than more advanced (but harder to visualize) plumbing solutions. I still wanted Heinz' project automation, I just wanted the result to be visible from a browser.

Like many efforts in the world of IT, this "simple" change ended up kicking off a plethora of other changes and design considerations. This series is born of that effort. 

<br><br>
NEXT: [Project Goals and Design Considerations](./02-project-goals-and-design-considerations.md)
