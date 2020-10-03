## Pod Resource Definition Syntax
A Pod Resource Definition provides Kubernetes with the configuration settings it needs to instantiate an instance of a pod. These can be defined in either JSON or YAML format, or generated from defaults via Openshift's `oc new-app` command.

### POD Resource
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

### Service Resource
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
* Declares a Kubernetes Service resource type _($.kind)_.
* Assigns a name to uniquely identify the Service _($.metadata.name)_.
* Defines the mapping exposed by the service _($.spec.ports.port)_ to the port on that pod that the service will forward traffic to _($.spec.ports.targetPort)_.
* selector is how the service finds pods to forward packets to. The target pods need to have matching labels in their metadata attributes. If the service finds multiple pods with matching labels, it load balances network connections between them _($.spec.selector)_.

TO DO: In a multi-container pod, only one container can grab a specific port exposed by the pod? Sounds like this is not the case due to the SELECTOR being able to handle multiple pod labels?

Each Service defined within an Openshift project is identified by two environment variables injected into each pod inside the same project:
* _${SERVICENAME}_SERVICE_HOST_
* _${SERVICENAME}_SERVICE_PORT_

You can also find the Service via OpenShift's internal DNS server (visible only to pods), using the following naming convention:
* _${SERVICENAME}.${PROJECTNAME}.svc.cluster.local_
