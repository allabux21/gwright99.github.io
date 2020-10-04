## Creating Our First Openshift Application
Lets build our first OC app!

```bash
# oc login -u ${USERNAME} -p ${PASSWORD} ${OCP_API}

# oc new-project mysql-openshift-project
# oc new-app --as-deployment-config --docker-image=registry.access.redhat.com/rhscl/mysql-57-rhel7:latest --name=mysql-openshift-app -e MYSQL_USER=user1 -e MYSQL_PASSWORD=mypa55 -e MYSQL_DATABASE=testdb -e MYSQL_ROOT_PASSWORD=r00tpas55

# oc status
# oc get pods -o=wide
<This returns podnames in the project, status, restarts, IP, Node, etc. Let's assume the pod is called 'mysql-openshift-app-1-abcde`>

# oc describe pod mysql-openshift-app-1-abcde

# oc get svc
<This returns services in the project. Lets assume the returned Service is called 'mysql-openshift-app'
# oc describe svc mysql-openshift-app
# oc expose svc mysql-openshift-app

# oc describe dc mysql-openshift-app

# oc port-forward mysql-openshift-app-1-abcde 3306:3306
# mysql -uuser1 -pmypa55 -protocol tcp -h localhost

> show databases;
> exit
```

These steps do the following: 
1. Pulls an MYSQL image from the Red Hat registry.
1. Instantiaties an instance of a MYSQL database.
1. Makes the service availble via an FQDN.
1. Establishes a connection between port 3306 on the local machine and port 3306 on the instantiated pod.
1. Connects to the container MYSQL database via the `mysql` utility on the local machine.
