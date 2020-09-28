## Podman Command Basics

### Searching for images
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

In most cases, I want to use an official base image released by the maintaining organization. In such cases, I can use the `--filter=is-official` flag to only return 'official' images. Executing `podman search --filter=-is-official alpine` cuts the previous list of 50 results down to one:
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


OFFICIAL Images
