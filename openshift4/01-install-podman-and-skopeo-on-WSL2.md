## Installing Podman & Skopeo on WSL2

[Podman](https://podman.io/) is RedHat's container management solution alternative to Docker. Two of the major reasons listed for why RedHat decided a new container solution was required are:
1. Docker requires root to run containers (_a big security risk_)
1. Docker has been extended with management layers that are better implemented through a Kubernetes implementation

I'm not going to spend any time on the pros/cons of this solution (_see [here](https://linux.slashdot.org/story/20/01/18/0236253/why-did-red-hat-drop-its-support-for-dockers-runtime-engine)  if you want to see what hardcore computer nerds think_). The training course taught us the `podman` & `skopeo` tools, and these have tight integration with the Openshift container environment (which my day job requires), so ... I'm using `podman` and `skopeo`.

If you followed my other on-going [Flask Blueprint Project](http://https://gwright99.github.io/flask-blueprint/index.html) series, you'll know that I'm running an Ubuntu 20.04 WSL2 implementation. I'll be using this same implementation to install the tooling.

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

#### Problem 1: Solving the systemd problem
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

#### Problem 2: Solving the missing config file problem
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

I could also successfully search the `docker.io` and `quay.io` registries for a desired image. `sudo podman search rhel` returned:
```bash
INDEX       NAME                                             DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
docker.io   docker.io/richxsl/rhel7                          RHEL 7 image with minimal installation            28
docker.io   docker.io/xplenty/rhel7-pod-infrastructure       registry.access.redhat.com/rhel7/pod-infrast...   1
docker.io   docker.io/bluedata/rhel7                         RHEL-7.x base container images                    1
docker.io   docker.io/sotax/rhel7.3                          base rhel_7.3 image with g++                      0
docker.io   docker.io/kdiraviam/rhel6-ruby-chef              This repo contains a rhel6 image with ruby a...   1

....

quay.io     quay.io/thoth-station/solver-rhel-8-py36         solver-rhel-8-py36                                0
quay.io     quay.io/distributedci/dci-rhel-agent             # DCI RHEL Agent `dci-rhel-agent` provides R...   0
quay.io     quay.io/zorondevops/java-rhel-7-base                                                               0
quay.io     quay.io/bluecat/gateway                          Official release repository for BlueCat Gate...   0
quay.io     quay.io/sdase/nginx                              # nginx OCI image  [nginx](https://nginx.org...   0
quay.io     quay.io/gravisbulgaria/solr                      Solr:8.1.1 based ot RHEL 8                        0
quay.io     quay.io/crw/plugin-java8-rhel8                   OpenJDK 8 + Maven 3.6, Node + npm, python3 +...   0
```

#### Problem 3: Solving the failed docker pull
The run command was working but I got the following error when I tried `sudo podman pull rhel`:
```bash
Trying to pull docker.io/library/rhel...
  denied: requested access to the resource is denied
Trying to pull quay.io/rhel...
  StatusCode: 404, <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final/...
Error: unable to pull rhel: 2 errors occurred:
        * Error initializing source docker://rhel:latest: Error reading manifest latest in docker.io/library/rhel: errors:
denied: requested access to the resource is denied
unauthorized: authentication required

        * Error initializing source docker://quay.io/rhel:latest: Error reading manifest latest in quay.io/rhel: StatusCode: 404, <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final/...
```

Now I needed to figure out what the problem was? Local system permissions? Registry problem? Time to start experimenting.

The first thing I did was try to pull from a different registry. As per the [Getting Started instructions at podman.io](http://podman.io/getting-started/), I tried ` podman pull registry.fedoraproject.org/f29/httpd`. This worked!
```bash
Trying to pull registry.fedoraproject.org/f29/httpd...
Getting image source signatures
Copying blob d77ff9f653ce done
Copying blob 7692efc5f81c done
Copying blob aaf5ad2e1aa3 done
Copying config 25c76f9dcd done
Writing manifest to image destination
Storing signatures
...
```

I confirmed the image was in my local repository via ```bash podman images```:
```bash
REPOSITORY                            TAG     IMAGE ID      CREATED        SIZE
docker.io/library/alpine              latest  a24bb4013296  4 months ago   5.85 MB
registry.fedoraproject.org/f29/httpd  latest  25c76f9dcdb5  17 months ago  482 MB
```

You'll notice that I didn't use `sudo` in the above commands, so I started to wonder if this was the problem. I removed the image via `podman rmi registry.fedoraproject.org/f29/httpd` and then tried pulling the image again with `sudo podman pull registry.fedoraproject.org/f29/httpd`.

I was still able to pull the image down, but got two errors when I tried `sudo podman run -it registry.fedoraproject.org/f29/httpd`:
```bash
ERRO[0000] unable to write pod event: "write unixgram @00014->/run/systemd/journal/socket: sendmsg: no such file or directory"
Error: systemd cgroup flag passed, but systemd support for managing cgroups is not available: OCI runtime error
```

It looks like the systemd problems had returned. More on that later. First I needed to see if I could run the image without sudo, so I tried 
```bash 
podman run -it registry.fedoraproject.org/f29/httpd
``` 

This got a different result:
```bash
Trying to pull registry.fedoraproject.org/f29/httpd...
Getting image source signatures
Copying blob 7692efc5f81c done
Copying blob d77ff9f653ce done
Copying blob aaf5ad2e1aa3 done
Copying config 25c76f9dcd done
Writing manifest to image destination
Storing signatures
=> sourcing 10-set-mpm.sh ...
=> sourcing 20-copy-config.sh ...
=> sourcing 40-ssl-certs.sh ...
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.2.100. Set the 'ServerName' directive globally to suppress this message
[Sun Sep 27 14:03:18.700163 2020] [ssl:warn] [pid 1:tid 140363836153216] AH01882: Init: this version of mod_ssl was compiled against a newer library (OpenSSL 1.1.1b FIPS  26 Feb 2019, version currently loaded is OpenSSL 1.1.1 FIPS  11 Sep 2018) - may result in undefined or erroneous behavior
[Sun Sep 27 14:03:18.701415 2020] [ssl:warn] [pid 1:tid 140363836153216] AH01909: 10.0.2.100:8443:0 server certificate does NOT include an ID which matches the server name
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.0.2.100. Set the 'ServerName' directive globally to suppress this message
[Sun Sep 27 14:03:18.767182 2020] [ssl:warn] [pid 1:tid 140363836153216] AH01882: Init: this version of mod_ssl was compiled against a newer library (OpenSSL 1.1.1b FIPS  26 Feb 2019, version currently loaded is OpenSSL 1.1.1 FIPS  11 Sep 2018) - may result in undefined or erroneous behavior
[Sun Sep 27 14:03:18.768234 2020] [ssl:warn] [pid 1:tid 140363836153216] AH01909: 10.0.2.100:8443:0 server certificate does NOT include an ID which matches the server name
[Sun Sep 27 14:03:18.768592 2020] [lbmethod_heartbeat:notice] [pid 1:tid 140363836153216] AH02282: No slotmem from mod_heartmonitor
[Sun Sep 27 14:03:18.772186 2020] [mpm_event:notice] [pid 1:tid 140363836153216] AH00489: Apache/2.4.39 (Fedora) OpenSSL/1.1.1 configured -- resuming normal operations
[Sun Sep 27 14:03:18.772257 2020] [core:notice] [pid 1:tid 140363836153216] AH00094: Command line: 'httpd -D FOREGROUND'

(GW: CLI hung at this point and I had to Ctrl-c to break out)
```

Let's [figure out if cgroup v2 is running in my WSL2 instance](https://unix.stackexchange.com/questions/471476/how-do-i-check-cgroup-v2-is-installed-on-my-machine) or not.
```bash
grep cgroup /proc/filesystem
```
This returned:
```bash
nodev   cgroup
nodev   cgroup2
```
So this means that `cgroup v2` is available, but I don't know if this is the version that I'm actually using. This becomes more important because the [Podman documentation for Basic Setup and Use of Podman in a rootless environment](https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md) says that you may need to change your default OCI runtime from `runc` to `crun` if your system is enabled cgroup v2.

I tried to change the runtime as per the provided command but couldn't get it to work:
* When I tried `sudo podman --runtime /usr/bin/crun`, I got an `Error: missing command 'podman COMMAND'` error.
* When I tried `podman --runtime /usr/bin/crun`, I got an `Error: missing command 'podman COMMAND'` error.
* When I tried `sudo podman run -it registry.fedoraproject.org/f29/httpd --runtime /usr/bin/crun`, I got the `Error: systemd cgroup flag passed, but systemd support for managing cgroups is not available: OCI runtime error` again.

Thankfully the documentation offers another way to change the runtime via the Podman configuration files. This then lead to me to this [page](https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md#user-configuration-files) and proves the value of READING THE DOCUMENTATION! Turns out Podman is configured to check file system folders in a particular order:
1. /usr/share/containers/containers.conf
1. /etc/containers/containers.conf
1. ~/.config/containers/containers.conf

This gets a bit messier, as it appears `/usr/share/containers/containers.conf` is the base file for BOTH the root user (with overrides coming from `/etc/containers/containers.conf`) and rootless user (with overrides coming from `$HOME/.config/containers/containers.conf`).

First, I need to reconfirm that my WSL2 account is a user and NOT root. Running `id` returns a `uid=1000` and `gid=1000`. Since these are not 0, this means I'm not running as root.

Since I'm trying to get rootless working, I need to go edit `$HOME/.config/containers/containers.conf`. I then navigated to the 'engine.runtimes' portion of the file, commenting out the `runc` array and uncommenting the `crun` array.
```bash
# Paths to look for a valid OCI runtime (runc, runv, kata, etc)
[engine.runtimes]
#runc = [
#        "/usr/lib/cri-o-runc/sbin/runc",
#        "/usr/sbin/runc",
#        "/usr/bin/runc",
#        "/usr/local/bin/runc",
#        "/usr/local/sbin/runc",
#        "/sbin/runc",
#        "/bin/runc",
#]

 crun = [
            "/usr/bin/crun",
            "/usr/sbin/crun",
            "/usr/local/bin/crun",
            "/usr/local/sbin/crun",
            "/sbin/crun",
            "/bin/crun",
            "/run/current-system/sw/bin/crun",
 ]
```

This still didn't work. I needed to confirm that my changes had actually taken effect so I ran `podman system info` and got:
```bash
host:
  arch: amd64
  buildahVersion: 1.16.1
  cgroupVersion: v1
  conmon:
    package: 'conmon: /usr/libexec/podman/conmon'
    path: /usr/libexec/podman/conmon
    version: 'conmon version 2.0.20, commit: '
  cpus: 4
  distribution:
    distribution: ubuntu
    version: "20.04"
  eventLogger: file
  hostname: <redacted>
  idMappings:
    gidmap:
    - container_id: 0
      host_id: 1000
      size: 1
    - container_id: 1
      host_id: 100000
      size: 65536
    uidmap:
    - container_id: 0
      host_id: 1000
      size: 1
    - container_id: 1
      host_id: 100000
      size: 65536
  kernel: 4.19.104-microsoft-standard
  linkmode: dynamic
  memFree: 15036280832
  memTotal: 20042506240
  ociRuntime:
    name: runc
    package: 'runc: /usr/sbin/runc'
    path: /usr/sbin/runc
    version: 'runc version spec: 1.0.1-dev'
  os: linux
  remoteSocket:
    path: /tmp/<redacted>-runtime/podman/podman.sock
  rootless: true
  slirp4netns:
    executable: /usr/bin/slirp4netns
    package: 'slirp4netns: /usr/bin/slirp4netns'
    version: |-
      slirp4netns version 1.1.4
      commit: unknown
      libslirp: 4.3.1-git
      SLIRP_CONFIG_VERSION_MAX: 3
  swapFree: 5368709120
  swapTotal: 5368709120
  uptime: 2h 22m 38.06s (Approximately 0.08 days)
registries:
  search:
  - docker.io
  - quay.io
store:
  configFile: /home/<redacted>/.config/containers/storage.conf
  containerStore:
    number: 6
    paused: 0
    running: 0
    stopped: 6
  graphDriverName: vfs
  graphOptions: {}
  graphRoot: /home/<redacted>/.local/share/containers/storage
  graphStatus: {}
  imageStore:
    number: 2
  runRoot: /tmp/run-1000/containers
  volumePath: /home/<redacted>/.local/share/containers/storage/volumes
version:
  APIVersion: 2.0.0
  Built: 0
  BuiltTime: Wed Dec 31 19:00:00 1969
  GitCommit: ""
  GoVersion: go1.15.2
  OsArch: linux/amd64
  Version: 2.1.0
```

I could see 2 potential problems:
1. It appeared that my cgroup was set to version 1
1. It appeared that the the runc to crun change had not taken effect. 

I did notice that `eventLogger: file` was present though, so it made me think some of my earlier changes had stuck. To test this, I edited the `~/.config/containers/containers.conf` and commented out the `events_logger = "file"` line. My expectation was that, when I ran `podman system info` again, I should now see `eventLogger: journald`. I ran the podman system info command again and found the 'journald' entry. This proved that changes I made in the user-level config file WAS being reflected in the podman system info.

