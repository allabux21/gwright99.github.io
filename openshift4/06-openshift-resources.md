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

### Route Resource
```json
{
	"apiVersion": "v1",
	"kind": "Route",
	"metadata": {
		"name": "quoteapp"
	},
	"spec": {
		"host": "quoteapp.apps.example.com",
		"to": {
			"kind": "Service",
			"name": "<SERVICE_NAME_HERE>"
		}
	}
}
```
Notes:
* `host` is the FQDN associated with the route. The DNS resolution of this value must point to the IP address of the OpenShift router.
* `oc new-app` does not create a route when building a pod (because it does not know if the pod is meant to be accessed from outside the OpenShift instance or not).
* Naming:
    * By default, routes use the following naming convention: `route-name-project-name.default-domain`
    * A custom name can be provided by using the name flag, e.g. `oc expose service <SERVICE-NAME> --name <CUSTOM ROUTE NAME>`

### Managing Resources at the CLI (154)
Use the `oc get` command to retrieve information about resources in the OpenShift cluster. Resource types are:
* Image Stream (is)
* DeploymentConfig (dc)
* ReplicationConfig (rc)
* Service (svc)
* Pod (po)

```bash
# oc get all
# oc describe <RESOURCE_TYPE> <RESOURCE_NAME>
# oc get <RESOURCE_TYPE> <RESOURCE_NAME> -o yaml
# oc get <RESOURCE_TYPE> <RESOURCE_NAME> -o json
# oc get svc, dc -l app=nexus

# oc create
# oc edit
# oc delete <RESOURCE_TYPE> <RESOURCE_NAME>
# oc exec <CONTAINER_ID> options command
```

### Labelling Resources
Labels are used to group resources by concepts like application and environment. Labels can be applied to Resources in their `metadata` section (applied as key-value pairs). These labels can also be used to limit the resources returned by the `oc get` command.

Example:
```yaml
apiVersion: v1
kind: Service
metadata:
...contents omitted...
 labels:
 app: nexus
 template: nexus-persistent-template
 name: nexus
...contents omitted...
```

Two notes about templates:
1. Any word can be used as a label, but both the `app` and `template` keys are traditionally used for specific purposes. `App` indicates the application related
to the resource. `Template` labels all resources generated by the same template with the template's name.
1. A template resource has TWO areas for labels:
    1.1. `$.labels` - labels defined here do not apply to the template itself, but only to resources generated from this template.
    1.1. `$.metadata.labels` - labels applied to the template resource itself.

```yaml
apiVersion: template.openshift.io/v1
kind: Template
labels:
 app: nexus
 template: nexus-persistent-template
metadata:
...contents omitted...
 labels:
 maintainer: redhat
 name: nexus-persistent
...contents omitted...
objects:
- apiVersion: v1
 kind: Service
 metadata:
 name: nexus
 labels:
 version: 1
...contents omitted...
```
