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
        <td>Solution must use Python3.7+</td>
        <td>Probably a little self-evident given that this is a series about <i>Flask</i>, but still worth calling out.<br><br>The solution must use Python3.7 or higher to take advantage of quality-of-life <a href="https://docs.python.org/3/whatsnew/">improvements</a> (f-string, native async/await, etc.) </td></tr>
    <tr><td>002</td>
        <td>Solution must be local machine centric</td>
        <td>I want to make use of cloud computing to host my completed solution but do not want to develop on the cloud platform itself. <br><br>The entire solution must be capable of being run on local consumer-grade machine.</td></tr>
    <tr><td>003</td>
        <td>Solution must be run direcly on Windows 10, not a VM</td>
        <td>My development laptop runs Windows 10. It has 32GB of memory and a mid-tier Intel Kaby Lake 7th gen i5-7200 CPU (dual core). <br><br>Given its <a href="http://www.laptoping.com/cpus/product/intel-core-i5-7200u/">mediocre benchmarks</a> and my expectation that future projects will be CPU-intensive, the solution cannot be run in a traditional resource-restricted VM. <br><br><i><b>Note: To avoid the "Dude, just install Linux as your base OS!" conversation, I'll note right now that I like to play games on this machine (which Windows makes far easier) and I feel that developing for a Windows-first OS is a good practice given that a majority of employers in the marketplace still equip their employees with Windows-based machines. So the Windows requirement sticks.</b></i> </td></tr>
    <tr><td>004</td>
        <td>Source control and CI/CD solutions must be served by same component</td>
        <td>Modern-day applications already require a plethora of technology components. <br><br>I do not want to add extra complexity by having to integrate two separate services (<i>e.g. GitLab and Jenkins</i>) if a single service alternative exists (<i>e.g. GitHub</i>).</td>
    </tr>
    <tr>
        <td>005</td>
        <td>Cloud-hosting provider must offer extensive service catalog, including managed Kubernetes hosting</td>
        <td>The solution being built is meant to serve as the nucleus of more advanced future applications. Despite the risk of vendor-lockin, the initial design must have the integration hooks necessary to allow easy deployment into the vendor's environment and/or make use of other vendor services like Identity Access Management.</td>
    </tr>
    <tr>
        <td>006</td>
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
        <td>Use tools that are specifically designed for their task</td>
        <td>Do not waste time trying to make a tool fulfill a role it is not specifically designed to do. <br><br><b>Example: Using SublimeText as an IDE.</b> In past development efforts, I have modified and extended <a href="https://www.sublimetext.com/">Sublime Text</a> as a "lite" IDE. The product is an excellent text editor, but the initial setup required hours of reading and decisions re: which packages to install and how to configure them properly, with ongoing support obligations that were frequent enough to be noticeaable but infrequent enough for me to immediately remember the solution. This took up limited time & attention resources which I could have better spent on actually writing code.</td>
    </tr>
    <tr>
        <td>002</td>
        <td>Minimize necessary system component configuration</td>
        <td>Favour using tools that require minimal effort to configure/integrate with other components. <br><br>I should not be expending effort to integrate my infrastructure if I can leverage the development & support efforts of a large software company instead. </td>
    </tr>
    <tr>
        <td>003</td>
        <td>Minimize Python package installation dependencies</td>
        <td>Favour a Python environment with the lowest likelihood of pip package installation failures.<br><br>I like being able to type `pip install` and having that package's functionality availble just seconds later. What drives me <u>crazy</u> is trying to install a package only to receive the dreaded <i>"Microsoft Visual C++ Build Tools" </i> error, because this means I'll spend the next twelve hours: 
            <ol>
                <li>Searching Google for an answer that actually works.</li>
                <li>Searching the endless Microsoft website for the exact required component.</li>
                <li>Watching my ISP data cap get chewed up as I download components from Microsoft that turn out to not be what I need</li>
                <li>Swearing at my computer and myself for not having learned from the last time I had to go through this ordeal.</li>
            </ol>
            This time <b><u>I'll do it RIGHT</u></b>
        </td>
    </tr>
    <tr>
        <td>004</td>
        <td>Use the standard Python release</td>
        <td>There are multiple ways around the Python package installation problem I've mentioned in in #003. Two methods I've used are:
            <ul>
                <li>Use <a href="https://www.lfd.uci.edu/~gohlke/pythonlibs/">pre-compiled libraries</a> from Christoph Gohlke</li>
                <li>Use the <a href="https://docs.anaconda.com/anaconda/install/">Anaconda distribution</a> of Python</li>
            </ul>
        While both of these solutions are viable and successfully solved my problems in the past, I dont like having to rely on them:
            <ul>
                <li>Using Gohlke's libraries makes me dependent on the generosity of a single contributor (which he even calls out as being unofficial) as well as requiring trust in the security of his web infrastructure. <br>(<i>I am <b><u>NOT</u></b> saying that Christoph Gohlke is untrustworthy. Rather, harkening back to my earlier Design Considerations, I philosophically don't like the increased complexity of Python library acquisition as well as the addition of an extra attack vector to any resulting application's attack surface).</i></li>
                <li>Using the Anaconda distribution ties me more tightly to the decisions and fortunes of Anaconda, Inc. Furthermore, it requires the use of the <a href="https://en.wikipedia.org/wiki/Conda_(package_manager)">conda package management system </a> and <a href="https://conda-forge.org/">conda-forge</a> for additional packages not directly available from conda. This solution is certainly viable but my personal preference is to avoid having to learn/remember the idiosyncracies of the Anaconda stack if I can fulfill the result just as easily with the standard distribution.</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>005</td>
        <td>Use an unopinionated framework</td>
        <td>This one is a matter of personal preference. Using an opinionated web framework like Django would likely make it easier to get a fully featured-website up and running. However, one of the reasons I'm engaged in this exercise is to learn, and that's harder to do if there is only "one" correct way to do things. As a result, (and acknowledging the days of frustration this decision is likely to cause) the solution should use a framework that leaves the majority of design decisions up to me.</td>
    </tr>
    <tr>
        <td>006</td>
        <td>Use a well-documented framework</td>
        <td>Given Design Consideration #005, I'm going to need to make many decisions and it's likely I wont immediately know what the correct decision is. As a result, I should choose a framework that is well-established and has a large community. This will increase the chance that someone has encountered the same problem I'm having, and that a solution already exists on some blog or Stack Overflow discussion page. For this reason, I should favour an older framework like Flask over a newer framework like FastApi.</td>
    </tr>
    <tr>
        <td>007</td>
        <td>Use synchronous framework</td>
        <td>Python has gotten better at supporting asynchronous programming <i>(e.g. making async/await reserved keywords in Python3.7)</i>. However, I find asynchronous programming more difficult for two reasons:
            <ul>
                <li>It requires logic to be structured and written in a different manner than traditional synchronous programs.</li>
                <li>It requires a different application server implementation (ASGI-compliant vs. WSGI-compliant) </li>
            </ul>
            Asynchronous programming is clearly preferrable when designing applications that must be performant under extremely high load, and this is programming style that I should become more familiar with for my own professional career. With that said, I don't think the extra complexity makes sense for my initial delivery goals. For this reason, I've decided to stick with WSGI and make a note that I should have a follow-up project that converts this WSGI-based solution to an ASGI-based framework <i>(e.g. moving from a framework like Flask to Starlette or FastApi)</i>.
        </td>
    </tr>
        
</table>

### Design Decisions
Use linux - Python scientific libs, more comfortable with Linux CLI, makefile
AWS
GitHub
WSL2
VSCode
