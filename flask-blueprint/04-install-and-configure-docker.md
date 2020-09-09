## Install and Configure Docker Desktop
In order to properly support the project, our Docker implementation (Docker Desktop) needs to do the following: 

1. Integrate with the WSL2 development environment
1. Integrate with the GitHub Package Registry

### Integrate Docker Desktop with WSL2
Follow [these steps](https://docs.docker.com/docker-for-windows/wsl/) to install the Docker Desktop application and tie it to you WSL2 instance. The steps are simple and 
straight-forward so there's not much to comment on. For those considering multi-stage Docker builds, you may wish to enable 
[DOCKER_BUILDKIT](https://www.docker.com/blog/docker-desktop-wsl-2-best-practices/) in order to use multiple CPU cores to run different build stages concurrently. I followed Docker's advice and added the entry to my `~/.profile`.<br>

Once this work is complete, docker is available via the WSL2 CLI through the `docker` command.

### Integrate Docker with GitHub
With Docker now available on the WSL2 CLI, it's time to integrate it with the GitHub Container Registry (**NOTE**: _Docker Container Registry was very recently announced (Sept 2020), and is set to supercede the existing GitHub Packages Docker registry. This has two implications readers should note: (1) As of Sept 9, 2020 the Martin Heinz [reference articles](./01-why-create-this-project.md) are not yet updated to reflect use of the new registry, (2) As oer the GitHub Container Registry [information page](https://docs.github.com/en/packages/getting-started-with-github-container-registry/about-github-container-registry), the Container Registry is still in public beta, subject to change, and does not yet have a definite storage & costing model. Beware.)

#### Steps
1. Open the [Personal Access Token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) screen in your GitHub console.

1. Create a PAT for the GitHub Container Registry, ensuring to include [these scopes](https://docs.github.com/en/packages/getting-started-with-github-container-registry/migrating-to-github-container-registry-for-docker-images#authenticating-with-the-container-registry).

1. Copy the newly-generated token to your local WSL2 environment:
    1. GitHub suggests storing the PAT as an environment variable named `CR_PAT`.
    
    1. Unfortunately, I already had a `CR_PAT` from a previous integration between Docker and the GitHub Packages Docker Registry, and immediately started struggling to remember which token granted access to what. To fix this, I created two new entries in `~/.profile` with clear (i.e. long) names: 
      * `GH_Container_Registry_PAT`
      * `GH_Packages_Docker_Registry_PAT`
      
    1. Add the GitHub-generated values to the newly-created environment variable(s).
    
    1. Type `source .profile` to load the newly-created environment variable(s).
    
    1. Connect your WSL2 docker to the GitHub Container Registry by typing `echo $GH_Container_Registry_PAT | docker login ghcr.io -u USERNAME_HERE --password-stdin`.
  <br> If all goes well, GHCR will return a 'Login Successful' message and the integration is complete.
    

Previous: [Project Goals and Design Considerations](./03-set-up-WSL2.md)<br>
Next: 
