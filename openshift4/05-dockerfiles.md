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

### Dockerfile
Here's an example Dockerfile and a few things to note:
```docker
# This is a comment line
FROM docker.io/library/ubuntu:latest
LABEL description="This is a custom httpd container image"
MAINTAINER John Doe <jdoe@xyz.com>
RUN RUN apt-get update && apt-get install -y --no-install-recommends --yes httpd
EXPOSE 80
ENV LogLevel "info"
ADD http://someserver.com/filename.pdf /var/www/html
COPY ./src/ /var/www/html/
USER apache
ENTRYPOINT ["/usr/sbin/httpd"]
CMD ["-D", "FOREGROUND"] 
```

Notes:
* The RUN instruction executes commands in a new layer on top of the current image. We should endeavour to run multiple steps in the same run command to minimize the number of layers in our image.
* The EXPOSE instruction defines metadata only. The port only becomes from the host if the container is invoked with the `podman run -p` command.
* The ADD instruction can:
    * Copy local OR remote sources to the container's file system. 
    * If used to copy local files, they must be in the current working directory.
    * ADD will also unpack local `.tar` files to the destination image folder.
 * The COPY command copies local files only to the container file system. It cannot copy remote sources, not unpack archive files.
 * ADD & COPY both use root as the owner. It is recommended to change the owner after transferring the files.
 * The USER command specifies the UID to use when executing RUN, CMD, and ENTRYPOINT instructions.
 * ENTRYPOINT & CMD are used to invoke a process in the container:
     * There should be, at most, one ENTRYPOINT and one CMD instruction (if there are more, only the last one takes effect).
     * ENTRYPOINT specifies the default command to execute in the container (it cannot be overrode during the `podman run` invocation, unlike CMD).
     * ENTRYPOINT defaults to `/bin/sh -c` if nothing is specified in the Dockerfile.
     * CMD provides default arguments for the ENTRYPOINT
     * CMD can be present without specifying an ENTRYPOINT.
     * Podman can override CMD during the `podman run` command.
     
     
I want to focus on ENTRYPOINT and CMD a bit more. Consider the following examples:
```bash
EXAMPLE 1
# ENTRYPOINT ["/bin/date", "+%H:%M"]

EXAMPLE 2
# ENTRYPOINT ["/bin/date"]
# CMD ["+%H:%M"]

EXAMPLE 3
# CMD ["date", "+%H:%M"]
```

Example 1 defines the command to be executed and its parameters. These cannot be changed at container invocation time.
Example 2 defines the command in ENTRYPOINT, but specifies its parameters in CMD. The parameters can be changed at container invocation time, but the container will always execute `/bin/date`.
Example 3 leverages the default ENTRYPOINT `/bin/sh -c` and passes in a shell command and related parameters. These can all be changed at container invocation time.

 #### Exec versus shell command form
 The ADD, COPY, CMD, and ENTRYPOINT commands have two different notation styles: _Exec_ and _Shell_. It is generally recommend to use Exec because it avoids wrapping the command in a shell (which can cause strange results).

Examples:
    * Exec form: `ENTRYPOINT ["command", "param1", "param2"]`
    * Shell form: `ENTRYPOINT command param1 param2`

    * Exec form: `COPY ["<source>", ... "<destination"]`
    * Shell form: `COPY <source> ... <destination>`
 
     
     
 EXEC vs SHELL form
 exec form preferred
