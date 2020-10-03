## Pod Resource Definition Syntax
A Pod Resource Definition provides Kubernetes with the configuration settings it needs to instantiate an instance of a pod. These can be defined in either JSON or YAML format, or generated from defaults via Openshift's `oc new-app` command.

### POD Resource Defintion
Sample resource definition for a Pod within Openshift (TO DO: Can I use this with a local Podman implementation?):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wildfly
  labels:
    name: wildfly
spec:
  containers:
    - resources:
        limits :
          cpu: 0.5
        image: do276/todojee
        name: wildfly
        ports:
         - containerPort: 8080
           name: wildfly
        env:
         - name: MYSQL_ENV_MYSQL_DATABASE
           value: items
         - name: MYSQL_ENV_MYSQL_USER
           value: user1
         - name: MYSQL_ENV_MYSQL_PASSWORD
           value: mypa55
```

This sample Pod definition does the following:
* Declares a Kubernetes pod resource type _($.kind)_.
* Assigns a name to uniquely identify the pod when invoking commands _($.metadata.name)_.
* Creates a label with a key `name`. This can be used by other Kubernetes resources to find this resource _($.metadata.labels.name)_.
* Specifies the port via which the container can be contacted _($.spec.containers.resources.port.containerPort)_.  TO DO: Confirm notation for YAML when there are hyphen entries like '- resources'.
* Defines environment variables for the container _($.spec.containers.resources.env.*)_.

### Service Resource Defintion
An OpenShift Service allows containers in one pod to open network connections to containers in another pod. Providing a stable IP address (tied to the Service name) provides the following benefits:
1. Avoids the need for pods to constantly rediscover the IP addresses of other pods after each restart
1. Allows a variable number horizontally-scaled pods to all be accessible through a singular IP.
```json
{
	"kind": "Service",
	"apiVersion": "v1",
	"metadata": {
		"name": "quotedb"
	},
	"spec": {
		"ports": [{
			"port": 3306,
			"targetPort": 3306
		}],
		"selector": {
			"name": "mysqldb"
		}
	}
}
```

This sample Service definition does the following:
* 