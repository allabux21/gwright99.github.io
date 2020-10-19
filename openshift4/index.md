## Openshift 4 Training
I was recently sent on two Openshift 4 (OCP4) training courses by my employer:
* [DO180 - Introduction to Containers, Kubernetes, and Red Hat Openshift](https://www.redhat.com/en/services/training/do180-introduction-containers-kubernetes-red-hat-openshift)
* [DO288 - Red Hat Openshift Development 1: Containerizing Applications](https://www.redhat.com/en/services/training/do288-red-hat-openshift-development-i-containerizing-applications)

To entrench and solidify my understanding of the material that was covered during the courses, I'll be going over the exercises again and documenting steps, gotchas, and notes on why things work a certain way.

I won't be covering the underlying theories or explanation of why one should use containers, but I will make notes on aspects of container implementations because these can require specific actions in some container deployment exercises.

### Articles
* [01 - Installing Podman & Skopeo on WSL2](./01-install-podman-and-skopeo-on-WSL2.md)
* [01A - The Configuration Nightmare](./01a-configuration-nightmare.md)
* [02 - Background Context](./02-background-context.md)
* [03 - Command Basics](./03-command-basics.md)
* [04 - Networking Basics](./04-networking-basics.md)
* [05 - Dockerfiles](./05-dockerfiles.md)
* [06 - Openshift Resources](./06-openshift-resources.md)
* [07 - oc Create](./07-oc-create.md)
* [08 - Source-to-Image (S2I)](./08-source-to-image.md)
* [09 - Building An Openshift App](./09-builiding-openshift-app.md)
* [10 - Multi-Container Applications](./10-multi-container-applications.md)

### Decision to Abandon Podman (mid-Oct 2020)
I've decided to abandon the use of the Podman tool. 

The decision was driven by problems I encountered while trying to create an Openshift POC by following an article written by Scott Zelenka, [How to do rapid prototyping with Flask, uWSGI, NGINX, and Docker on OpenShift](https://towardsdatascience.com/how-to-do-rapid-prototyping-with-flask-uwsgi-nginx-and-docker-on-openshift-f0ef144033cb). 

In short, Zelenka bases his Dockerfile on the official NGINX image, adds in `supervisord` magic that allows multiple processes to be run within the container (_i.e. the supervisord process - the primary process in the container - will in turn spin up an NGINX process and uWSGI process_), and then modifies system permissions so that the resulting image will comply with Openshift's security posture that requires all containers to be launched as an arbitrary user and forbids the use of privileged ports.

Ignoring the whole "in Production, your web server should probably be separate from your application server" issue, this seemed like a perfect way to quickly get a POC stood up:
1. Write some code on my local machine.
2. Create the image and push to a Docker registry.
3. Pull the image down onto my work computer.
4. Deploy an application based on this image to the work OpenShift cluster.

Easy peasy, right? No.

At the beginning of this series, I specifically decided to configure my local Podman installation to run as rootless. This was done for good reasons at the time: the need for Docker to run as root has been repeatedly called out as a security risk, and the rootless options seemed like a great way to avoid this problem. What I failed to do, however, is to adhere to the guiding principle that I should __at all costs__ minimize the complexity of my tooling and the cognitive load needed to implement working solutions. 

Even with this basic example - excluding any of the Python, Flask, or web programming knowledge I would need to do anything more complicated than a basic GET "Hello World" service - I still needed to devote brainpower to all the following:

* Understand the Nginx configuration 
* Understand the uWSGI configuration
* Understand the the relationship between NGINX, uWSGI, and Flask (something I already understood at a conceptual level already, but now I had to figure out how the actual configuration values interacted)
* Figure out what on earth supervisord is, how to use it, and what (if any) pitfalls it introduced.
* Refresh my Bash scripting knowledge so I could grok the shell script used to create an arbitrary user in the container
* Understand the sequence of `chmod` and `chgrp` commands, and how they differed from a non-OCP container
* Figure out which kludges in the Dockerfile were no longer necessary due to subsequent development/fixes since September 2018.
* Figure out how to get the resulting image off my own computer and onto my work computer when:
    1. I wanted to use the Github Container Repository instead of Docker Hub to host my image;
    1. My work machine did not have Docker installed;
    1. My RedHat training materials did not cover this particular use case.
* Figure out and properly configure all the OCP/Kubernetes components I would need so that the OCP container was actually accessible and useable.

Podman (rootless) immediately began causing me grief as soon as I had built the local files and started trying to instantiate a local application from the generated Docker image. The most notable problem was the error `ERRO[0000] error joining network namespace for container` that would cause the container to immediately crash. Like most things Podman-related, my Google search returned a very limited number of results, and fewer of those provided helpful guidance. Most looked something like [this](https://github.com/containers/podman/issues/6800) - a confusing exchange, followed by references to systemd and SELinux (instilling me with dread), followed by a (to me at least) very unclear resolution before the issue was closed. Based on other articles I found, the problem seemed to be centered around the fact that the NGINX container expects to run as root .... maybe?

Could I have figured this out if I put more effort into troubleshooting? Maybe.
Did I think this would be complicated given my Podman was rootless and running on a Windows WSL2 instance? Yes.
Did I believe I had enough hardcore Linux knowledge to figure out how to implement a solution and - if I totally screwed things up - how to back it out without messing up the rest of my WSL2 environment (with other project files that I could not afford to lose)? No way.

At this point I stopped what I was doing and asked myself an important question: _"Why are you needlessly overcomplicating your life?!"_.

Instead of using the industry-standard Docker (which has an immense body of documentation and support topics), I was using a niche product from RedHat where even the official documentation points to out-of-date material ([see my Configuration Nightmare post](./01a-configuration-nightmare.md)) and Google struggles to return useful results.

Instead of using a well-understood and well-documented pattern of "container runs as root" (despite its security risk), I was struggling to figure out workarounds and modifications to the "Hello World"-equivalent tutorial I had embarked upon to gain a BASIC understanding of the OpenShift environment. What's worse, given the limitations of the OpenShift platform, it seemed like running rootless was pointless because we had to modify permissions & ownership anyways!


<KEEP GOING HERE>

In my mind, it just didn't seem to make sense. 

So ... I ditched Podman and switched to Docker instead. A few error-free commands later, the resultant image was created and successfully pushed to GitHub.


