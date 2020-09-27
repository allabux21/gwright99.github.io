## The Nightmare Non-Root Configuration Effort

This was *really* painful, particularly since I don't have deep mastery of core Linux systems like `systemd` and `cgroups`. If you don't want to read about my ordeal, just read the



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

basically '\[\[ -z "$XDG_RUNTIME_DIR" \]\]' checks to see if the XDG_RUNTIME_DIR has a length equal to zero. If yes, we set it. 

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

The first thing I found was that I had missed a runtime entry. I found the line `# runtime = "runc"` and added a new line below `runtime = "crun"`. This change was confirmed when I checked the podman system info again.

So how the hell do I change my cgroup version? This was starting to turn into a goosechase:
1. This [stackoverflow post](https://stackoverflow.com/questions/61140609/how-do-i-change-the-cgroup-version-for-podman) asked the exact same question. The single answer mentioned 'systemd' (which clearly wasn't going to help me since WSL2 doesnt have it, or OpenRC (I had no idea what this was). A response to the answer mentioned that for Ubuntu20.04, the poster ended up editing /etc/default/grub.d/50-cloudimg-settings.cfg, followed by update-grub, followed by a reboot, to get podman to show cgroups v2. But the poster didnt say what they had actually added.

1. This [GitHub Gist](https://gist.github.com/trevorwhitney/d83353ff59cc0b7d8ae58116d1fe98f0) said that - to enable cgroups v2 on a Google Cloud instance, you need to edit /etc/default/grub.d/50-cloudimg-settings.cfg and add `cgroup_no_v1=all` to the end of the GRUB_CMDLINE_LINUX_DEFAULT, so it looks something like `GRUB_CMDLINE_LINUX_DEFAULT="console=tty50 cgroup_no_v1=all".

I had a decision to make did I even need cgroups v2? I only made the changes earlier because of the installation steps saying you had to make changes to accommodate cgroups v2. But if podman thought it was using cgroups v1, was I ok? I had no idea.

I started thinking about the actual error I received again: `Error: systemd cgroup flag passed, but systemd support for managing cgroups is not available: OCI runtime error`. Even though it didn't show up in the `podman system info` output, I had added an uncommented `cgroup_manager="cgroupfs"` into the file (meant to supplant the default `cgroup_manager="systemd"`. So why was podman complaining about the systemd cgroup flag?!

I took another look at the ~/.config/containers/containers.conf file, via `less ~/.config/containers/containers.conf | grep cgroup`. This resulted in the following hits:
```bash
# Default way to to create a cgroup namespace for the container
# cgroupns = "private"
# Control container cgroup configuration
# `enabled`   Enable cgroup support within container
# `disabled`  Disable cgroup support, will inherit cgroups from parent
# cgroups = "enabled"
# Valid options “systemd” or “cgroupfs”
# cgroup_manager = "systemd"
cgroup_manager = "cgroupfs"
# List of the OCI runtimes that supports running containers without cgroups.
# runtime_supports_nocgroups = ["crun"]
```

sudo podman --cgroup-manager=cgroupfs run -it --runtime=/usr/local/sbin/crun docker.io/library/alpine
Trying to pull docker.io/library/alpine...
Getting image source signatures
Copying blob df20fa9351a1 done
Copying config a24bb40132 done
Writing manifest to image destination
Storing signatures
ERRO[0015] unable to write pod event: "write unixgram @0003d->/run/systemd/journal/socket: sendmsg: no such file or directory"
Error: mount `/sys/fs/cgroup/systemd` to '/sys/fs/cgroup/systemd': No such file or directory: OCI runtime command not found error





If I'm on cgroupsv1 did I need to make the v2 changes I did earlier? Dont think so, went back to runc instead of crun.

confirmed I am not root by using `cat /etc/password`. User account has UID 1000, where there is also a root account with UID 0.cat
Changed /usr/share/containers/backup-containers.conf back to containers.conf
Deleted /etc/containers/containers.conf (since I'm running as non-privileged and this is meant for root)
sudo apt-get install slirp4netns was already installed

sudo apt-cache search fuse-overlayfs
sudo apt-cache show fuse-overlayfs (Version: 1.1.2~1)

Check ~/.config/containers/storage.conf (missing), copy it in from /etc/containers/storage.conf.
set 'driver = "overlay"'
set 'mount_program = "/usr/bin/fuse-overlayfs'


Issue with ~/.config/container/* files belonging to root instead of deep learning? chownd -R to deeplearning


Didn't fix the unprivileged ping issue (https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md)

Copied /etc/containers/registries.conf back to ~/.config/containers/registries.conf




podman run -v ~/OCP4/podmanvolumes:/container/volume docker.io/alpine /bin/bash
Error: error creating runtime static files directory /var/lib/containers/storage/libpod: mkdir /var/lib/containers/storage/libpod: permission denied

storage.conf's 'graphroot' key has the value of /var/lib/containers/storage/libpod.

in ~/.config/containers/storage.conf, made the following changes:
graphroot="$HOME/.local/share/containers/storage"
runroot="$XDG_RUNTIME_DIR/containers"


ERRO[0000] User-selected graph driver "overlay" overwritten by graph driver "vfs" from database - delete libpod local files to resolve
had to delete ~./local/share/containers/

~/.local/share/containers/storage/ had lots of files like libpod vfs etc.


Screwed up, deleted system folders /usr/share/containers/ and /etc/containers/ . Repopulated by recreating by hand and copying in file content from Github.
First copied containers.conf to /usr/ ...
Second copies registries.conf and storage.conf to /etc/containers
Copied all three files to ~/.config/containers/
chown -R deeplearning:deeplearning -R ~/.config/containers/

realized was missing stuff
did forced reinstall of packages (sudo apt-get --reinstall install <pkgname>). Used conmon, containers-common containers-golang containers-image libgpgme11 (as per https://computingforgeeks.com/install-cri-o-container-runtime-on-ubuntu-linux/), poddman skopeo

Had to grab policy.json too
https://github.com/containers/image/blob/3149c1a414eb131a61397efdebf765853b76972b/signature/fixtures/policy.json
https://github.com/cri-o/cri-o/issues/901

Tried running `podman run -it docker.io/alpine:latest`. Got error: Error: error creating runtime static files directory /var/lib/containers/storage/libpod: mkdir /var/lib/containers/storage/libpod: permission denied

Edited ~/.config/containers/storage.conf
Using paths given by https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md
graphroot="$HOME/.local/share/containers/storage"
runroot="$XDG_RUNTIME_DIR/containers"

runroot will end up being /tmp/deeplearning-runtime/containers (based on .bashrc entry)

While updating also made driver = "overlay" and mount_program = "/usr/bin/fuse-overlayfs"


Ended up installing an Ubuntu18.04 instance, reinstalling Podman and then copying /usr/share/containers/*.* and /etc/containers/*.* from the 18.04 instance to the 20.04 instance through windows. Seems to have got me back to stable.

didn't change runc.
changed storage to use overlay, new graphroot and runroot, and mount_program.
changed ~/containers to set 'events_logger = "file"'
