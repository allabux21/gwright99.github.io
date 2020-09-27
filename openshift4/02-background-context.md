## Background Context

Although I won't be explaining basic container details/concepts, there are a few facets of Podman/Openshift implementation that should be called out due to their impact on implementation efforts.

### Linux Kernel Security Features
The Linux kernel uses several technologies to isolate a process inside a server, while still allowing that process access to necessary system resources. These technologies include:
| Technology | Description | Implication |
| :--------- | :---------- | :---------- |
| Namespaces | Used to isolate system resources so that only processes that are members of the namespace can see them. This includes resources like network interfaces, process ID list, mount points, and the system's host name information. | tbd |
| Control groups (cgroups) | Defines groupings of processes and their children, and throttles the amount of system resources each group can use. | tbd|
| Seccomp | Limits how processes can use system calls, via security profiles which whitelist allowed system calls, parameters, and file descriptors. | tbd |
| SELinux (Security-Enhanced Linux) | Access control system for processes, used by the kernel to protect the host and its processes from interference by other processes. | tbd |

From a Linux kernel perspective, a container is simply a process with some restrictions. A container process is not a binary file, however, but rather a file-system bundle that contains all the dependencies needed to execute the process (including files in the file system, necessary packages, resources, and kernel modules).

### Podman Features
Podman is primarily used to interact with container images and the resulting container processes. Some salient key points:
* It implements the [Openshift Container Initiative](https://www.opencontainers.org) image specification.
* It stores images in a local file-system.
* It shadows the Docker CLI command syntax
* It is Kubernetes-compatible

### Kubernetes Features
| Feature | Description | Implication |
| :------ | :---------- | :---------- |
| Service Discovery & Load Balancing| Kubernetes assigns a single DNS entry to the set of containers that comprise a service. This provide a single consistent URL for other consuming services to use, while allowing the cluster to change the location and IP of those containers as needed. | tbd |
| Horizontal scaling | Pods can scale up and down in response to traffic spikes and troughs. |  tbd |
| Self-Healing | Kubernetes uses liveness and readiness probes to moniter the health of its containers, restarting failing containers as needed. | tbd |
| Automated rollout | Kubernetes can roll out new versions of containers in response to defined system events. | tbd |
| Secrets & Configuration Management | Sensitive data can made available to containers without requiring commitment into source control | tbd |
| Operators | Kubernetes applications that use the Kubernetes API to update the cluster's state in response to application state. | tbd | 


# Openshift Features
The Openshift Container Platform extends a base Kubernetes implementation with additional features such as:
ADD LATER
