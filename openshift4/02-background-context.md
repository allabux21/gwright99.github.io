## Background Context

Although I won't be explaining basic container details/concepts, there are a few facets of Podman/Openshift implementation that need to be further described due to their impact on implementation efforts.

### Linux Kernel
The Linux kernel uses several technologies to isolate a process inside a server including:
|--|--|
|Namespaces|ipsem| 
* Control groups (cgroups)
* Seccomp
* SELinux (Security-Enhanced Linux)

### Podman
Podman is primarily used to interact with container images and the resulting container processes. Some salient key points:
* It implements the [Openshift Container Initiative](https://www.opencontainers.org) image specification.
* It stores images in a local file-system.
* It shadows the Docker CLI command syntax
* It is Kubernetes-compatible
