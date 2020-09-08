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
        <td>The solution being built is meant to serve as the nucleus of more advanced future applications. Despite the risk of vendor-lockin, the initial design must have the integration hooks necessary to allow easy deployment into the vendor's environment and/or make use of other vendor services like Identity Access Management.</td>
    </tr>
    <tr>
        <td>005</td>
        <td>Container solution must be Docker</td>
        <td>Docker is the de facto base containerization solution of the day, and is well-understood and documented. The solution will ignore newer Kubernetes-centric containerization solutions like CRI-O in order to minimize the risk of unresolvable implementation errors. </td>
    </tr>
    <tr>
        <td>006</td>
        <td>Solution must provide HTTP-accessible visual responses</td>
        <td>Any solution I build is meant for portfolio demonstration purposes and/or for potential future web-based businesses. Building in visual response capabilities (<i>i.e. webpage responses</i>) from the outset means my project can easily support both human-based portal-type and system-based data API-type usage models. </td>
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
        <td>Use tools that are specifically designed for theit task</td>
        <td>Do not waste time trying to make a tool fulfill a role it is not specifically designed to do. <br><br><b>Example: Using SublimeText as an IDE.</b> In past development efforts, I have modified and extended <a href="https://www.sublimetext.com/">Sublime Text</a> as a "lite" IDE. The product is an excellent text editor, but the initial setup required hours of reading and decisions re: which packages to install and how to configure them properly, with ongoing support obligations that were frequent enough to be noticeaable but infrequent enough for me to immediately remember the solution. This took up limited time & attention resources which I could have better spent on actually writing code.</td>
    </tr>
    
    <tr>
        <td>002</td>
        <td>Minimize necessary system component configuration</td>
        <td>My approach should favour using tools that require minimal effort to configure/integrate with other components. <br><br>I should not be wasting my own limited time and attention of setting up my infrastructure if I have the option of leveraging the development & support efforts of a large software company. <br> For example: I like the experience of using SublimeText as a text editor, and even managed to transform it into a small resources of a </td>
    </tr>
    
    <tr>
        <td>001</td>
        <td>Minimize pip package installation problems</td>
        <td>Although Python runs on both Windows and Linux ...</td>
    </tr>
</table>


### Design Decisions
Use linux - Python scientific libs, more comfortable with Linux CLI, makefile
AWS
GitHub
WSL2
VSCode
