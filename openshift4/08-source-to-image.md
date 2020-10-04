## Source-to-Image (S2I)
Source-to-Image is an OpenShift tool used to create containers based on application source code. The tool collects the source code from a Git repository, injects the source code into a base container that is configured for its programming language, and produces a new container image that runs the assembled application.

<img src="./img/source-to-image.png"><br>

S2I is the suggested primary strategy for building applicationns in OpenShift because:
* It removes the need for developers to understand Dockerfiles and many Linux system commands.
* Allows for rebuilding applications consistently if a base image needs a patch due to a security issue.
* Assembly process can perform a large number of complex operations without requiring a layer for each step.
* Encourages reuse of base images and scripts across multiple types of applications.

### What's an Image Stream?

