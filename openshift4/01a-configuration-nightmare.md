## The Nightmare Non-Root Configuration Effort

This was *really* painful, particularly since I don't have deep mastery of core Linux systems like `systemd` and `cgroups`. If you don't want to read about my ordeal, skip to [what worked](#what-worked). If you want to vicariously share my suffering, let me take you on a journey ...

*NOTE:* Regardless of your reading choice __DO NOT__ run Podman without first reading this section and making sure you've got the right configuration.

### Root versus Non-Root (reminder: just READ this, dont' actually do the steps)
One of Podman's selling points is the ability to run containers without requiring root permissions. This arguably results in a safer security posture, and I felt it was worth using the tool as Non-Root given that I could just use Docker if I wanted a tool with root privileges.

My journey started with a January 30, 2020 [article](https://www.redhat.com/sysadmin/podman-windows-wsl2) by RedHat software engineer Brent Baude provides some configuration suggestions for WSL2-based podman installations. I began to implement his changes but immediately encountered problems.

Baude suggests creating (and then editing) a Podman configuration file for a rootless user, `$HOME/.config/containers/libpod.conf`, by executing this command:
```bash
podman info
```

I ran the command and it resulted in two problems:
1. I received a `ERRO[0000] unable to write system event: "write unixgram @0000d->/run/systemd/journal/socket: sendmsg: no such file or directory"` error
1. The config file was not generated. 

Subsequent Google searches provided insight into both problems:
1. As per [this GitHub thread](https://github.com/containers/podman/issues/4325), the error occured due to the lack of a systemd journal in WSL2. 
1. As of at least April 2020, podman [no longer generates config files](https://github.com/containers/podman/issues/5722)  in the user's home directory automatically.

I found a Sept 10, 2020 [article](https://dev.to/bowmanjd/using-podman-on-windows-subsystem-for-linux-wsl-58ji) by Jonathan Bowman that seemed to have some promising solutions, so I began implementing. (A reminder for the readers now since the authors' names are close enough that I even started to forget who was who when I was writing: Baude == older article that didn't seem accurate anymore, Bowman == newer blog that seemed to have fixes).  

#### Problem 1: Missing runtime storage folder
Ironically, the first problem I fixed was one I didn't even know that I had: a missing runtime folder due to the lack of systemd on WSL2. 
Since systemd won't auotmatically create the folder and set the XDG_RUNTIME_DIR, we need to check and provide a folder for temporary files ourselves. 

Following Bowman's advice, I added the following to my `~/.bashrc`:
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
As per Bowman, _"This script checks if the $XDG_RUNTIME_DIR is set (by checking the length of the value). If it is 0, the script sets it to the default systemd location (/run/user/$UID). If that does not exist, then set and create a temporary directory for the current user."_ 

With the change implemented, reload the your shell via `source ~/.bashrc` and there's one less proble to worry about.


#### Problem 2: Where are the missing configuration files?
The next problem was trying to figure out where the podman configuration files where and which one I should be using:
* The `$HOME/.config/containers/libpod.conf` configuration file that Bauman says I should be editing was nowhere to be found. 
* Bowman's article mentioned I should see `/etc/containers/containers.conf` and/or `~/.config/containers/containers.conf. 
* My system didn't have `$HOME/.config/` but it did have an `/etc/containers/` and `/usr/share/containers`. 
* Even better, both of the folders I found on my system each had a copy of `containers.conf` and NO copies of `libpod.conf`. Greaaaaaaaat ...

The first thing to do was figure out if both instances of `containers.conf` were the same. I did this by diffing the two files (they matched):
```bash
diff /etc/containers/containers.conf /usr/share/containers/containers.conf
```

Now I had to figure out which configuration file was the correct one to use `libpod.conf` or `containers.conf` (i.e. a proxy for "who do I trust more, Baude or Bowman?"). Even though Baude's article is referred to by the official steps of [podman.io](https://podman.io), Bowman's was newer and had already fixed my folder problem. I decided to trust Bowman.

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
....
quay.io     quay.io/thoth-station/solver-rhel-8-py36         solver-rhel-8-py36                                0
quay.io     quay.io/distributedci/dci-rhel-agent             # DCI RHEL Agent `dci-rhel-agent` provides R...   0
quay.io     quay.io/zorondevops/java-rhel-7-base                                                               0
...
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

I was still able to pull the image, but got two errors when I tried `sudo podman run -it registry.fedoraproject.org/f29/httpd`:
```bash
ERRO[0000] unable to write pod event: "write unixgram @00014->/run/systemd/journal/socket: sendmsg: no such file or directory"
Error: systemd cgroup flag passed, but systemd support for managing cgroups is not available: OCI runtime error
```

It looks like the systemd problems had returned. More on that later. First I needed to see if I could run the image without sudo, so I tried 
```bash 
podman run -it registry.fedoraproject.org/f29/httpd
``` 

This worked, with the web server spinning up (despite some initial complaints):
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

```

At this point, I was getting confused and frustrated. Some things worked, some things did not, and I could really explain either set of results. Time to step back and check on some fundamentals.


#### Problem 4: What cgroup version is my WSL2 Ubuntu instance running?
The `Error: systemd cgroup flag ...` error was making me suspicious: I had changed my `~/.config/containers/containers.conf` configuration to use cgroupfs instead of systemd, and to log events to file instead of journald. Why was it still complaining?

The [Podman documentation for Basic Setup and Use of Podman in a rootless environment](https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md) says you needed to change your runtime from `runc` to `crun` depending on whether your system was using cgroups v2 or not. This made me wonder if the cgroupfs change may have also been dependent on the cgroup version?

I needed to figure out how to find this information in my WSL2 Ubuntu instance, and found a solution [here](https://unix.stackexchange.com/questions/471476/how-do-i-check-cgroup-v2-is-installed-on-my-machine). Running `grep cgroup /proc/filesystem` returned:
```bash
nodev   cgroup
nodev   cgroup2
```

This meant that `cgroup v2` was at least available on my system, but I don't know if it was the version I was actually using. 

I tried to change the runtime as per the provided command but couldn't get it to work:
* When I tried `sudo podman --runtime /usr/bin/crun`, I got an `Error: missing command 'podman COMMAND'` error.
* When I tried `podman --runtime /usr/bin/crun`, I got an `Error: missing command 'podman COMMAND'` error.
* When I tried `sudo podman run -it registry.fedoraproject.org/f29/httpd --runtime /usr/bin/crun`, I got the `Error: systemd cgroup flag passed, but systemd support for managing cgroups is not available: OCI runtime error` again.

Thankfully the documentation offers another way to change the runtime via the Podman configuration files. This then led to me to this [page](https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md#user-configuration-files) and proves the value of READING THE DOCUMENTATION! 


#### Problem 5: I temporarily put aside my cgroups version question to figure Podman configuration file override flows
Turns out Podman is configured to check for configuration files in a particular order:
1. /usr/share/containers/containers.conf
1. /etc/containers/containers.conf
1. ~/.config/containers/containers.conf

This gets a bit messier, as it appears `/usr/share/containers/containers.conf` is the base file for BOTH the root user (with overrides coming from `/etc/containers/containers.conf`) and rootless user (with overrides coming from `$HOME/.config/containers/containers.conf`).

At this point, I had spent hours looking for material on the web and was quickly losing confidence that my previous successes were actually successes at all. Had I even been running podman as rootless like I thought I was?


#### Problem 6: Have I been executing as root or non-root?
I needed to be confident that WSL2 account was a user-type and not root-type since this was going to affect which files I had to update. 

I ran `whoami` to confirm that my user account was active, and then ran `id` to retrieve my UID & GID. The sytem returned `uid=1000` and `gid=1000`. I double-checked this by running `cat /etc/passwd/` and confirmed there was a root account with UID/GID 0, whereas my user account was another discrete entry with UID/GID 1000. I was confident I was not the root user (_although I was still a bit suspcious if the use of 'sudo' to invoke podman commands was having some unintended effect_).

Now that I had confirmed I needed to follow the rootless implementation path, it meant I could ignore the files in `/etc/containers/` (because these were mean for the root user) and focus on edits in the `~/.config/containers/containers.conf`. Since I had seen the `cgroup2` entry when I checked my cgroups, I assumed this meant I was running v2 and should configure accordingly (_I happened to be wrong. More on that later_).

 I commented out the `runc` array and uncommented the `crun` array.
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

I wanted to confirm that my changes had actually taken effect so I ran `podman system info`, which resulted in the following:
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

I did notice that `eventLogger: file` was present though, so it made me think some of my earlier changes had stuck. To test this, I edited the `~/.config/containers/containers.conf` and commented out the `events_logger = "file"` line. My expectation was that, when I ran `podman system info` again, I should now see the default `eventLogger: journald`. 

I ran `podman system info` again and found the 'journald' entry in the new output. This proved that changes I made in the user-level config file was being reflected in the podman system info, so I must have missed something when I was changing `runc` to `crun`. Turns out that - although I had uncommented the crun paths - I had forgotten to change the actual runtime entry. I found the line `# runtime = "runc"` and added a new line below `runtime = "crun"`. This change was confirmed when I checked the podman system info again.


#### Problem 7: How the heck do I change my cgroup version!?
I had managed to make other changes in the config file but was still stumped as to how to change the cgroup version. I continued searching but this was quickly devolving into a wild goosechase: 

1. This [stackoverflow post](https://stackoverflow.com/questions/61140609/how-do-i-change-the-cgroup-version-for-podman) asked the exact same question. The single answer mentioned 'systemd' (which clearly wasn't going to help me since WSL2 doesnt have it), or OpenRC (I had no idea what this was). A response to the answer mentioned that for Ubuntu 20.04, the poster ended up editing `/etc/default/grub.d/50-cloudimg-settings.cfg`, followed by update-grub, followed by a reboot, to get podman to show cgroups v2. But the poster didnt say what they had actually added.

1. This [GitHub Gist](https://gist.github.com/trevorwhitney/d83353ff59cc0b7d8ae58116d1fe98f0) said that - to enable cgroups v2 on a Google Cloud instance, you need to edit /etc/default/grub.d/50-cloudimg-settings.cfg and add `cgroup_no_v1=all` to the end of the GRUB_CMDLINE_LINUX_DEFAULT, so it looks something like `GRUB_CMDLINE_LINUX_DEFAULT="console=tty50 cgroup_no_v1=all"`. Now we were quickly veering into exotic bootloader territory and a whole other realm of system knowledge I was going to have to learn.

1. I found other podman bug reports that mentioned the same errors I was encountering, but it was mostly the package developers discussing the inner workings of Linux OS implementations amongst themselves and I had no idea what on earth they were talking about.

I was starting to lose hope. I just wanted to spin up some 'hello world'-type containers - why was I stuck drowning in Linux configuration hell?! Then I had an epiphany: _Maybe I didn't need to change my cgroups version at all_? Did I even need cgroups v2? I only made the changes earlier because of the installation steps saying you had to make changes to accommodate cgroups v2 (_and my assumption that that was what I was running_). But if podman thought it was using cgroups v1, was I ok?

I started thinking about the actual error I received again: `Error: systemd cgroup flag passed, but systemd support for managing cgroups is not available: OCI runtime error`. Even though it didn't show up in the `podman system info` output, I had added an uncommented `cgroup_manager="cgroupfs"` into the file (meant to supplant the default `cgroup_manager="systemd"`. So why was podman complaining about the systemd cgroup flag?!

I took another look at the containers.conf file via `less ~/.config/containers/containers.conf | grep cgroup` and got back the following:
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
I didn't see any missed systemd entries, so I assumed something else had to be the problem.

I then ran a command that I found during one of my searches.
```bash
sudo podman --cgroup-manager=cgroupfs run -it --runtime=/usr/local/sbin/crun docker.io/library/alpine
```
This resulted in:
```bash
Trying to pull docker.io/library/alpine...
Getting image source signatures
Copying blob df20fa9351a1 done
Copying config a24bb40132 done
Writing manifest to image destination
Storing signatures
ERRO[0015] unable to write pod event: "write unixgram @0003d->/run/systemd/journal/socket: sendmsg: no such file or directory"
Error: mount `/sys/fs/cgroup/systemd` to '/sys/fs/cgroup/systemd': No such file or directory: OCI runtime command not found error
```
Something was still messed up.


#### Problem 8: Maybe I missed something else?
I began to re-examine my earlier steps to see if I had messed something up earlier in the process? Reviewing [this podman.io page](https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md) made me these possibilities came to mind:
* Resetting the `crun` changes back to `runc`
* Ensuring other dependencies like `slirp4netns` and `fuse-overlayfs` were installed on my system
* Updating the `storage.conf` file with new values

I confirmed slirp4netns was installed:
```bash
apt list --installed | grep slirpnetns
```

I found and installed `fuse-overlayfs` (version 1.1.2~1):
```bash
sudo apt-cache search fuse-overlayfs
sudo apt-cache show fuse-overlayfs
sudo apt-get install fuse-overlayfs
```

I ensured that both the `containers.conf`, `storage.conf`, and `registries.conf` were populated in the `~/.config/containers/` folder, and changed the ownership from `root` to my own account. Would this finally work? 

With trepidation, I tried running another container with an attached volume:
```bash
> podman run -v ~/OCP4/podmanvolumes:/container/volume docker.io/alpine /bin/bash

Error: error creating runtime static files directory /var/lib/containers/storage/libpod: mkdir /var/lib/containers/storage/libpod: permission denied
ERRO[0000] User-selected graph driver "overlay" overwritten by graph driver "vfs" from database - delete libpod local files to resolve
had to delete ~./local/share/containers/
```
F****.

#### Problem 9: Where on earth are the libpod local files? (A.k.a Graham deletes a few system folders he really shouldn't have)
It turns out that the fact I had run podman BEFORE I did all the configuration meant there was now a system conflict. As per the error message, I could solve it by deleting some local libpod files, but I had no idea where they were! I thought the files might have been the ones I was dealing with in the following locations:
* `/usr/share/containers/`
* `/etc/containers/`
* `~/.config/containers/`

Going full cowboy, I deleted all three folders (without backing up) and ran podman again. 
```bash
ERRO[0000] User-selected graph driver "overlay" overwritten by graph driver "vfs" from database - delete libpod local files to resolve
had to delete ~./local/share/containers/
```
Uhoh - Was I being cursed by configuration files from beyond the grave!?! More Googling ensued, resulting in the discovery that podman stored more artefacts in `~/.local/share/containers/storage/`. This was the folder I should have obliterated. It was quickly hit by `rm -rf` with extreme rage-filled prejudice.

I felt like I was finally nearly a solution. But wait a sec, some of those folders I deleted weren't created by me. Oh crap, did I just delete system folders?!

#### Problem 10: Stupid cowboy, how are we supposed to get those system files back?!



Check ~/.config/containers/storage.conf (missing), copy it in from /etc/containers/storage.conf.
set 'driver = "overlay"'
set 'mount_program = "/usr/bin/fuse-overlayfs'
chownd -R to deeplearning
Didn't fix the unprivileged ping issue (https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md)
Copied /etc/containers/registries.conf back to ~/.config/containers/registries.conf



storage.conf's 'graphroot' key has the value of /var/lib/containers/storage/libpod.

in ~/.config/containers/storage.conf, made the following changes:
graphroot="$HOME/.local/share/containers/storage"
runroot="$XDG_RUNTIME_DIR/containers"

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


### <a name="what-worked">The painful path to figure out the solution</a>
