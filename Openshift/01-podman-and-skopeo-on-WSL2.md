## Installing Podman & Skopeo on WSL2

[Podman](https://podman.io/) is RedHat's container management solution alternative to Docker. Two of the major reasons listed for why RedHat decided a 
new container solution was required are:
1. Docker requires root to run containers (_a big security risk_)
1. Docker has been extended with management layers that are better implemented through a Kubernetes implementation

I'm not going to spend any time on the pros/cons of this solution (see [here](https://linux.slashdot.org/story/20/01/18/0236253/why-did-red-hat-drop-its-support-for-dockers-runtime-engine) 
if you want to see what hardcore computer nerds think). 

The training course taught us the `podman` & `skopeo` tools, and these have tight integration with the Openshift container
environment (which my day job requires), so ... I'm using `podman` and `skopeo`.

If you followed my other on-going [Flask Blueprint Project](http://https://gwright99.github.io/flask-blueprint/) series, you'll know that I'm running an Ubuntu 20.04 WSL2 implementation.
Given that my personal machine is so much better than my work-provisioned potato, I'll be trying to do the exercises from the personal machine. This means some tool installation will 
is a necessary first step.

###Install Podman
I followed the instructions from [https://podman.io/getting-started/installation](https://podman.io/getting-started/installation). To install `podman` on Ubuntu seemed pretty straightforward:
```bash
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/Release.key | sudo apt-key add -
sudo apt-get update
sudo apt-get -y upgrade 
sudo apt-get -y install podman
```

Note the `xUbuntu_${VERSION_ID}` part of the repository URL. I didn't have this environment variable set, so my `apt-get update` failed until I replaced this with `xUbuntu_20.04`
(you could have set the environment variable instead too). If all goes well, podman will now be available on the CLI.

###Install Skopeo
[Skopeo](https://www.redhat.com/en/blog/skopeo-10-released) is a tool for examining local/remote container images, and for moving container images between different container repositories.
During training, two of the use-cases provided for using Skopeo were:
1. You could use it to examine the container in its remote repository (_rather than having to download it to your local repo first, only to discover you justed wasted hundreds of MBs of download cap to pull an image you didn't actually want)
1. You can move an image from one remote repository to another without having to execute intermediate steps of downloading & retagging the image in your local repository

