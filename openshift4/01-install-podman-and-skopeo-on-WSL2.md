## Installing Podman & Skopeo on WSL2

[Podman](https://podman.io/) is RedHat's container management solution alternative to Docker. Two of the major reasons listed for why RedHat decided a new container solution was required are:
1. Docker requires root to run containers (_a big security risk_)
1. Docker has been extended with management layers that are better implemented through a Kubernetes implementation

I'm not going to spend any time on the pros/cons of this solution (see [here](https://linux.slashdot.org/story/20/01/18/0236253/why-did-red-hat-drop-its-support-for-dockers-runtime-engine)  if you want to see what hardcore computer nerds think). 

The training course taught us the `podman` & `skopeo` tools, and these have tight integration with the Openshift container
environment (which my day job requires), so ... I'm using `podman` and `skopeo`.

If you followed my other on-going [Flask Blueprint Project](http://https://gwright99.github.io/flask-blueprint/index.html) series, you'll know that I'm running an Ubuntu 20.04 WSL2 implementation. Given that my personal machine is so much better than my work-provisioned potato, I'll be trying to do the exercises from the personal machine. This means some tool installation will is a necessary first step.

### Install Podman
As of September 26, 2020 I followed the Ubuntu instructions from [https://podman.io/getting-started/installation](https://podman.io/getting-started/installation), with one slight modification.
```bash
source /etc/os-release
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/xUbuntu_${VERSION_ID}/Release.key | sudo apt-key add -
sudo apt-get update
sudo apt-get -y upgrade 
sudo apt-get -y install podman
```

### Install Skopeo
[Skopeo](https://www.redhat.com/en/blog/skopeo-10-released) is a tool for examining local/remote container images, and for moving container images between different container repositories.

During training, two of the use-cases provided for using Skopeo were:
1. You could use it to examine the container in its remote repository (_rather than having to download it to your local repo first, only to discover you justed wasted hundreds of MBs of download cap to pull an image you didn't actually want)
1. You can move an image from one remote repository to another without having to execute intermediate steps of downloading & retagging the image in your local repository

As of September 26, 2020 I followed the Ubuntu instructions from [https://github.com/containers/skopeo/blob/master/install.md](https://github.com/containers/skopeo/blob/master/install.md), with one slight modification (_Note: If you already completed the steps above to install Podman, you will already have added the repo and OpenPGP key. I've retained the steps for completeness_):
```bash
source /etc/os-release
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/x${NAME}_${VERSION_ID}/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/x${NAME}_${VERSION_ID}/Release.key | sudo apt-key add -
sudo apt-get -y update
sudo apt-get -y install skopeo
```

### To alias Podman or not?
Several of the articles I consulted suggested creating an alias between Podman and Docker a la `alias docker=podman`. I decided not to take this step as I have minimal legacy artefacts that would be invoking the `docker` command on the CLI. My preference is to keep both tools independently installed on my system and I can revist the decision to alias later once I have a better idea of what implications this will have (if any).

### Update the Podman configuration for WSL2 quality of life gains
A January 30, 2020 [article](https://www.redhat.com/sysadmin/podman-windows-wsl2) by RedHat software engineer Brent Baude provides some configuration suggestions for WSL2-based podman installations. I began to implement his changes but immediately encountered problems.

Baude suggests creating (and then editing) a Podman configuration file for a rootless user, `$HOME/.config/containers/libpod.conf`, by executing this command:
```bash
podman info
```
I ran the command and it resulted in two problems:
1. I received a `ERRO[0000] unable to write system event: "write unixgram @0000d->/run/systemd/journal/socket: sendmsg: no such file or directory"` error
1. The config file was not generated. 

A subsequent Google searchs revealed answers to both problems:
1. As per [this GitHub thread](https://github.com/containers/podman/issues/4325), the error occured due to the lack of a systemd journal in WSL2. 
1. As of at least April 2020, podman [no longer generates config files](https://github.com/containers/podman/issues/5722)  in the user's home directory automatically.

#### Solving the systemd problem
I solved the systemd error by following this Sept 10, 2020 [article](https://dev.to/bowmanjd/using-podman-on-windows-subsystem-for-linux-wsl-58ji) by Jonathan Bowman. 

Due to the lack of systemd, we need to provide a folder for podman to use for temporary files. Following Bowman's advice, I added the following to my `~/.bashrc`:
```bash
if [[ -z "$XDG_RUNTIME_DIR" ]]; then
  export XDG_RUNTIME_DIR=/run/user/$UID
  if [[ ! -d "$XDG_RUNTIME_DIR" ]]; then
    export XDG_RUNTIME_DIR=/tmp/$USER-runtime
    if [[ ! -d "$XDG_RUNTIME_DIR" ]]; then
      mkdir -m 0700 "$XDG_RUNTIME_DIR"
    fi
  fi
fi
```
As per Bowman, _"This script checks if the $XDG_RUNTIME_DIR is set, and, if not, sets it to the default systemd location (/run/user/$UID). If that does not exist, then set and create a temporary directory for the current user."_ 

With the change added, I reloaded the environment variables via `source ~/.bashrc` and retried the `podman info` command. Success - no more error!

#### Solving the missing config file problem
Despite fixing the systemd error, the `$HOME/.config/containers/libpod.conf` podman configuration file that Bauman says I should be editing was still missing. Jonathan Bowman's artice comes to the rescue again. 

I didn't have a `$HOME/.config/` folder, but I did have two other podman-related folders:
* `/etc/containers/`
* `/usr/share/containers/`

Things got a bit confusing once I looked at the contents of each folder: there was no sign of a `libpod.conf` file, but both folders had an instance of `containers.conf`.

I ran the following command to ensure these two files contained the same content (they did):
```bash
diff /etc/containers/containers.conf /usr/share/containers/containers.conf
```

At this point, I had to decide who I trusted more: Brent Baude or Jonathan Bowman? 
Even though Baude's article is linked to in the official steps of [podman.io](https://podman.io), Bowman's was newer and had already fixed my systemd problem. I decided to trust Bowman.

Following Bowman's instructions, I created the config file for a rootless user:
```bash
mkdir ~/.config/containers
sudo cp /etc/containers/containers.conf ~/.config/containers/containers.conf`
```

I then modified `~/.config/containers/containers.conf`, making the following two changes:
* `cgroup_manager = "cgroupfs"`
* `events_logger = "file"`

Once these changes were implemented I was able to successful pull and run an Alpine Linux image via:
```bash
podman run -it docker.io/library/alpine:latest
```
