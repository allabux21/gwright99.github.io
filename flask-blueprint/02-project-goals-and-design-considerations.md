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
        <td>I want to make use of cloud computing to host my completed solution but do not want to develop on the cloud platform itself. The solution must be capable of being run on local development machine.</td></tr>
    <tr><td>002</td>
        <td>Solution must be run direcly on Windows 10, not a VM</td>
        <td>My development laptop runs Windows 10. It has 32GB of memory and a mid-tier Intel Kaby Lake 7th gen i5-7200 CPU (dual core). Given its [mediocre benchmarks](www.laptoping.com/cpus/product/intel-core-i5-7200u/) and my expectation that future projects will be CPU-intensive, the solution cannot be run in a traditional resource-restricted VM. </td></tr>
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
