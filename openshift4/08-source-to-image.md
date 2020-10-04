## Source-to-Image (S2I)
Source-to-Image is an OpenShift tool used to create containers based on application source code. The tool collects the source code from a Git repository, injects the source code into a base container that is configured for its programming language, and produces a new container image that runs the assembled application.

<img src="./img/source-to-image.png"><br>

S2I is the suggested primary strategy for building applicationns in OpenShift because:
* It removes the need for developers to understand Dockerfiles and many Linux system commands.
* Allows for rebuilding applications consistently if a base image needs a patch due to a security issue.
* Assembly process can perform a large number of complex operations without requiring a layer for each step.
* Encourages reuse of base images and scripts across multiple types of applications.

### What's an Image Stream?
The [OCP documentation] defines an Image Stream as:

_'an abstraction for referencing Docker images within the OpenShift Container Platform. The image stream and its tags allow you to see what images are available and ensure you are using the specific image you need even if the image in the repository changes. <br>Image streams do not contain actual image data, but present a single virtual view of related images, similar to an image repository.<br>You can configure Builds and Deployments to watch an image stream for notifications when new images are added and react by performing a Build or Deployment, respectively.<br>For example, if a Deployment is using a certain image and a new version of that image is created, a Deployment could be automatically performed to pick up the new version of that image.<br>However, if the image stream tag used by the Deployment or Build is not updated, then even if the Docker image in the Docker registry is updated, the Build or Deployment will continue using the previous (presumably known good) image.'_

Essentially, I'm thinking about it like this: You've build an application on top of an OS image that is discovered to have a zero-day vulnerability. There's nothing wrong with your own code, it's the OS that needs patching. If you build the application using an Image Stream as the reference to your base OS, OCP will automatically update all your pods when the referenced (patched) image is deployed to your image repository. If you pinned to a specific container image instead of using an Image Stream, you would need to go through a new manual repackaging & repromotion exercise to get a fixed image back into Production. 

My example focuses on a security patch, but this holds true equally well for the builder images that are used to create images via the OpenShift Source-to-Image technique. To determine the available image streams in your cluster use `oc get is -n openshift`
```bash
# oc get is -n openshift
NAME            IMAGE REPOSITORY                        TAGS
cli             ...svc:5000/openshift/cli               latest
dotnet          ...svc:5000/openshift/dotnet            2.0,2.1,latest
dotnet-runtime  ...svc:5000/openshift/dotnet-runtime    2.0,2.1,latest
httpd           ...svc:5000/openshift/httpd             2.4,latest
jenkins         ...svc:5000/openshift/jenkins           1,2
...
```

S2i can build images based on:
* Remote code repositories
* Local code repositories (must be working directory)
* Dockerfile

To create an application via S2i, you must make some minor modifications to the `oc new-app` command:
```bash
# oc new-app --as-deployment-config php~http://<REMOTE.GIT.URL>/<REMOTE.REPOSITORY --name=myapp
# oc new-app --as-deployment-config -i php http://<REMOTE.GIT.URL>/<REMOTE.REPOSITORY --name=myapp
# oc new-app --as-deployment-config .  --name=myapp
```
The language-specific builder image can be defined in two ways:
* To the left of the tilde `~` that connects the language to the source code repository (e.g. `php~http://<REMOTE.GIT.URL>/....`
* Specified with the `-i` flag (e.g. `... -i php http://<REMOTE.GIT.URL>/....

If you do not specify the language, Openshift will try to identify the correct image stream for building the application (based on the presence of specific language-specific files in the folder). Given how easy it is to identify the correct image stream, not explicitly identifying the image stream seems very lazy.

You can further specify the location of your source code by also identifying the `context-dir` or `branch`. Examples:
```bash
Specific folder in the repository:
# oc new-app --as-deployment-config https://github.com/openshift/sti-ruby.git --context-dir=2.0/test/puma-test-app

Specific branch in the repository:
# oc new-app --as-deployment-config https://github.com/openshift/ruby-hello-world.git#beta4
```

<img src="./img/s2i-context-dir.png">

This is great, but how does it actually work? Let's start by generating a new Image Stream-based project and output it to a json file so we can examine it (Note: My OCP lab doesn't allow me to cut and paste content from its console to this blog, so I wont be able to paste everything).

Outputting the results of the new-app command will create a List object that contains four other objects :
* ImageStream
* BuildConfig
* DeploymentConfig
* Service

```bash
# oc -o json new-app --as-deployment-config php~http://services.lab.example.com/app --name=myapp > s2i.json
{
  "kind": "List",
  "apiVersion": "v1",
  "metadata": {},
  "items": [
    {
      "kind": "ImageStream",
      "apiVersion": "image.openshift.io/v1",
      "metadata": {
        "name": "myapp",
        "creationTimestamp": null,
        "labels": {
          "app": "myapp",
          "app.kubernetes.io/component": "myapp",
          "app.kubernetes.io/instance": "myapp"
        },
        "annotations": {
          "openshift.io/generated-by": "OperShiftNewApp"
        }
      },
      "spec": {
        "lookupPolicy": {
          "local": false
        }
      },
      "status": {
        "dockerImageRepository": ""
      }
    },
    {
      "kind": "BuildConfig",
      "apiVersion": "build.openshift.io/v1",
      "metadata": {
        "name": "myapp",
        "creationTimestamp": null,
        "labels": {
          "app": "myapp",
          "app.kubernetes.io/component": "myapp",
          "app.kubernetes.io/instance": "myapp"
        },
        "annotations": {
          "openshift.io/generated-by": "OpenShiftNewApp"
        }
      },
      "spec": {
        "triggers": [
          {
            "type": "GitHub",
            "github": {
              "secret": <SECRET_HERE>
            }
          },
          {
            "type": "Generic",
            "generic": {
              "secret": <SECRET_HERE>
            }
          },
          {
            "type": "ConfigChange",
          },
          {
            "type": "ImageChange",
            "imageChange": {}
          }
        ],
        "source": {
          "type": "Git",
          "git": {
            "uri": "http://services.lab.example.com/app"
          }
        },
        "strategy": {
          "type": "Source",
          "sourceStrategy": {
            "from": {
              "kind": "ImageStreamTag",
              "namespace": "openshift",
              "name": "php:7.3"
            }
          }
        },
        "output": {
          "to": {
            "kind": "ImageStreamTag",
            "name": "myapp:latest"
          }
        },
        "resources": {},
        "postCommit": {},
        "nodeSeletor": null
      },
      "status": {
        "lastVersion": 0
      }
    },
    {
      "kind": "DeploymentConfig"
      ...
    },
    {
      "kind": "Service"
      ...
    }
  ]
}

  
```
