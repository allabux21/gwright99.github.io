### Project Goals
By completing the project, I aim to fulfill the following goals.
    
1. Establish a project template that:<br><br>
    1. Has its dependencies isolated from other projects on the system.
    1. Comes with a pre-configured linting solution.
    1. Comes with a pre-configured testing solution.
    1. Comes with a pre-configured database solution.
    1. Comes with a pre-configured containerization solution (Dockerfile and secret management).
    1. Comes with a pre-configured Github Actions integration (for CI/CD and automatic deployment to a cloud-hosting service).
    1. Comes with the pre-developed capability to respond to browser-based RESTful transactions<br><br> 
1. Define the processes and supporting configuration scripts necessary to easily deploy the template into a newly-instantiated local project.

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
        <td>I want to use cloud computing to host my completed solution but do not want to develop in the cloud. <br><br>The entire solution must be capable of being run on a local consumer-grade machine.</td></tr>
    <tr><td>003</td>
        <td>Solution must be run direcly on Windows 10 (not a VM)</td>
        <td>My development machine runs Windows 10, with  32GB of memory and a mid-tier Intel Kaby Lake 7th gen i5-7200 CPU (dual core). <br><br>Given its <a href="http://www.laptoping.com/cpus/product/intel-core-i5-7200u/">mediocre benchmarks</a> and my expectation that future projects will be CPU-intensive, the solution cannot be run in a traditional resource-restricted VM. <br><br><i><b>Note:</b> To avoid the "Dude, just install Linux as your base OS!" conversation, I also game on this machine (which is far easier on Windows). Furthermore, I feel a Windows-based solution is a necessary approach given that many employers still equip their employees with Windows-based machines.</i> </td></tr>
    <tr><td>004</td>
        <td>Source control and CI/CD solutions must use same component</td>
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
        <td>Use a synchronous framework</td>
        <td>Python has gotten better at supporting asynchronous programming <i>(e.g. making async/await reserved keywords in Python3.7)</i>. However, I find asynchronous programming more difficult for two reasons:
            <ul>
                <li>It requires logic to be structured and written in a different manner than traditional synchronous programs.</li>
                <li>It requires a different application server implementation (ASGI-compliant vs. WSGI-compliant) </li>
            </ul>
            Asynchronous programming is clearly preferrable when designing applications that must be performant under extremely high load, and this is programming style that I should become more familiar with for my own professional career. With that said, I don't think the extra complexity makes sense for my initial delivery goals. For this reason, I've decided to stick with WSGI and make a note that I should have a follow-up project that converts this WSGI-based solution to an ASGI-based framework <i>(e.g. moving from a framework like Flask to Starlette or FastApi)</i>.
        </td>
    </tr>
    <tr>
        <td>008</td>
        <td>Find right level of database abstraction</td>
        <td>Most real-world applications are going to require some sort of stateful storage. As a result, the solution should come with an pre-built database integration component. <br><br>The major decision I will need to make is whether to use an ORM like SQLAlchemy which heavily abstracts my database interactions, or use a less fulsome solution that places more reliance on the developer to interact with the database directly.<br><br><b>NOTE:</b>This particular item caused several downstream complications, which I will write about more in later project documentation pages.</td>
    </tr>
    <tr>
        <td>009</td>
        <td>Use source control system that easily integrates with CI/CD solutions</td>
        <td>I'm going to make alot of mistakes in this project, so I cannot be afraid to modify working code lest I never manage to recreate a useable product. As such, I absolutely must have a source control system that can operate in my preferred local environment, but also integrates easily with a cloud source control solution, for backup and CI/CD purposes.</td>
    </tr>
</table>

### Design Decisions
After mulling my design constraints, design considerations, previous project experience, availability of documentation/guides, and future expected benefits to both my job duties and personal goals, I decided on the following solution stack:

<ul>
    <li>
        <b>Development OS</b> <span><h4>Ubuntu 20.04 WSL2 running on Windows 10</h4></span>
        This struck me as the only viable technology stack given my stated constraints and preferences. I didn't want to have to move away from Windows 10 as my OS, but I *really* didn't want to directly develop on it either.Thankfully, Microsoft has perfect solution: <a href="https://docs.microsoft.com/en-us/windows/wsl/wsl2-index">Windows Subsystem for Linux 2</a>.<br><br>The Microsoft documentation re: WSL2 capabilities is extensive and explains the Subsystem's capabilities far better than I could, so I wont try to repeat it here. What is most important to note is that the WSL2 provides access to a full Linux-development environment (accessible to Windows). This is important note only for avoiding Windows compilation errors, but also gives native access to a host of Linux support tools like SSH and Make.<br><br>I chose Ubuntu 20.04 as my base Linux image because I have some previous experience working with that flavour of Linux, and because it is the most recent long term support release, with <a href="https://ubuntu.com/blog/what-is-an-ubuntu-lts-release">regular updates until 2029</a>.
    </li>
    <br><li>
    <b>Python package management and isolation solution</b><span><h4>pip & venv</h4></span>
            <a href="https://docs.python.org/3/tutorial/venv.html#managing-packages-with-pip">Pip</a> and <a href="https://docs.python.org/3/library/venv.html">venv</a> aren't new or fancy, but they work so why change?.<br>I may eventually look at newer solutions like <a href="https://pypi.org/project/pipenv/">pipenv</a> or <a href="https://python-poetry.org/docs/">poetry</a>, but see no compelling reason to complicate my life when I already have acceptable solutions shipped with the standard library.
    </li>
    <br><li>
        <b>Local source control</b><span><h4>Git</h4></span>
        Git is extremely easy to set up in Linux and integrate with a Git-based cloud solution <i>(e.g. GitHub, GitLab)</i>. Any developer worth their salt should know the basics of Git, so this is a must-learn topic regardless of project outcome.
    </li>
    <br><li>
        <b>Cloud source control & CI/CD</b><span><h4>GitHub</h4></span>
        It's established, versatile, and now a member of the Microsoft product family. GitHub offers easy-to-use source control, extensive CI/CD capabilities via GitHub Actions, project documentation facilities, and project management capabilities. It's possible other services may provide better individual functionality, but GitHub provides a one-stop shop for many. I would rather learn one platform well, than be mediocre at several.
    </li>
    <br><li>
        <b>Cloud hosting provider</b><span><h4>AWS</h4></span>
        Given my evident preference for other Microsoft solutions, you'd be forgiven for thinking I'd pick Azure. Azure is probably worth looking at in the long-term, but I have more familiarity with the AWS platform and have a few Serverless tutorials that I would like to try soon that are all based on AWS infrastructure. Winner: AWS. 
    </li>
    <br><li>
        <b>Container solution</b><span><h4>Docker</h4></span>
        The only legitimate contender in my mind. Docker is well-documented and well-supported. It helps that the latest iteration of Docker For Windows has tight integration with WSL2. 
    </li>
    <br><li>
        <b>IDE</b><span><h4>VSCode</h4></span>
        It's a free, full-feared IDE with great official Microsoft language support plugins. Better yet, it is fully integrated with WSL2 and can be directly invoked from the venv CLI. Plus I like how it looks.
    </li>
    <br><li>
        <b>Testing framework</b><span><h4>Pytest</h4></span>
        I'm not super familiar with Python testing frameworks. Regardless, I know I need an automated testing solution in order to leverage CI/CD effectively, and Pytest works in a way that I find more natural than the Python standard library's unittest module. 
    </li>
    <br><li>
        <b>Database</b><span><h4>SQLite3</h4></span>
        I'll probably revist this at a later point when my Docker container management skills are better. From a getting started point-of-view, however, it's just too easy to use SQLite3 to consider using anything else. It is effortless to install, cross-platform, and exceedingly easy to manage (given that the whole database is just a single file). Sqlite3 is not as robust as other database solutions when it comes to handling concurrent connections (i.e. <a href="https://en.wikipedia.org/wiki/Comparison_of_relational_database_management_systems">it can't</a>), but this was not deemed to be a be a deal-breaker for the purposes of my initial activities.
    </li>
    <br><li>
        <b>ORM</b><span><h4>*-SQLAlchemy</h4></span>
        When one talks web frameworks and databases, ORMs always seem to eventually show up. To be honest, I really dislike ORMs; every time I try to learn SQLAlchemy, I find I get confused and eventually lose motivation to continue my project. I told myself this time would be different and I would finally hunker down and figure it out <i>(foreshadow alert: it didn't go so well this time either)</i>.<br>I'll be documenting my opinion on flask-sqlalchemy and SQLAlchemy more in later articles in the series (to reflect what I encountered while trying to implement my solution). For the purposes of this technology selection portion, I'll simply say that I started with flask-sqlalchemy and then started to immediately regress once I began to learn more.
    </li>
    <br><li>
        <b>Web framework</b><span><h4>Flask</h4></span>
        Surprise! Given that this series is about creating a Flask solution, this should shock no one. As per the <a href="https://www.jetbrains.com/lp/python-developers-survey-2019/">2019 Jetbrains Developer survey</a>, Flask is the #1 web framework - slightly ahead of Django and miles ahead of the rest of the competition. Given its market dominance and (theoretical) ease of initial deployment, Flask is the obvious selection.
    </li>
    
</ul>
