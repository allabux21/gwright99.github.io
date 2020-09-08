### Project Goals
By working on this project, I hope to accomplish the following:
1. Settle on a development environment that:
    1. Removes the aggravation of Python development in a Windows environment.
    1. Allows me to work locally but can migrate to the cloud effortlessly.
    &nbsp;
    
2. Establish a project template that:
    1. Has its dependencies isolated from other projects on the system.
    1. Comes with a pre-configured linting solution.
    1. Comes with a pre-configured testing solution.
    1. Comes with a pre-configured database solution.
    1. Comes with a pre-configured containerization solution (Dockerfile and secret management).
    1. Comes with a pre-configured Github Actions integration (for CI/CD and automatic deployment to a cloud-hosting service).
    1. Comes with the pre-developed capability to respond to browser-based RESTful transactions
  
  
3. Define the processes and supporting configuration scripts necessary to easily deploy the template into a newly-instantiated local project.

### Design Constraints
<table>
    <tr>
        <th>#</th>
        <th>Constraint</th>
        <th>Description</th></tr>
    <tr><td>001</td>
        <td>Solution must be local machine centric</td>
        <td>I want to make use of cloud computing to host my completed solution but do not want to develop on the cloud platform itself. <br><br>The entire solution must be capable of being run on local consumer-grade machine.</td></tr>
    <tr><td>002</td>
        <td>Solution must be run direcly on Windows 10, not a VM</td>
        <td>My development laptop runs Windows 10. It has 32GB of memory and a mid-tier Intel Kaby Lake 7th gen i5-7200 CPU (dual core). <br><br>Given its <a href="http://www.laptoping.com/cpus/product/intel-core-i5-7200u/">mediocre benchmarks</a> and my expectation that future projects will be CPU-intensive, the solution cannot be run in a traditional resource-restricted VM. </td></tr>
    <tr><td>003</td>
        <td>Source control and CI/CD solutions must be served by same component</td>
        <td>Modern-day applications already require a plethora of technology components. <br><br>I do not want to add extra complexity by having to integrate two separate services (<i>e.g. GitLab and Jenkins</i>) if a single service alternative exists (<i>e.g. GitHub</i>).</td>
    </tr>
    <tr>
        <td>004</td>
        <td>Cloud-hosting provider must offer extensive service catalog, including managed Kubernetes hosting</td>
        <td>The solution being built is meant to serve as the nucleus of more advanced future applications. Despite the risk of vendor-lockin, the initial design must have the integration hooks necessary to allow easy deployment into the vendor's environment and / or make use of other vendor services (<i>like Identity Access Management</i>).</td>
    </tr>
    <tr>
        <td>005</td>
        <td>Container solution must be Docker</td>
        <td>Docker is the de facto base containerization solution of the day, and is well-understood and documented. The solution will ignore newer Kubernetes-centric containerization solutions like CRI-O in order to minimize the risk of unresolvable implementation errors. </td>
    </tr>
    
</table>

### Design Considerations
<table>
    <tr>
        <th>#</th>
        <th>Consideration</th>
        <th>Description</th></tr>
    <tr>
        <td>001</td>
        <td>Minimize pip package installation problems</td>
        <td>Although Python runs on both Windows and Linux ...</td>
    </tr>
</table>


### Design Decisions
