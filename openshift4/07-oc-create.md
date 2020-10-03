## Creating OpenShift Applications

The `oc new-app` command can create OpenShift pods in a few different ways:
1. Axisting docker image
1. A Dockerfile
1. Source code (via source-to-image)

Notes: 
* OpenShift 4.5 `oc new-app` produces Deployment resources instead of DeploymentConfig resources. Differences defined [here](https://docs.openshift.com/container-platform/4.5/applications/deployments/what-deployments-are.html#what-deployments-are).
    * Main difference appearss to be DeploymentConfig use of "ReplicationController", whereas Deployment uses "ReplicaSet" (with ReplicaSet being the successor of ReplicationController). As per the OpenShift docs: _"The difference between a ReplicaSet and a ReplicationController is that a ReplicaSet supports set-based selector requirements whereas a replication controller only supports equality-based selector requirements."_. Dunno what this means.
* Use `oc new-app -h` to see all options available through the command
* Can generate skeleton templates using `oc new-app -o json` or oc new-app -o yaml`. These skeletons can then be customized and invoked via `oc create -f <FILENAME>`.

### Create an Application From an Image
Official openshift documentation [here](https://docs.openshift.com/enterprise/3.1/dev_guide/new_app.html#specifying-an-image).

Images can come from:
* Image from the Docker Hub registry (default)
* Local Docker server
* Image from a remote repository
* Image Streams on the OpenShift Enterprise server


Example image creation methods: (TO DO: verify this is true - the docs are not entirely clear on remote repositories. Mashing RedHat content with DO180 p. 154)):
```bash
1) Create application from Docker Hub (default):
# oc new-app mysql

2) Create application using a local registry image:
# oc new-app --docker-image=myregistry:5000/myrepository/myimage --name=myapp --as-deployment-config

3) Create application using an image from a remote registry:
# oc new-app --docker-image=docker.io/remoteregistry/remoteimage

4) Create application from an existing image stream:
# oc new-app --image=https://github.com/openshift/ruby-hello-world --name=ruby-hello --as-deployment-config

```
Notes: 
* I don't like the implicit sourcing from 'Docker Hub' in #1. I do not intend to use it, and will always explicitly define the image path.
* As per the OC docs, `new-app` will try to figure out if the image passed to it is a Docker-type or ImageStream. Like my Docker Hub comment, I don't like implicit so I'll use the `--docker-image` and `--image` during the `oc new-app` invocation.

Example creation command with labels and environment variables:
```bash
# oc new-app docker.io/library/mysql:latest --as-deployment-config MYSQL_USER=user MYSQL_PASSWORD=pass MYSQL_DATABSE=testdb -l db=mysql
```
This command:
* Creates a container based on the `mysql` image hosted on docker.io
* Adds a label key `db` and sets the value to `mysql`
* Passes in several MYSQL environment variables


### Create an Application From a Template
NOTE: THE RED HAT DOCS DONT MAKE SENSE TO ME. REVIEW AFTER SEEING CONTENT IN TRAINING DOCS. https://docs.openshift.com/enterprise/3.1/dev_guide/new_app.html
`oc new-app` can create applications from templates in two different ways:
1. Stored template file
1. Local Template file

Example template invocations:
```bash
1) Create application from stored template
# oc create -f examples/sample-app/application-template-stibuild.json

2) Create application from 
```
