## Multi-Container Applications
Lets suppose our application requires a few different components to fulfill its service (e.g. NGINX and MYSQL). How can we create a solution which does not require manually juggling many different container instances?

### Networking Recap
Podman uses a CNI to create an SDN between all containers on the host. Any time a new container is started, it will be assigned a unique IP. Furthermore, any container exposed on an SDN is available to every other container on that SDN. Containers are not accessible to the outer world unless they are explicitly told to do so. IP discovery becomes a challenge, however, as restarted containers may have a different IP address.  

Kubernetes solves this problem by using services. Services provide a static DNS to be used to communicate with a group of containers that provide that service. A service consumer need not know the specific IPs of the containers that will fulfill that requesst - Kubernetes handles this dynamic mapping in the background. Like a duck swiming on the lake, a service provide a calm, consistent exterior to the frantic churning of feet under the surface.

Services managed by Kubernetes are communicates to all containers by virtue of several environment variables: 
* <SERVICE_NAME>_SERVICE_HOST: Represents the IP address enabled by a service to access a pod.
* <SERVICE_NAME>_SERVICE_PORT: Represents the port where the server port is listed.
* <SERVICE_NAME>_PORT: Represents the address, port, and protocol provided by the service for external access.
* <SERVICE_NAME>_PORT_<PORT_NUMBER>_<PROTOCOL>: Defines an alias for the <SERVICE_NAME>_PORT.
* <SERVICE_NAME>_PORT_<PORT_NUMBER>_<PROTOCOL>_PROTO: Identifies the protocol type (TCP or UDP).
* <SERVICE_NAME>_PORT_<PORT_NUMBER>_<PROTOCOL>_PORT: Defines an alias for <SERVICE_NAME>_SERVICE_PORT.
* <SERVICE_NAME>_PORT_<PORT_NUMBER>_<PROTOCOL>_ADDR: Defines an alias for <SERVICE_NAME>_SERVICE_HOST

For example, the following Service definition would translate into the following environment variables.
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    name: mysql
    name: mysql
spec:
  ports:
    - protocol: TCP
    - port: 3306
 selector:
   name: mysql
```

```bash
MYSQL_SERVICE_HOST=10.0.0.11
MYSQL_SERVICE_PORT=3306
MYSQL_PORT=tcp://10.0.0.11:3306
MYSQL_PORT_3306_TCP=tcp://10.0.0.11:3306
MYSQL_PORT_3306_TCP_PROTO=tcp
MYSQL_PORT_3306_TCP_PORT=3306
MYSQL_PORT_3306_TCP_ADDR=10.0.0.11
```
NOTE: If the Protocol component of an environment variable is undefined, Kubernetes will default to TCP. It also spawns other environment variables based on the protocol (e.g. `MYSQL_PORT=tcp://10.0.0.11:3306` causes `MYSQL_PORT_3306_TCP`, `MYSQL_PORT_3306_TCP_PROTO`, `MYSQL_PORT_3306_TCP_PORT`, and `MYSQL_PORT_3306_TCP_ADDR` to be spawned too).

### Example Multi-Container Application - To Do App
* Presentation layer built as AngularJS SPA
* Business tier layer built as NodeJS API backend
* Persistence tier layer built as a MySQL database server

Exercise says that we are using a custom MySQL 5.7 image that is configure to automatically run any scripts in the `/var/lib/mysql/init` directory (which load the schema and sample data when the container starts). I wanted to learn more about the image so I started trying to find it.

First, I looked at the Dockerfile which I had access to. Almost everything was commented except for the reference to `rhscl/mysql-57-rhel7` and adding `ADD root /`.
```bash
# cat /home/student/DO180/labs/multicontainer-design/images/mysql/Dockerfile
```
The Dockerfile:
```docker
FROM rhscl/mysql-57-rhel7
# MySQL image for DO180
#
# Volume:
# * /var/lib/mysql/data - Datastore for MySQL
#   /var/lib/mysql/init - Folder to load *.sql scripts
# Environment
# * $MYSQL_USER - Database user name
# * $MYSQL_PASSWORD - User's password
# * $MYSQL_DATABASE - Name of the database to create
# * $MYSQL_ROOT_PASSWORD (Optional) - Password for the 'root' MySQL account

ADD root /
```

Next, I looked for the image in the local repository with `sudo podman images`, but nothing was present there. I concluded the image had to be remote, but in which repository?
Since I was using `sudo` that meant the repository details were in `/etc/containers/registries.conf`.

Checking the `registries.conf` file, I found this entry:
```bash
[registries.search]
registries = ['registry.access.redhat.com']
```

So now I could conclude that the image was remote and could be retrieved from `registry.access.redhat.com/do180/mysql-57-rhel7`. Podman can only be used to inspect local images but Skopeo can help! Unfortunately, there wasn't much useful returned. Maybe I could do something else to figure out how this was special? I don't know.
```bash
# skopeo inspect docker://registry.access.redhat.com/rhscl/mysql-57-rhel7
```
<img src="/img/skopeo-mysql.png">
