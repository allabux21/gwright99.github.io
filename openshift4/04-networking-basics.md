## Podman Networking Basics

Podman uses the Container Networking Interface (CNI) [open source project](https://www.cncf.io/blog/2017/05/23/cncf-hosts-container-networking-interface-cni/). The CNI aims to standardize the network interface for containers in cloud environments.

Podman uses the CNI project to implement a software-defined network (SDN) to link containers on a host. Each container is assigned a private IP address and attached a virtual bridge (not sure what this means). The CNI settings file can be found at `/etc/cni/net.d/87-podman-bridge.conflist`.

<img src="./img/PodmanNetworking.png">
