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

At the beginning of this series, I specifically decided to configure my local Podman installation to run as rootless. This was done for good reasons at the time: the need for Docker to run as root has been repeatedly called out as a security risk, and the rootless options seemed like a great way to avoid this problem. 

What I failed to do, however, is to adhere to the guiding principle that I should __at all costs__ minimize the complexity of my tooling and the cognitive load needed to implement working solutions.


