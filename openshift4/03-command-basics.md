## Podman Command Basics

I'm dumping various commands at the top for reference, with additional commentary below
```bash
# podman search alpine
# podman search --filter=is-official alpine
# podman search --filter=-is-official --no-trunc alpine
# podman search docker.io/postgresql-10 --limit 5

# podman login github.io
# podman search github.io/

# podman pull docker.io/library/alpine:latest
# podman images
# podman inspect docker.io/library/alpine | less
# podman inspect -l -f "{{.NetworkSettings.IPAddress}}"
# skopeo inspect docker://docker.io/library/alpine

# podman run -d -t docker.io/library/ubuntu
# podman run -it docker.io/library/ubuntu /bin/bash
# podman run -it --name helloubuntu docker.io/library/ubuntu /bin/bash
# podman run --name helloubuntu docker.io/library/ubuntu echo 'Hello!'
# podman run -e GREET=Hello -e TARGET=Reader ubuntu printenv GREET TARGET

# podman exec --it <IMAGE NAME> /bin/bash

# podman stop -f <CONTAINER ID or NAME>
# podman restart <CONTAINER ID or NAME>
# podman rm <CONTAINER ID or NAME>
# podman rmi <REPOSITORY NAME>
# podman rm -a  (!!DANGEROUS)

# podman ps -a
# podman ps --format "{{.ID}} {{.Image}} {{.Names}}"
# podman unshare
# podman mount <CONTAINER ID FROM podman ps>
# podman unmount <CONTAINER ID FROM podman ps>

THIS DID NOT STORE DATA IN VOLUME (had to remove '/data:Z' from /var/lib/mysql/data
# podman run -d -v /home/deeplearning/PodmanVolumes/mariadb-data:/var/lib/mysql -e MYSQL_USER=user1 -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -e MYSQL_ROOT_PASSWORD=t00r -p 3306:3306 docker.io/library/mariadb
#     mysql --user=user1 --password=pass -h 127.0.0.1 -P 3306 -t db
#     CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20), species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
#     INSERT INTO pet (name, owner) VALUES ('graham', 'fulgencio');


```

### Searching for and pulling images
For full details on searching for images via podman, see the official [Red Hat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/working-with-container-images_building-running-and-managing-containers). For my own purposes, I'll highlight and expand upon some of the tool's behaviours

NOTE: By default, podman will sequentially search for an image in all the repositories listed in `[registries.search]` section of `registries.conf`. Since I'm using a rootless implementation of podman, any change I make will need to be to the `~/.config/containers/registries.conf` file.

#### Searching for a specific image and/or official image
Running a generic search like `podman search rhel` will return a large amount of images:
```bash
INDEX       NAME                                             DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
docker.io   docker.io/library/alpine                         A minimal Docker image based on Alpine Linux...   6793    [OK]
docker.io   docker.io/anapsix/alpine-java                    Oracle Java 8 (and 7) with GLIBC 2.28 over A...   453                [OK]
docker.io   docker.io/byrnedo/alpine-curl                    Alpine linux with curl installed and set as ...   33                 [OK]
docker.io   docker.io/alpine/git                             A  simple git container running in alpine li...   146                [OK]
docker.io   docker.io/yobasystems/alpine-mariadb             MariaDB running on Alpine Linux [docker] [am...   69                 [OK]
docker.io   docker.io/mhart/alpine-node                      Minimal Node.js built on Alpine Linux             475
docker.io   docker.io/jfloff/alpine-python                   A small, more complete, Python Docker image ...   37                 [OK]
docker.io   docker.io/alpine/socat                           Run socat command in alpine container             58                 [OK]
docker.io   docker.io/hermsi/alpine-fpm-php                  FPM-PHP 7.0 to 7.4, shipped along with tons ...   25                 [OK]
docker.io   docker.io/hermsi/alpine-sshd                     Dockerize your OpenSSH-server with rsync and...   32                 [OK]
docker.io   docker.io/davidcaste/alpine-java-unlimited-jce   Oracle Java 8 (and 7) with GLIBC 2.21 over A...   13                 [OK]
docker.io   docker.io/frolvlad/alpine-glibc                  Alpine Docker image with glibc (~12MB)            247                [OK]
docker.io   docker.io/kiasaki/alpine-postgres                PostgreSQL docker image based on Alpine Linu...   45                 [OK]
docker.io   docker.io/zenika/alpine-chrome                   Chrome running in headless mode in a tiny Al...   24                 [OK]
docker.io   docker.io/etopian/alpine-php-wordpress           Alpine WordPress Nginx PHP-FPM WP-CLI             24                 [OK]
docker.io   docker.io/roribio16/alpine-sqs                   Dockerized ElasticMQ server + web UI over Al...   11                 [OK]
docker.io   docker.io/bushrangers/alpine-caddy               Alpine Linux Docker Container running Caddys...   1                  [OK]
docker.io   docker.io/spotify/alpine                         Alpine image with `bash` and `curl`.              11                 [OK]
docker.io   docker.io/mvertes/alpine-mongo                   light MongoDB container                           116                [OK]
docker.io   docker.io/cfmanteiga/alpine-bash-curl-jq         Docker Alpine image with Bash, curl and jq p...   6                  [OK]
docker.io   docker.io/bashell/alpine-bash                    Alpine Linux with /bin/bash as a default she...   18                 [OK]
docker.io   docker.io/davidcaste/alpine-tomcat               Apache Tomcat 7/8 using Oracle Java 7/8 with...   42                 [OK]
docker.io   docker.io/gliderlabs/alpine                      Image based on Alpine Linux will help you wi...   182
docker.io   docker.io/goodguykoi/alpine-curl-internal        simple alpine image with curl installed no C...   0                  [OK]
docker.io   docker.io/hermsi/alpine-varnish                  Dockerize Varnish upon a lightweight alpine-...   3                  [OK]
quay.io     quay.io/wire/alpine-deps                                                                           0
quay.io     quay.io/wire/alpine-builder                                                                        0
quay.io     quay.io/wire/alpine-intermediate                                                                   0
quay.io     quay.io/wire/alpine-prebuilder                                                                     0
quay.io     quay.io/0xff/alpine-sshd                                                                           0
quay.io     quay.io/wire/alpine-git                                                                            0
quay.io     quay.io/wire/alpine-helm                                                                           0
quay.io     quay.io/watchdogpolska/alpine-curl                                                                 0
quay.io     quay.io/tkaefer/wordpress-fpm-alpine                                                               0
quay.io     quay.io/yanxuehe/alpine-run-as-root                                                                0
quay.io     quay.io/giantswarm/alpine                                                                          0
quay.io     quay.io/opaqnetworks/alpine                                                                        0
quay.io     quay.io/codimd/server                            CodiMD container ===  [![Build Status](https...   0
quay.io     quay.io/karbon/alpine                                                                              0
quay.io     quay.io/aptible/alpine                           ![](https://quay.io/repository/aptible/alpin...   0
quay.io     quay.io/jitesoft/alpine                          # Alpine linux  [![Docker Pulls](https://img...   0
quay.io     quay.io/ukhomeofficedigital/alpine                                                                 0
quay.io     quay.io/signalfuse/collectd-alpine                                                                 0
quay.io     quay.io/kruczjak/docker-alpine-curl                                                                0
quay.io     quay.io/coreos/alpine-sh                                                                           0
quay.io     quay.io/kinvolk/alpine-dig                       A multi-arch Alpine image which contains the...   0
quay.io     quay.io/snapserv/base-alpine                                                                       0
quay.io     quay.io/cogentwebworks/alpine-s6                                                                   0
quay.io     quay.io/beyond/alpine                                                                              0
quay.io     quay.io/eschen42/alpine-cbuilder                 # A build environment for musl to target Alp...   0
```

In most cases, I want to use an official base image released by the maintaining organization. In such cases, I can use the `--filter=is-official` flag to only return 'official' images. Executing `podman search --filter=is-official alpine` cuts the previous list of 50 results down to one:
```bash
INDEX       NAME                       DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
docker.io   docker.io/library/alpine   A minimal Docker image based on Alpine Linux...   6793    [OK]     
``` 

If I wanted to see the full description, I can also include the `--no-trunc` flag. Executing `podman search --filter=-is-official --no-trunc alpine` returns:
```bash
INDEX       NAME                       DESCRIPTION                                                                                         STARS   OFFICIAL   AUTOMATED
docker.io   docker.io/library/alpine   A minimal Docker image based on Alpine Linux with a complete package index and only 5 MB in size!   6793    [OK]
```

"What is an official image?" you may ask. I had the same question and a strangely difficult time trying to find the answer. From what I can gather, an [official image](https://docs.docker.com/docker-hub/official_images/) is one that has gone through an extensive assurance process with the Docker Official Images team (which ultimately controls the repositories in which these images are published). The [docker search documentation](https://docs.docker.com/engine/reference/commandline/search/) notes that the `--is-official` flag looks for _'~images with the searched-for-name, at least 3 stars, and are official builds_'.  Based on some cursory searches I conducted, the returned results are always from the docker.io/library/ namespace, making me think that this must be a Docker-specific repository designation. Long-story short, I'm going to assume the image is ok if it looks something like `docker.io/libary/<image_name_here>`.

#### Search a specific registry for an image
Simply specify the registry before the image search term. For example, `podman search registry.redhat.io/postgresql-10` will only return results from the redhat registry:
```bash
INDEX       NAME                                           DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
redhat.io   registry.redhat.io/rhscl/postgresql-10-rhel7   PostgreSQL is an advanced Object-Relational ...   0
redhat.io   registry.redhat.io/rhel8/postgresql-10         This container image provides a containerize...   0
```

If we were to run the same command but to target the `docker.io` registry instead, we'd only get docker.io images. Running `podman search docker.io/postgresql-10 --limit 5` results in:
```bash
INDEX       NAME                                                      DESCRIPTION                                       STARS   OFFICIAL   AUTOMATED
docker.io   docker.io/centos/postgresql-10-centos7                    PostgreSQL is an advanced Object-Relational ...   18
docker.io   docker.io/crbrka/postgresql-10-postgis                    Postgresql-10 with PostGIS extension.             0
docker.io   docker.io/mcyprian/postgresql-10-fedora29                                                                   0
docker.io   docker.io/centos/postgresql-10-centos8                                                                      0
docker.io   docker.io/outtherelabs/postgresql-10-centos7-extensions   PostgreSQL Extensions Image                       0
```

#### Search a registry for ALL available images
I don't know why you wouldn't want to do this, but if you want to search for every possible available image on an image registry, you need to do the following:
1. Log in to the registry, (e.g. `podman login github.io`)
2. Execute your search, being sure to include a slash at the end (e.g. `podman search github.io/`)


#### Pulling images
Use the `podman pull` command to retrieve images from a remote registry. The command syntax looks like `podman pull <registry>[:<port>]/[<namespace>/]<name>:<tag>`.

While it is possible not to specify the registry, I think this is a good practice to follow as it makes it explicitly clear upfront where we are pulling our image from. For example, running `podman pull alpine` worked but I needed to check the image name to determien that I had pulled it from `docker.io/library/alpine` (the official Docker image). 

Rather than risk pulling the wrong image, I'll always be specifying the full image name during my initial pulls like `podman pull docker.io/library/alpine:latest`


### Interacting with local images
This section handles basic commands for interacting with images once you've copied them into your local image repository

#### List available images
To see the images in your local repository, type `podman images`. This will return something like:
```bash
REPOSITORY                TAG     IMAGE ID      CREATED       SIZE
docker.io/library/ubuntu  latest  9140108b62dc  2 days ago    75.3 MB
docker.io/library/alpine  latest  a24bb4013296  4 months ago  5.85 MB
```

##### Inspect on disk image
To see more details about the image we've downloaded, execute `podman inspect docker.io/library/alpine` (and pipe to `less` for easier reading). This will return something like:
```bash
[
    {
        "Id": "a24bb4013296f61e89ba57005a7b3e52274d8edd3ae2077d04395f806b63d83e",
        "Digest": "sha256:185518070891758909c9f839cf4ca393ee977ac378609f700f60a771a2dfe321",
        "RepoTags": [
            "docker.io/library/alpine:latest"
        ],
        "RepoDigests": [
            "docker.io/library/alpine@sha256:185518070891758909c9f839cf4ca393ee977ac378609f700f60a771a2dfe321",
            "docker.io/library/alpine@sha256:a15790640a6690aa1730c38cf0a440e2aa44aaca9b0e8931a9f2b0d7cc90fd65"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2020-05-29T21:19:46.363518345Z",
        "Config": {
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh"
            ]
        },
        "Version": "18.09.7",
        "Author": "",
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 5849195,
        "VirtualSize": 5849195,
        "GraphDriver": {
            "Name": "overlay",
            "Data": {
                "UpperDir": "/home/deeplearning/.local/share/containers/storage/overlay/50644c29ef5a27c9a40c393a73ece2479de78325cae7d762ef3cdc19bf42dd0a/diff",
                "WorkDir": "/home/deeplearning/.local/share/containers/storage/overlay/50644c29ef5a27c9a40c393a73ece2479de78325cae7d762ef3cdc19bf42dd0a/work"
            }
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:50644c29ef5a27c9a40c393a73ece2479de78325cae7d762ef3cdc19bf42dd0a"
            ]
        },
        "Labels": null,
        "Annotations": {},
        "ManifestType": "application/vnd.docker.distribution.manifest.v2+json",
        "User": "",
        "History": [
            {
                "created": "2020-05-29T21:19:46.192045972Z",
                "created_by": "/bin/sh -c #(nop) ADD file:c92c248239f8c7b9b3c067650954815f391b7bcb09023f984972c082ace2a8d0 in / "
            },
            {
                "created": "2020-05-29T21:19:46.363518345Z",
                "created_by": "/bin/sh -c #(nop)  CMD [\"/bin/sh\"]",
                "empty_layer": true
            }
        ],
        "NamesHistory": []
    }
]
```

Note that the $.GraphDriver key is of type 'overlay' and points to the directory we had specified in the `~/.config/containers/storage.conf` file (as part of the previous configuration effort).

Also of interest is the $.Config.Cmd key (and $.Config.Entrypoint if present). This lets us know that the container will drop you into a bash shell if you started it with `podman run` and did not overwrite it with a commandline argument. As another best practice, I'll always inspect the containers I download - even if they are official images - to ensure that nothing malicious immediately jumps out at me.  

The [Redhat documentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/working-with-container-images_building-running-and-managing-containers) has an interesting example involving `registry.redhat.io/rhel8/rsyslog` that involves an $.Labels.install value, but I've never encountered this workflow so I'll just make a note of it now and revisit later when I know more. 

For a shorter way of invoking the inspect command you can use `podman inspect -l -f "{{.<KEY_PATH_HERE>}}"`. It roughly translates as 'show me the (-l)atest container/image id and (-f)ormat the output using a Go template to specifically return the element(s) defined by the dot notation path  (e.g. .NetworkSettings.IPAddress).
NOTE: This does not appear to work with rootless podman. I'll need to investigate more later.

#### Inspect image on remote registry
We installed Skopeo on our system for a reason and this was it! Instead of potentially wasted hundreds of MBs of download cap, just examine the image on the remote registry!
Using `skopeo inspect docker://docker.io/library/alpine` we get back a response like:
```bash
{
    "Name": "docker.io/library/alpine",
    "Digest": "sha256:185518070891758909c9f839cf4ca393ee977ac378609f700f60a771a2dfe321",
    "RepoTags": [
        "2.6",
        "2.7",
        "20190228",
        "20190408",
        "20190508",
        "20190707",
        "20190809",
        "20190925",
        "20191114",
        "20191219",
        "20200122",
        "20200319",
        "20200428",
        "20200626",
        "20200917",
        "3.1",
        "3.10.0",
        "3.10.1",
        "3.10.2",
        "3.10.3",
        "3.10.4",
        "3.10.5",
        "3.10",
        "3.11.0",
        "3.11.2",
        "3.11.3",
        "3.11.5",
        "3.11.6",
        "3.11",
        "3.12.0",
        "3.12",
        "3.2",
        "3.3",
        "3.4",
        "3.5",
        "3.6.5",
        "3.6",
        "3.7.3",
        "3.7",
        "3.8.4",
        "3.8.5",
        "3.8",
        "3.9.2",
        "3.9.3",
        "3.9.4",
        "3.9.5",
        "3.9.6",
        "3.9",
        "3",
        "edge",
        "latest"
    ],
    "Created": "2020-05-29T21:19:46.363518345Z",
    "DockerVersion": "18.09.7",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:df20fa9351a15782c64e6dddb2d4a6f50bf6d3688060a34c4014b0d9a752eb4c"
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    ]
}
```
I'm not a seasoned Skopeo user yet, so I'm sure there is plenty more one can do. This command provides some basic details and allows you to see all the tagged versions of the image (in case you ever want something older than 'latest'). 

The Skopeo output looks different than what we saw when we inspected through podman, so I suspect we need to specify a version to inspect before we can replicate what we got from `podman inspect ...`.


#### Mount a volume to your container

#### Mount a running container to your local file system
This was something I didn't learn about during the training but saw during subsequent reading of the Red Hat docs. Apparently you can invoke a container in detached mode and then mount it your filesystem to explore it without having to connect to it via an `exec` command. Cool!

The problem is that this doesn't quite work out-of-the-box when you are running a rootless podman. I was able to get this working with some minor modifications (specifically the `unshare` command):
```bash
# podman images
REPOSITORY                TAG     IMAGE ID      CREATED       SIZE
docker.io/library/ubuntu  latest  9140108b62dc  3 days ago    75.3 MB
docker.io/library/alpine  latest  a24bb4013296  4 months ago  5.85 MB

# podman ps
CONTAINER ID  IMAGE                            COMMAND    CREATED       STATUS           PORTS   NAMES
654d90bbf6a7  docker.io/library/ubuntu:latest  /bin/bash  13 hours ago  Up 13 hours ago          epic_bose

# podman unshare
   (this put my shell into a user namespace where I was treated as root, with the prompt showing "root@HOST")
   
# podman mount 654d90bbf6a7
/home/deeplearning/.local/share/containers/storage/overlay/4731890665c7c418c04ffdf57b6f3d963cf82f3f889511753db2b916169377f0/merged

# ls /home/deeplearning/.local/share/containers/storage/overlay/4731890665c7c418c04ffdf57b6f3d963cf82f3f889511753db2b916169377f0/merged
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# exit 
    (this exits the user namespace and returns to my normal shell)
    
# ls /home/deeplearning/.local/share/containers/storage/overlay/4731890665c7c418c04ffdf57b6f3d963cf82f3f889511753db2b916169377f0/merged
    (nothing)
```

The Red Hat example also had a way to check the packages installed on the mounted rhel image via `# rpm -qa --root=/var/lib/containers/storage/overlay/65881e78.../merged`. Since Ubuntu doesn't have rpm, I had to modify the command to work with `apt`:
```bash
# apt list --installed -o Dir=/home/deeplearning/.local/share/containers/storage/overlay/4731890665c7c418c04ffdf57b6f3d963cf82f3f889511753db2b916169377f0/merged

Listing... Done
adduser/now 3.118ubuntu2 all [installed,local]
apt/now 2.0.2ubuntu0.1 amd64 [installed,local]
base-files/now 11ubuntu5.2 amd64 [installed,local]
base-passwd/now 3.5.47 amd64 [installed,local]
bash/now 5.0-6ubuntu1.1 amd64 [installed,local]
bsdutils/now 1:2.34-0.1ubuntu9.1 amd64 [installed,local]
bzip2/now 1.0.8-2 amd64 [installed,local]
coreutils/now 8.30-3ubuntu2 amd64 [installed,local]
dash/now 0.5.10.2-6 amd64 [installed,local]
debconf/now 1.5.73 all [installed,local]
debianutils/now 4.9.1 amd64 [installed,local]
diffutils/now 1:3.7-3 amd64 [installed,local]
dpkg/now 1.19.7ubuntu3 amd64 [installed,local]
e2fsprogs/now 1.45.5-2ubuntu1 amd64 [installed,local]
fdisk/now 2.34-0.1ubuntu9.1 amd64 [installed,local]
findutils/now 4.7.0-1ubuntu1 amd64 [installed,local]
gcc-10-base/now 10-20200411-0ubuntu1 amd64 [installed,local]
gpgv/now 2.2.19-3ubuntu2 amd64 [installed,local]
grep/now 3.4-1 amd64 [installed,local]
gzip/now 1.10-0ubuntu4 amd64 [installed,local]
hostname/now 3.23 amd64 [installed,local]
init-system-helpers/now 1.57 all [installed,local]
libacl1/now 2.2.53-6 amd64 [installed,local]
libapt-pkg6.0/now 2.0.2ubuntu0.1 amd64 [installed,local]
libattr1/now 1:2.4.48-5 amd64 [installed,local]
libaudit-common/now 1:2.8.5-2ubuntu6 all [installed,local]
libaudit1/now 1:2.8.5-2ubuntu6 amd64 [installed,local]
libblkid1/now 2.34-0.1ubuntu9.1 amd64 [installed,local]
libbz2-1.0/now 1.0.8-2 amd64 [installed,local]
libc-bin/now 2.31-0ubuntu9.1 amd64 [installed,local]
libc6/now 2.31-0ubuntu9.1 amd64 [installed,local]
libcap-ng0/now 0.7.9-2.1build1 amd64 [installed,local]
libcom-err2/now 1.45.5-2ubuntu1 amd64 [installed,local]
libcrypt1/now 1:4.4.10-10ubuntu4 amd64 [installed,local]
libdb5.3/now 5.3.28+dfsg1-0.6ubuntu2 amd64 [installed,local]
libdebconfclient0/now 0.251ubuntu1 amd64 [installed,local]
libext2fs2/now 1.45.5-2ubuntu1 amd64 [installed,local]
libfdisk1/now 2.34-0.1ubuntu9.1 amd64 [installed,local]
libffi7/now 3.3-4 amd64 [installed,local]
libgcc-s1/now 10-20200411-0ubuntu1 amd64 [installed,local]
libgcrypt20/now 1.8.5-5ubuntu1 amd64 [installed,local]
libgmp10/now 2:6.2.0+dfsg-4 amd64 [installed,local]
libgnutls30/now 3.6.13-2ubuntu1.3 amd64 [installed,local]
libgpg-error0/now 1.37-1 amd64 [installed,local]
libhogweed5/now 3.5.1+really3.5.1-2 amd64 [installed,local]
libidn2-0/now 2.2.0-2 amd64 [installed,local]
liblz4-1/now 1.9.2-2 amd64 [installed,local]
liblzma5/now 5.2.4-1ubuntu1 amd64 [installed,local]
libmount1/now 2.34-0.1ubuntu9.1 amd64 [installed,local]
libncurses6/now 6.2-0ubuntu2 amd64 [installed,local]
libncursesw6/now 6.2-0ubuntu2 amd64 [installed,local]
libnettle7/now 3.5.1+really3.5.1-2 amd64 [installed,local]
libp11-kit0/now 0.23.20-1build1 amd64 [installed,local]
libpam-modules-bin/now 1.3.1-5ubuntu4.1 amd64 [installed,local]
libpam-modules/now 1.3.1-5ubuntu4.1 amd64 [installed,local]
libpam-runtime/now 1.3.1-5ubuntu4.1 all [installed,local]
libpam0g/now 1.3.1-5ubuntu4.1 amd64 [installed,local]
libpcre2-8-0/now 10.34-7 amd64 [installed,local]
libpcre3/now 2:8.39-12build1 amd64 [installed,local]
libprocps8/now 2:3.3.16-1ubuntu2 amd64 [installed,local]
libseccomp2/now 2.4.3-1ubuntu3.20.04.3 amd64 [installed,local]
libselinux1/now 3.0-1build2 amd64 [installed,local]
libsemanage-common/now 3.0-1build2 all [installed,local]
libsemanage1/now 3.0-1build2 amd64 [installed,local]
libsepol1/now 3.0-1 amd64 [installed,local]
libsmartcols1/now 2.34-0.1ubuntu9.1 amd64 [installed,local]
libss2/now 1.45.5-2ubuntu1 amd64 [installed,local]
libstdc++6/now 10-20200411-0ubuntu1 amd64 [installed,local]
libsystemd0/now 245.4-4ubuntu3.2 amd64 [installed,local]
libtasn1-6/now 4.16.0-2 amd64 [installed,local]
libtinfo6/now 6.2-0ubuntu2 amd64 [installed,local]
libudev1/now 245.4-4ubuntu3.2 amd64 [installed,local]
libunistring2/now 0.9.10-2 amd64 [installed,local]
libuuid1/now 2.34-0.1ubuntu9.1 amd64 [installed,local]
libzstd1/now 1.4.4+dfsg-3 amd64 [installed,local]
login/now 1:4.8.1-1ubuntu5.20.04 amd64 [installed,local]
logsave/now 1.45.5-2ubuntu1 amd64 [installed,local]
lsb-base/now 11.1.0ubuntu2 all [installed,local]
mawk/now 1.3.4.20200120-2 amd64 [installed,local]
mount/now 2.34-0.1ubuntu9.1 amd64 [installed,local]
ncurses-base/now 6.2-0ubuntu2 all [installed,local]
ncurses-bin/now 6.2-0ubuntu2 amd64 [installed,local]
passwd/now 1:4.8.1-1ubuntu5.20.04 amd64 [installed,local]
perl-base/now 5.30.0-9build1 amd64 [installed,local]
procps/now 2:3.3.16-1ubuntu2 amd64 [installed,local]
sed/now 4.7-1 amd64 [installed,local]
sensible-utils/now 0.0.12+nmu1 all [installed,local]
sysvinit-utils/now 2.96-2.1ubuntu1 amd64 [installed,local]
tar/now 1.30+dfsg-7 amd64 [installed,local]
ubuntu-keyring/now 2020.02.11.2 all [installed,local]
util-linux/now 2.34-0.1ubuntu9.1 amd64 [installed,local]
zlib1g/now 1:1.2.11.dfsg-2ubuntu1 amd64 [installed,local]
```

I assume once you are done with the container you need to unmount & exit. I did the following and didn't get any errors, but the mount path remained on the host system (albeit without any of the container files). Not sure if I'm missing a step here, I'll come back when I know more:
```bash
# podman unmount 654d90bbf6a7

# podman exit
    (this drops you back into the normal shell)
    
# root@HOST:~$ ls /home/deeplearning/.local/share/containers/storage/overlay/4731890665c7c418c04ffdf57b6f3d963cf82f3f889511753db2b916169377f0/merged
    (nothing)
```

To be honest, I don't really understand why all of this actually works. I'll try expanding on namespaces, UIDs, GIDs in the [Background](./02-background-context.md) over time.

Update: Turns out the `~/.local/share/containers/storage/overlay/` folder is important! 

Picking up from my `unmount` and `exit` step, I decided I didn't like having all these long-named folders in my overlay folder so ... I `rm -rf`ed them. I then tried to create a new ubuntu container and got an error:
```bash
# podman run -d -t ubuntu
Error: stat /home/deeplearning/.local/share/containers/storage/overlay/cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b: no such file or directory
```

My system reported that it still had the ubuntu image, so clearly something was up:
```bash
# podman images
REPOSITORY                TAG     IMAGE ID      CREATED       SIZE
docker.io/library/ubuntu  latest  9140108b62dc  3 days ago    75.3 MB
docker.io/library/alpine  latest  a24bb4013296  4 months ago  5.85 MB
```

I assumed I had deleted a folder(s) in the `~/.local/share/containers/storage/overlay/` folder that belonged to the base image rather than the running container I had mounted in my previous efforts. To test this, I purged the local image registry and pulled down a new copy of ubuntu, and then checked the contents of the previously empty `overlay` folder:
```bash
# podman pull docker.io/library/ubuntu
Trying to pull docker.io/library/ubuntu...
Getting image source signatures
Copying blob d72e567cc804 done
Copying blob b6a83d81d1f4 done
Copying blob 0f3630e5ff08 done
Copying config 9140108b62 done
Writing manifest to image destination
Storing signatures
9140108b62dc87d9b278bb0d4fd6a3e44c2959646eb966b86531306faa81b09b

# ls -al /home/deeplearning/.local/share/containers/storage/overlay/
total 24
drwx------ 6 root root 4096 Sep 29 12:04 .
drwx------ 9 root root 4096 Sep 27 15:15 ..
drwx------ 5 root root 4096 Sep 29 12:04 69ea2c30b91fcfb00060eac2734467275f0e3549858a877a92efad72c7cc21ea
drwx------ 5 root root 4096 Sep 29 12:04 cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b
drwx------ 6 root root 4096 Sep 29 12:04 d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92
drwx------ 2 root root 4096 Sep 29 12:04 l
```

I then created a running detached instance of the ubuntu image and checked again:
```bash
# podman run -d -t docker.io/library/ubuntu
8c4b2e29c8e6bb68fadf1f218221e29b77db8b9be01c48c7762cc1219c803c36

# ls -al /home/deeplearning/.local/share/containers/storage/overlay/
total 28
drwx------ 7 root root 4096 Sep 29 12:04 .
drwx------ 9 root root 4096 Sep 27 15:15 ..
drwx------ 5 root root 4096 Sep 29 12:04 69ea2c30b91fcfb00060eac2734467275f0e3549858a877a92efad72c7cc21ea
drwx------ 5 root root 4096 Sep 29 12:04 cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b
drwx------ 6 root root 4096 Sep 29 12:04 d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92
drwx------ 5 root root 4096 Sep 29 12:04 f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993
drwx------ 2 root root 4096 Sep 29 12:04 l
```

There was a new entry! But ... the container id reported by the `run` command didnt' match the layer? Hrm. Let's attach it and see what happens:
```bash
# podman mount 8c4b2e29c8e6
/home/deeplearning/.local/share/containers/storage/overlay/f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993/merged
```

Ok, so I can conclude the `f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993` folder is tied to the ubuntu container image I instantiated, while the other three folders are related to the base ubuntu image itself. I checked the base ubuntu image with the `podman inspect docker.io/library/ubuntu` command and looked at the $.RootFS entry. The `d42a4fd...` entry matched, but the other two did not. Maybe the others are different because I'm using the overlayfs configuration? No idea.
```bash
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92",
                "sha256:90ac32a0d9ab11e7745283f3051e990054616d631812ac63e324c1a36d2677f5",
                "sha256:782f5f011ddaf2a0bfd38cc2ccabd634095d6e35c8034302d788423f486bb177"
            ]
        },
```

I didn't feel like going much further down this rabbithole, but I wanted to make sure I was on the right track so I repeated the process with an alpine instance.
```bash
# podman pull docker.io/library/alpine
Trying to pull docker.io/library/alpine...
Getting image source signatures
Copying blob df20fa9351a1 done
Copying config a24bb40132 done
Writing manifest to image destination
Storing signatures
a24bb4013296f61e89ba57005a7b3e52274d8edd3ae2077d04395f806b63d83e

# ls -al /home/deeplearning/.local/share/containers/storage/overlay/
total 32
drwx------ 8 root root 4096 Sep 29 12:16 .
drwx------ 9 root root 4096 Sep 27 15:15 ..
drwx------ 6 root root 4096 Sep 29 12:16 50644c29ef5a27c9a40c393a73ece2479de78325cae7d762ef3cdc19bf42dd0a
drwx------ 5 root root 4096 Sep 29 12:04 69ea2c30b91fcfb00060eac2734467275f0e3549858a877a92efad72c7cc21ea
drwx------ 5 root root 4096 Sep 29 12:04 cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b
drwx------ 6 root root 4096 Sep 29 12:04 d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92
drwx------ 5 root root 4096 Sep 29 12:04 f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993
drwx------ 2 root root 4096 Sep 29 12:16 l
```

There was a new `50644c29...` entry. When I instantiated the alpine container, another folder appeared `2c3c214...`.
```bash
# podman run -d -t docker.io/library/alpine
8a399101835b8f6a5a797c4221fa0c93f2b27142e87d596faa8394750a3cbeca

# ls -al /home/deeplearning/.local/share/containers/storage/overlay/
total 36
drwx------ 9 deeplearning deeplearning 4096 Sep 29 12:18 .
drwx------ 9 deeplearning deeplearning 4096 Sep 27 15:15 ..
drwx------ 5 deeplearning deeplearning 4096 Sep 29 12:18 2c3c21491f951807f323ddf34f8de29cc20ccceb3365cd91d0401c36f8e78d11
drwx------ 6 deeplearning deeplearning 4096 Sep 29 12:16 50644c29ef5a27c9a40c393a73ece2479de78325cae7d762ef3cdc19bf42dd0a
drwx------ 5 deeplearning deeplearning 4096 Sep 29 12:04 69ea2c30b91fcfb00060eac2734467275f0e3549858a877a92efad72c7cc21ea
drwx------ 5 deeplearning deeplearning 4096 Sep 29 12:04 cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b
drwx------ 6 deeplearning deeplearning 4096 Sep 29 12:04 d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92
drwx------ 5 deeplearning deeplearning 4096 Sep 29 12:04 f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993
drwx------ 2 deeplearning deeplearning 4096 Sep 29 12:18 l
```

Mounting the alpine instance resulted in the system using the ``2c3c214...`:
```bash
# podman ps
CONTAINER ID  IMAGE                            COMMAND    CREATED         STATUS             PORTS   NAMES
8a399101835b  docker.io/library/alpine:latest  /bin/sh    2 minutes ago   Up 2 minutes ago           pedantic_mirzakhani
8c4b2e29c8e6  docker.io/library/ubuntu:latest  /bin/bash  16 minutes ago  Up 16 minutes ago          cool_bartik

# podman mount 8a399101835b
/home/deeplearning/.local/share/containers/storage/overlay/2c3c21491f951807f323ddf34f8de29cc20ccceb3365cd91d0401c36f8e78d11/merged
```

But what happens if I delete the container instance?
```bash
# podman stop 8a399101835b
8a399101835b8f6a5a797c4221fa0c93f2b27142e87d596faa8394750a3cbeca

# podman ps -a
CONTAINER ID  IMAGE                            COMMAND    CREATED         STATUS                       PORTS   NAMES
8a399101835b  docker.io/library/alpine:latest  /bin/sh    5 minutes ago   Exited (137) 11 seconds ago          pedantic_mirzakhani
8c4b2e29c8e6  docker.io/library/ubuntu:latest  /bin/bash  19 minutes ago  Up 19 minutes ago                    cool_bartik

# ls -al /home/deeplearning/.local/share/containers/storage/overlay/
total 36
drwx------ 9 root root 4096 Sep 29 12:18 .
drwx------ 9 root root 4096 Sep 27 15:15 ..
drwx------ 5 root root 4096 Sep 29 12:18 2c3c21491f951807f323ddf34f8de29cc20ccceb3365cd91d0401c36f8e78d11
drwx------ 6 root root 4096 Sep 29 12:16 50644c29ef5a27c9a40c393a73ece2479de78325cae7d762ef3cdc19bf42dd0a
drwx------ 5 root root 4096 Sep 29 12:04 69ea2c30b91fcfb00060eac2734467275f0e3549858a877a92efad72c7cc21ea
drwx------ 5 root root 4096 Sep 29 12:04 cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b
drwx------ 6 root root 4096 Sep 29 12:04 d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92
drwx------ 5 root root 4096 Sep 29 12:04 f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993
drwx------ 2 root root 4096 Sep 29 12:18 l

# podman rm 8a399101835b
8a399101835b8f6a5a797c4221fa0c93f2b27142e87d596faa8394750a3cbeca

# ls -al /home/deeplearning/.local/share/containers/storage/overlay/
total 32
drwx------ 8 root root 4096 Sep 29 12:24 .
drwx------ 9 root root 4096 Sep 27 15:15 ..
drwx------ 6 root root 4096 Sep 29 12:16 50644c29ef5a27c9a40c393a73ece2479de78325cae7d762ef3cdc19bf42dd0a
drwx------ 5 root root 4096 Sep 29 12:04 69ea2c30b91fcfb00060eac2734467275f0e3549858a877a92efad72c7cc21ea
drwx------ 5 root root 4096 Sep 29 12:04 cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b
drwx------ 6 root root 4096 Sep 29 12:04 d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92
drwx------ 5 root root 4096 Sep 29 12:04 f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993
drwx------ 2 root root 4096 Sep 29 12:24 l
```

Ok, so here's what I can conclude:
* Podman stores base image layers in `$/.local/share/container/storage/overlay/`
* This storage path coincides with the `graphroot` and `rootless_storage_path` settings I specified in my `~/.config/containers/storage.conf` file during initial setup.
* Instantiated containers will also be written to this same folder.
* The instantiated container folder will disappear when I `podman rm <container>` the container
* The image folder(s) will be removed when I `podman rmi <image>` the image

_Note to self: cease deleting folders in ~/.local/share/containers/storage/overlay/ !!_

As a follow-up, I was reviewing the entries and wondered why I saw that the image folders belonged to root - hadn't I been doing this rootless?! I haven't included the CLI prompt in my system outpot, but most of this material was taken AFTER I had executed `podman unshare` (which put me into a custom user namespace where I was treated as root). Once I exited the custom name space (reverting back to my normal shell where I'm user `deeplearning`), permissions began showing as belonging to the `deeplearning` UID and GID:
```bash
root@DESKTOP-MC34QIL:~# ls -al /home/deeplearning/.local/share/containers/storage/overlay/
total 28
drwx------ 7 root root 4096 Sep 29 12:30 .
drwx------ 9 root root 4096 Sep 27 15:15 ..
drwx------ 5 root root 4096 Sep 29 12:04 69ea2c30b91fcfb00060eac2734467275f0e3549858a877a92efad72c7cc21ea
drwx------ 5 root root 4096 Sep 29 12:04 cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b
drwx------ 6 root root 4096 Sep 29 12:04 d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92
drwx------ 5 root root 4096 Sep 29 12:04 f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993
drwx------ 2 root root 4096 Sep 29 12:30 l

root@DESKTOP-MC34QIL:~# exit
exit

deeplearning@DESKTOP-MC34QIL:~$ ls -al /home/deeplearning/.local/share/containers/storage/overlay/
total 28
drwx------ 7 deeplearning deeplearning 4096 Sep 29 12:30 .
drwx------ 9 deeplearning deeplearning 4096 Sep 27 15:15 ..
drwx------ 5 deeplearning deeplearning 4096 Sep 29 12:04 69ea2c30b91fcfb00060eac2734467275f0e3549858a877a92efad72c7cc21ea
drwx------ 5 deeplearning deeplearning 4096 Sep 29 12:04 cbca8e9eddab4a2a6f403394c59c0be317a50c5ee826c24cfbc7f7519686148b
drwx------ 6 deeplearning deeplearning 4096 Sep 29 12:04 d42a4fdf4b2ae8662ff2ca1b695eae571c652a62973c1beb81a296a4f4263d92
drwx------ 5 deeplearning deeplearning 4096 Sep 29 12:04 f7cf6801755e051f1ca5d0111572d432a332fdebbdefe46dd64d6883961c7993
drwx------ 2 deeplearning deeplearning 4096 Sep 29 12:30 l
```

### Set up database with stateful storage
Stateless containers are great but I'm going to need to save state at some point, so I set about quickly figuring this out.

Older articles (circa 2018, like the [Managing containerized system services with Podman](https://developers.redhat.com/blog/2018/11/29/managing-containerized-system-services-with-podman/)) note a step that requires changing the ownership of the local volume that is being mounted into the container. This is so that the container process can successfully write data into the folder, ensuring persistence should that container ever be destroyed. 

Given that I'm experimented with Podman late September 2020, was this still true? I set out to find out. To do my test I needed to:
* Download a database container image (_I chose mariadb since I had already seen a few steps involved in setting up an instance_)
* Mount a local volume inside the container, where the mariadb process stored its data
* Add a record to the database
* Exit and destroy the container
* Instantiate a new container image, using the same mount parameter as before
* See if the data had persisted

```bash
# podman pull docker.io/library/mariadb
Trying to pull docker.io/library/mariadb...
Getting image source signatures
Copying blob d72e567cc804 skipped: already exists
...
Copying config 41fa9265d4 done
Writing manifest to image destination
Storing signatures
41fa9265d4dfb214f0a79ee919392687d09babc3522df21fce946292f9c8149c

# mkdir -p ~/PodmanVolume/mariadb-data

# ls -al /home/deeplearning/PodmanVolumes
drwxr-xr-x  4 deeplearning deeplearning 4096 Sep 30 12:32 .
drwxr-xr-x 14 deeplearning deeplearning 4096 Sep 30 16:07 ..
drwxr-xr-x  5 deeplearning deeplearning 4096 Sep 30 12:52 mariadb-data

# podman run -d -v /home/deeplearning/PodmanVolumes/mariadb-data:/var/lib/mysql -e MYSQL_USER=user1 -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -e MYSQL_ROOT_PASSWORD=t00r -p 3306:3306 docker.io/library/mariadb

# podman ps
CONTAINER ID  IMAGE                             COMMAND  CREATED         STATUS             PORTS                   NAMES
2c9b9f7bac39  docker.io/library/mariadb:latest  mysqld   13 minutes ago  Up 13 minutes ago  0.0.0.0:3306->3306/tcp  eager_lamarr

# ls -al /home/deeplearning/PodmanVolumes
drwxr-xr-x  4 deeplearning deeplearning 4096 Sep 30 12:32 .
drwxr-xr-x 14 deeplearning deeplearning 4096 Sep 30 16:07 ..
drwxr-xr-x  5       100998 deeplearning 4096 Sep 30 16:15 mariadb-data

# ls -al mariadb-data/
total 122944
drwxr-xr-x 5       100998 deeplearning      4096 Sep 30 16:18 .
drwxr-xr-x 4 deeplearning deeplearning      4096 Sep 30 16:16 ..
-rw-rw---- 1       100998       100998     32768 Sep 30 16:17 aria_log.00000001
-rw-rw---- 1       100998       100998        52 Sep 30 16:17 aria_log_control
drwx------ 2       100998       100998      4096 Sep 30 16:17 db
-rw-rw---- 1       100998       100998       992 Sep 30 16:17 ib_buffer_pool
-rw-rw---- 1       100998       100998 100663296 Sep 30 16:18 ib_logfile0
-rw-rw---- 1       100998       100998  12582912 Sep 30 16:17 ibdata1
-rw-rw---- 1       100998       100998  12582912 Sep 30 16:18 ibtmp1
-rw-rw---- 1       100998       100998         0 Sep 30 16:17 multi-master.info
drwx------ 2       100998       100998      4096 Sep 30 16:17 mysql
drwx------ 2       100998       100998      4096 Sep 30 16:17 performance_schema

# podman logs eager_lamarr
2020-09-30 20:18:03+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 1:10.5.5+maria~focal started.
2020-09-30 20:18:03+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2020-09-30 20:18:03+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 1:10.5.5+maria~focal started.
2020-09-30 20:18:04 0 [Note] mysqld (mysqld 10.5.5-MariaDB-1:10.5.5+maria~focal) starting as process 1 ...
2020-09-30 20:18:04 0 [Warning] Could not increase number of max_open_files to more than 4096 (request: 32186)
2020-09-30 20:18:04 0 [Warning] Changed limits: max_open_files: 4096  max_connections: 151 (was 151)  table_cache: 1957 (was 2000)
2020-09-30 20:18:04 0 [Note] InnoDB: Using Linux native AIO
2020-09-30 20:18:04 0 [Note] InnoDB: Uses event mutexes
2020-09-30 20:18:04 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2020-09-30 20:18:04 0 [Note] InnoDB: Number of pools: 1
2020-09-30 20:18:04 0 [Note] InnoDB: Using SSE4.2 crc32 instructions
2020-09-30 20:18:04 0 [Note] mysqld: O_TMPFILE is not supported on /tmp (disabling future attempts)
2020-09-30 20:18:04 0 [Note] InnoDB: Initializing buffer pool, total size = 134217728, chunk size = 134217728
2020-09-30 20:18:04 0 [Note] InnoDB: Completed initialization of buffer pool
2020-09-30 20:18:04 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2020-09-30 20:18:04 0 [Note] InnoDB: 128 rollback segments are active.
2020-09-30 20:18:04 0 [Note] InnoDB: Creating shared tablespace for temporary tables
2020-09-30 20:18:04 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
2020-09-30 20:18:04 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
2020-09-30 20:18:04 0 [Note] InnoDB: 10.5.5 started; log sequence number 47663; transaction id 33
2020-09-30 20:18:04 0 [Note] InnoDB: Loading buffer pool(s) from /var/lib/mysql/ib_buffer_pool
2020-09-30 20:18:04 0 [Note] Plugin 'FEEDBACK' is disabled.
2020-09-30 20:18:04 0 [Note] InnoDB: Buffer pool(s) load completed at 200930 20:18:04
2020-09-30 20:18:04 0 [Note] Server socket created on IP: '::'.
2020-09-30 20:18:04 0 [Warning] 'proxies_priv' entry '@% root@21cc644dd03c' ignored in --skip-name-resolve mode.
2020-09-30 20:18:04 0 [Note] Reading of all Master_info entries succeeded
2020-09-30 20:18:04 0 [Note] Added new Master_info '' to hash table
2020-09-30 20:18:04 0 [Note] mysqld: ready for connections.
Version: '10.5.5-MariaDB-1:10.5.5+maria~focal'  socket: '/run/mysqld/mysqld.sock'  port: 3306  mariadb.org binary distribution

# mysql --user=user1 --password=pass -h 127.0.0.1 -P 3306 -t db
    (this dropped me into the SQL prompt on the container)
    
> CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20), species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
    Query OK, 0 rows affected (0.04 sec)

> mysql> INSERT INTO pet (name, owner) VALUES ('graham', 'fulgencio');
    Query OK, 1 row affected (0.01 sec)
    
> exit
    Bye
    
# podman rm -f eager_lamarr

# podman run -d -v /home/deeplearning/PodmanVolumes/mariadb-data:/var/lib/mysql -e MYSQL_USER=user1 -e MYSQL_PASSWORD=pass -e MYSQL_DATABASE=db -e MYSQL_ROOT_PASSWORD=t00r -p 3306:3306 docker.io/library/mariadb
3757a2216835de45dc21662ac9275ae99aee838a8a2c613ed9e5ad13021b4246

# podman ps
CONTAINER ID  IMAGE                             COMMAND  CREATED        STATUS            PORTS                   NAMES
3757a2216835  docker.io/library/mariadb:latest  mysqld   4 seconds ago  Up 4 seconds ago  0.0.0.0:3306->3306/tcp  flamboyant_ellis

# mysql --user=user1 --password=pass -h 127.0.0.1 -P 3306 -t db
   (this dropped me into the SQL prompt on the container)
   
> SELECT * from pet;
+--------+-----------+---------+------+-------+-------+
| name   | owner     | species | sex  | birth | death |
+--------+-----------+---------+------+-------+-------+
| graham | fulgencio | NULL    | NULL | NULL  | NULL  |
+--------+-----------+---------+------+-------+-------+
1 row in set (0.00 sec)

```

As you can see, Podman automatically invoked a `chown` on the /home/deepleearning/PodmanVolumes/mariadb-test/ folder, setting it to another UID. The newly-instantiated MariaDB database was then able to save its initial state and the additional record I later added to the database. When I deleted the first container and replaced it with another, the new MariaDB container was able to verify my user identity and return the data I had saved via the first container. Success!

Update: As I look over my Redhat training guide (DO180, p.57), this may still be true for some OSes? The training used RHEL and required SELinux changes to a local directory before a MySql container could save files to that mounted volume.
```bash
# sudo mkdir /var/dbfiles
# sudo chown -R 27:27 /var/dbfiles
# sudo semanage fcontext -a -t container_file_t '/var/dbfiles(/.*)?'
# ls -dZ /var/dbfiles
# sudo restorecon -Rv /var/dbfiles
```

### Limiting Container Resources
While this focused on the `docker` tool, [MariaDB and Docker use cases, Part 1](https://mariadb.com/resources/blog/mariadb-and-docker-use-cases-part-1/) has two interesting ideas that I'll take a second to note:
1. Running instances concurrently on the same host.<br>The article mentions that solutions like MySQL Sandbox do exist to facilitate this, but it's much easier to just spin up three containers, with each pulling a different image based on the iamge tage (e.g. docker.io/library/mariadb:10.0) and mapping a different host port.
1. Resource limiting like:
    1.1. CPU percentage sharing (via `--cpu-shares`)
    1.1. CPU assignemtn (via `--cpuset-cpus`)
    1.1. Block IO sharing (via `--blkio-weight`)
    1.1. Memory limitation (via `--memory`)
    
### Running Pods Rather than Containers
While Kubernetes is the likely pod-wrangling solution for Production systems, I don't particularly have to run it locally if I can avoid it. As on might have figured out from its name, podman ("pod manager") can do this too!

If running multiple containers locally, using pods starts to make sense to facilitate networking between containers. [Podman: Managing pods and containers in a local container runtime](https://developers.redhat.com/blog/2019/01/15/podman-managing-containers-pods/), an early 2019 article by Brent Baude, describes how it works:
* Each pod contains an "Infra Container" that is responsible for:
    * Reserving the namespace associated with the pod, other vital attributes like port bindings, cgroup values, etc.
    * Used by Podman to connect other containers to the pod.
    * Is used to control the state of the POD (whereas you can still turn individual container on/off without impacting the overall pod state).

The Baude article is a little out-of-date: the results returned by `podman pod create --help` are a bit different for Podman v2.1.0 (which I'm using) versus whatever version Baude was using at the start of 2019, but it's close enough to still be worth reading. I followed Baude's material, re-executing the commands against my own version to see what still worked (I left in my previous MariaDB singleton container to see the differences when running `ps`:

```bash
# podman create pod
e94689355ac7c6954e2d86335cba190230f7144112ef6f207c156be0a73274c7

# podman pod list
POD ID        NAME         STATUS   CREATED         # OF CONTAINERS  INFRA ID
e94689355ac7  bold_banach  Created  48 seconds ago  1                a4fa2fc0619d

# podman ps -a --pod
CONTAINER ID  IMAGE                             COMMAND  CREATED         STATUS          PORTS                   NAMES               POD ID        PODNAME
a4fa2fc0619d  k8s.gcr.io/pause:3.2                       11 minutes ago  Created                                 e94689355ac7-infra  e94689355ac7  bold_banach
3757a2216835  docker.io/library/mariadb:latest  mysqld   2 hours ago     Up 2 hours ago  0.0.0.0:3306->3306/tcp  flamboyant_ellis  

# podman run -d -t --pod bold_banach docker.io/library/alpine top
a700757aca10259f9ef1ebfb1dd484e76c53dec03ab92a4324bb955770262740

# podman pod ps
POD ID        NAME         STATUS   CREATED         # OF CONTAINERS  INFRA ID
e94689355ac7  bold_banach  Running  13 minutes ago  2                a4fa2fc0619d

# podman ps -a --pod
CONTAINER ID  IMAGE                             COMMAND  CREATED         STATUS            PORTS                   NAMES               POD ID        PODNAME
a700757aca10  docker.io/library/alpine:latest   top      2 minutes ago   Up 2 minutes ago                          crazy_shtern        e94689355ac7  bold_banach
a4fa2fc0619d  k8s.gcr.io/pause:3.2                       15 minutes ago  Up 2 minutes ago                          e94689355ac7-infra  e94689355ac7  bold_banach
3757a2216835  docker.io/library/mariadb:latest  mysqld   2 hours ago     Up 2 hours ago    0.0.0.0:3306->3306/tcp  flamboyant_ellis 
```

Looks like all the commands are still working at this point, let's try the commands that create a pod and container in the same command. But first let's remind ourselves which containers and pods are already on the system:
```bash
# podman ps -a --pod
CONTAINER ID  IMAGE                             COMMAND  CREATED      STATUS          PORTS                   NAMES               POD ID        PODNAME
a700757aca10  docker.io/library/alpine:latest   top      2 hours ago  Up 2 hours ago                          crazy_shtern        e94689355ac7  bold_banach
a4fa2fc0619d  k8s.gcr.io/pause:3.2                       2 hours ago  Up 2 hours ago                          e94689355ac7-infra  e94689355ac7  bold_banach
3757a2216835  docker.io/library/mariadb:latest  mysqld   4 hours ago  Up 4 hours ago  0.0.0.0:3306->3306/tcp  flamboyant_ellis

# podman pod ps
POD ID        NAME         STATUS   CREATED      # OF CONTAINERS  INFRA ID
e94689355ac7  bold_banach  Running  3 hours ago  2                a4fa2fc0619d
```

Now let's try creating a pod and container at the same time:
```bash
# podman pull docker.io/library/nginx
...
Storing signatures
7e4d58f0e5f3b60077e9a5d96b4be1b974b5a484f54f9393000a99f3b6816e3d

# podman run -d -t --pod new:nginx -p 32597:80 docker.io/library/nginx
73d129c7edbbb703f9739164ba03524381ccbaf113c9a947efdf36bc341f6a10

# podman pod ps
POD ID        NAME         STATUS   CREATED         # OF CONTAINERS  INFRA ID
f59faabb7cd3  nginx        Running  33 seconds ago  2                e4a44f543a70
e94689355ac7  bold_banach  Running  3 hours ago     2                a4fa2fc0619d

# podman ps -a --pod
CONTAINER ID  IMAGE                             COMMAND               CREATED         STATUS             PORTS                   NAMES                POD ID        PODNAME
73d129c7edbb  docker.io/library/nginx:latest    nginx -g daemon o...  52 seconds ago  Up 52 seconds ago  0.0.0.0:32597->80/tcp   recursing_albattani  f59faabb7cd3  nginx
e4a44f543a70  k8s.gcr.io/pause:3.2                                    52 seconds ago  Up 52 seconds ago  0.0.0.0:32597->80/tcp   f59faabb7cd3-infra   f59faabb7cd3  nginx
a700757aca10  docker.io/library/alpine:latest   top                   2 hours ago     Up 2 hours ago                             crazy_shtern         e94689355ac7  bold_banach
a4fa2fc0619d  k8s.gcr.io/pause:3.2                                    3 hours ago     Up 2 hours ago                             e94689355ac7-infra   e94689355ac7  bold_banach
3757a2216835  docker.io/library/mariadb:latest  mysqld                4 hours ago     Up 4 hours ago     0.0.0.0:3306->3306/tcp  flamboyant_ellis

(Can shorten the flags: instead of using -a --pod, coudl do -ap)
# podman ps -ap
CONTAINER ID  IMAGE                             COMMAND               CREATED         STATUS             PORTS                   NAMES                POD ID        PODNAME
73d129c7edbb  docker.io/library/nginx:latest    nginx -g daemon o...  52 seconds ago  Up 52 seconds ago  0.0.0.0:32597->80/tcp   recursing_albattani  f59faabb7cd3  nginx
e4a44f543a70  k8s.gcr.io/pause:3.2                                    52 seconds ago  Up 52 seconds ago  0.0.0.0:32597->80/tcp   f59faabb7cd3-infra   f59faabb7cd3  nginx
a700757aca10  docker.io/library/alpine:latest   top                   2 hours ago     Up 2 hours ago                             crazy_shtern         e94689355ac7  bold_banach
a4fa2fc0619d  k8s.gcr.io/pause:3.2                                    3 hours ago     Up 2 hours ago                             e94689355ac7-infra   e94689355ac7  bold_banach
3757a2216835  docker.io/library/mariadb:latest  mysqld                4 hours ago     Up 4 hours ago     0.0.0.0:3306->3306/tcp  flamboyant_ellis
```

Adding a pod is pretty straightforward too:
```bash
# podman ps -a --pod
CONTAINER ID  IMAGE                             COMMAND               CREATED         STATUS             PORTS                   NAMES                POD ID        PODNAME
73d129c7edbb  docker.io/library/nginx:latest    nginx -g daemon o...  22 minutes ago  Up 22 minutes ago  0.0.0.0:32597->80/tcp   recursing_albattani  f59faabb7cd3  nginx
e4a44f543a70  k8s.gcr.io/pause:3.2                                    22 minutes ago  Up 22 minutes ago  0.0.0.0:32597->80/tcp   f59faabb7cd3-infra   f59faabb7cd3  nginx
a700757aca10  docker.io/library/alpine:latest   top                   3 hours ago     Up 3 hours ago                             crazy_shtern         e94689355ac7  bold_banach
a4fa2fc0619d  k8s.gcr.io/pause:3.2                                    3 hours ago     Up 3 hours ago                             e94689355ac7-infra   e94689355ac7  bold_banach
3757a2216835  docker.io/library/mariadb:latest  mysqld                4 hours ago     Up 4 hours ago     0.0.0.0:3306->3306/tcp  flamboyant_ellis

# podman pod ps
POD ID        NAME         STATUS   CREATED         # OF CONTAINERS  INFRA ID
f59faabb7cd3  nginx        Running  25 minutes ago  2                e4a44f543a70
e94689355ac7  bold_banach  Running  3 hours ago     2                a4fa2fc0619d

# podman run -dt --name add_container_to_pod --pod nginx ubuntu top
8b9b580120b835098c4f3c96990fa9d6d86edf193b4a4414ca97d9e0a7d4ec3f

# podman ps -a --pod
CONTAINER ID  IMAGE                             COMMAND               CREATED         STATUS             PORTS                   NAMES                 POD ID        PODNAME
8b9b580120b8  docker.io/library/ubuntu:latest   top                   4 seconds ago   Up 4 seconds ago   0.0.0.0:32597->80/tcp   add_container_to_pod  f59faabb7cd3  nginx
73d129c7edbb  docker.io/library/nginx:latest    nginx -g daemon o...  23 minutes ago  Up 23 minutes ago  0.0.0.0:32597->80/tcp   recursing_albattani   f59faabb7cd3  nginx
e4a44f543a70  k8s.gcr.io/pause:3.2                                    23 minutes ago  Up 23 minutes ago  0.0.0.0:32597->80/tcp   f59faabb7cd3-infra    f59faabb7cd3  nginx
a700757aca10  docker.io/library/alpine:latest   top                   3 hours ago     Up 3 hours ago                             crazy_shtern          e94689355ac7  bold_banach
a4fa2fc0619d  k8s.gcr.io/pause:3.2                                    3 hours ago     Up 3 hours ago                             e94689355ac7-infra    e94689355ac7  bold_banach
3757a2216835  docker.io/library/mariadb:latest  mysqld                4 hours ago     Up 4 hours ago     0.0.0.0:3306->3306/tcp  flamboyant_ellis

# podman pod list
POD ID        NAME         STATUS   CREATED         # OF CONTAINERS  INFRA ID
f59faabb7cd3  nginx        Running  26 minutes ago  3                e4a44f543a70
e94689355ac7  bold_banach  Running  3 hours ago     2                a4fa2fc0619d
```

The Baude article referenced an video that showed how to create a pod that had:
* A MariaDB container bound to the 127.0.0.1 address (meaning only containers in the same pod could access it)
* Adds an nginx container to the pod
* Installs the MariaDB-client package onto the alpine container
* Connects to the MariaDB container from the alpine container

The link in the article appears to be broken, but I think I [found an alternative](https://asciinema.org/a/245493). Here are the steps:
```bash
# podman pod list
POD ID        NAME         STATUS   CREATED      # OF CONTAINERS  INFRA ID

# podman ps -a
CONTAINER ID  IMAGE   COMMAND  CREATED  STATUS  PORTS   NAMES

# podman run -dt -e MYSQL_ROOT_PASSWORD=t00r --pod new:db docker.io/library/mariadb
060fc40f35653cb3dad75280572f1bec3469079ac5b4a18465cc1c7f2fe7f967

# podman pod ps
POD ID        NAME    STATUS   CREATED         # OF CONTAINERS  INFRA ID
2ec3f212979f  db      Running  20 seconds ago  2                7cc2bbe6f229

# podman ps -ap
CONTAINER ID  IMAGE                             COMMAND  CREATED         STATUS             PORTS   NAMES               POD ID        PODNAME
060fc40f3565  docker.io/library/mariadb:latest  mysqld   52 seconds ago  Up 51 seconds ago          nervous_dubinsky    2ec3f212979f  db
7cc2bbe6f229  k8s.gcr.io/pause:3.2                       52 seconds ago  Up 52 seconds ago          2ec3f212979f-infra  2ec3f212979f  db

# podman run -it --rm --pod db docker.io/library/alpine /bin/sh
    (drops onto the alpine container shell)

> apk add mariadb-client
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
(1/6) Installing mariadb-common (10.4.13-r0)
(2/6) Installing ncurses-terminfo-base (6.2_p20200523-r0)
(3/6) Installing ncurses-libs (6.2_p20200523-r0)
(4/6) Installing libgcc (9.3.0-r2)
(5/6) Installing libstdc++ (9.3.0-r2)
(6/6) Installing mariadb-client (10.4.13-r0)
Executing busybox-1.31.1-r16.trigger
OK: 39 MiB in 20 packages

> mysql -u root -P 3306 -h 127.0.0.1 -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.5.5-MariaDB-1:10.5.5+maria~focal mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.001 sec)

```

To stop pod:
> podman stop <POD_INFRA_ID>   <- MUST BE the infra id. Should stop all containers in the pod. Doesn't seem to be true.
> podman start <POD_INFRA_ID>



