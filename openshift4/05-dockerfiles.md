## Dockerfile Fundamentals

Several ways to generate a container:
1. Modify an existing container and save the result
1. Generate an image from a Dockerfile
1. Generae an image from source code (i.e. Source-to-Image aka S2I)

Note to self: (115) Red Hat Software Collections Library (RHSCL):
* Use to provide newer development tools that do not fit the standard RHEL release schedule.
* Do not replace or conflict with default RHEL package - they are installed side-by-side.
* All RHEL subscribers have access to RHSCL
    * Need to enable RHSCL software Yum repositories
    * Naming convention: MySQL5.7 identified as `rh-mysql57`
    * Red Hat provides RHSCL Dockerfiles and related sources in the rhscl-dockerfiles package in the RHSCL repository. 
    * Dockerfiles for CentOS-based images available from https://github.com/sclorg?q=-container.
 
 ### Openshift Source-to-Image (S2I)
 S2I is an alternative to Dockerfiles that uses the following process:
 1. Initiates a _builder image_ (a container that includes a programming language runtime & development tools like compilers and package managers)
 1. The builder image fetches source code from a Version Control system (e.g. Github)
 1. The builder image builds the application binary files inside the container.
 1. The result is saved as a new image that will be used in production deployment.
 
 This process is generally used inside Openshift (why I note it), but also available outside Openshift via the `s2i` utility.
