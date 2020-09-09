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
        <td>Modern-day applications already require a plethora of technology components. I do not want to add extra complexity by having to integrate two separate services (<i>e.g. GitLab and Jenkins</i>) if a single service alternative exists (<i>e.g. GitHub</i>).</td>
    </tr>
    <tr>
        <td>005</td>
        <td>Cloud provider must have extensive service catalog</td>
        <td>The solution being built is meant to serve as the nucleus of more advanced future applications. Despite the risk of vendor-lockin, the initial design must have the integration hooks necessary to allow easy deployment into the vendor's environment and/or make use of other vendor services like Identity Access Management.</td>
    </tr>
    <tr>
        <td>006</td>
        <td>Container solution must be Docker</td>
        <td>Docker is the de facto base containerization solution of the day. The solution will ignore newer Kubernetes-centric containerization solutions like CRI-O in order to minimize the risk of unresolvable implementation errors. </td>
    </tr>
    <tr>
        <td>006</td>
        <td>Solution must provide RESTful HTML responses</td>
        <td>Any solution I build is meant for portfolio demonstration purposes and/or future web-based businesses. Building HTML response capabilities from the outset means my project can easily support both human-based portal-type and system-based data API-type usage models. </td>
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
        <td>Do not waste time trying to make a tool fulfill a role it is not specifically designed to do. <br><br><b>Example: Using SublimeText as an IDE.</b><br> In the past, I have configured <a href="https://www.sublimetext.com/">Sublime Text</a> as a "lite" IDE. Although an excellent text editor, the product was not designed as a first-class IDE. Getting it to a mostly-tolerable state required extensive configuration, modification, and ongoing maintenance. This demanded a portion of my limited time & attention resources which I could have better spent on actually writing code.</td>
    </tr>
    <tr>
        <td>002</td>
        <td>Minimize necessary system component configuration</td>
        <td>Favour tools that require minimal effort to configure/integrate with other components. <br><br>I should not be expending effort to integrate my infrastructure if I can leverage the development & support efforts of a large software company instead. </td>
    </tr>
    <tr>
        <td>003</td>
        <td>Minimize Python package installation dependencies</td>
        <td>Favour a Python environment with the lowest likelihood of pip package installation failures.<br><br>I like being able to type `pip install` and have that package's functionality availble just seconds later. It drives me crazy when I receive the dreaded <i>"Microsoft Visual C++ Build Tools" </i> package installation error, because this means I'll spend the next twelve hours:<br>
            <ol>
                <li>Searching Google for an answer that actually works.</li>
                <li>Searching the endless Microsoft website for the exact required component.</li>
                <li>Watching my ISP data cap get chewed up as I download components from Microsoft that turn out to not be what I needed.</li>
                <li>Swearing at my computer (and myself) for not having documented the solution the last time I had to endure this ordeal.</li>
            </ol>
        </td>
    </tr>
    <tr>
        <td>004</td>
        <td>Use the standard Python release</td>
        <td>There are multiple solutions to the Python package installation problem mentioned in #003. Two methods I've used are:<br>
            <ul>
                <li>Use <a href="https://www.lfd.uci.edu/~gohlke/pythonlibs/">pre-compiled libraries</a> from Christoph Gohlke</li>
                <li>Use the <a href="https://docs.anaconda.com/anaconda/install/">Anaconda distribution</a> of Python</li>
            </ul>
        While both of these solutions are viable and successfully solved my problems in the past, I dont like having to rely on them beacuse:<br><br>
            <ul>
                <li>Using Gohlke's libraries makes me dependent on the unofficial generosity of a single contributor, and requires trusting the security of his packages.<br><br>(<i>I am <b><u>NOT</u></b> saying that Christoph Gohlke is untrustworthy. Rather, philosophically I don't like the increased complexity of Python library acquisition, and the addition of an extra attack vector).</i></li><br>
                <li>Using the Anaconda distribution ties me to the decisions of Anaconda, Inc. Furthermore, it requires the use of the <a href="https://en.wikipedia.org/wiki/Conda_(package_manager)">conda package management system </a> and <a href="https://conda-forge.org/">conda-forge</a> for packages not available through conda. My personal preference is to avoid learning/remembering the idiosyncracies of the Anaconda stack if I can fulfill the same result with the standard distribution.</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>005</td>
        <td>Use an unopinionated framework</td>
        <td>Using an opinionated web framework like Django would make it easier to stand-up a fully featured-website. However, I'm doing this work to learn and that's harder to do if there is only "one" correct way to do things. Despite the extra problem-solving time and research that an unopinionate framework will demand, the solution should leave the majority of design decisions up to me.</td>
    </tr>
    <tr>
        <td>006</td>
        <td>Use a well-documented framework</td>
        <td>Given Design Consideration #005, I'll need to make many semi-blind decisions. As a result, I should choose a framework that is well-established and has a large community. This will increase the chance that the problem has been previously encountered, solved, and documented. For this reason, I should favour an older framework like Flask over a newer framework like FastApi.</td>
    </tr>
    <tr>
        <td>007</td>
        <td>Use a synchronous framework</td>
        <td>Python has improved its asynchronous programming support <i>(e.g. making async/await reserved keywords in Python3.7)</i>. However, I find asynchronous programming more difficult for two reasons:
            <ul>
                <li>It requires logic to be structured and written differently than traditional synchronous programs.</li>
                <li>It requires a different application server implementation (ASGI-compliant vs. WSGI-compliant) </li>
            </ul>
            Asynchronous programming is clearly preferrable when designing applications to be performant under high load, and this is a style that I should become more familiar with. With that said, I don't believe the extra complexity makes sense for my initial delivery goals. For this reason, I'll stick with WSGI and make a note that I should have a follow-up project to convert to an ASGI-based framework <i>(e.g. moving to a framework like Starlette or FastApi)</i>.
        </td>
    </tr>
    <tr>
        <td>008</td>
        <td>Find right level of database abstraction</td>
        <td>Most real-world applications will require stateful storage. As a result, the solution should come with an pre-built database integration component. <br><br>The major decision to make is whether to leverage an ORM like SQLAlchemy which heavily abstracts my database interactions, or rely more heavily on raw SQL / an SQL query builder.<br><br><b>NOTE:</b>This particular item caused several downstream complications, which I will write about more in later project documentation pages.</td>
    </tr>
    <tr>
        <td>009</td>
        <td>Use source control system that easily integrates with CI/CD solutions</td>
        <td>I will make many mistakes during this project, so I must have a way to roll back changes that I fail to implement succesfully. Source control is mandatory and must not only operate in my preferred local environment but also integrate easily with the cloud source control & CI/CD solution.</td>
    </tr>
</table>

### Design Decisions
Accounting for the stated design constraints, design considerations, my previous project experience, availability of documentation/guides, and future expected benefits to both my job duties and personal goals, I decided on the following solution stack:

<ul>
    <li>
        <b>Development OS</b> <span><h4>Ubuntu 20.04 WSL2 running on Windows 10</h4></span>
        This struck me as the only viable technology stack given my stated constraints and preferences. I didn't want to abandon Windows 10 as my OS, but I *really* didn't want to directly develop on it either.<br><br>Thankfully, Microsoft has perfect solution: <a href="https://docs.microsoft.com/en-us/windows/wsl/wsl2-index">Windows Subsystem for Linux 2</a>.<br><br>The Microsoft documentation re: WSL2 capabilities is extensive and explains the Subsystem's capabilities far better than I could, so I wont try to repeat it here. What is most important to note is that the WSL2 provides a full Linux-development environment that is accessible to Windows. This is not only avoids Windows compilation errors, but also gives native access to a host of Linux support tools like SSH and Make.<br><br>I chose Ubuntu 20.04 LTS as my base Linux image because I have some previous experience working with this flavour of Linux and because it has long-term support <a href="https://ubuntu.com/blog/what-is-an-ubuntu-lts-release">updates until 2029</a>.
    </li>
    <br><li>
    <b>Python package management and isolation solution</b><span><h4>pip & venv</h4></span>
            <a href="https://docs.python.org/3/tutorial/venv.html#managing-packages-with-pip">Pip</a> and <a href="https://docs.python.org/3/library/venv.html">venv</a> aren't new or fancy, but they work so why change?<br>I may eventually look at newer solutions like <a href="https://pypi.org/project/pipenv/">pipenv</a> or <a href="https://python-poetry.org/docs/">poetry</a>, but I see no compelling reason to complicate my life when I already have acceptable solutions shipped with the standard library.
    </li>
    <br><li>
        <b>Local source control</b><span><h4>Git</h4></span>
        Git is extremely easy to set up in Linux and integrate with a Git-based cloud solution <i>(e.g. GitHub, GitLab)</i>. Any developer worth their salt should know the basics of Git, so this is a must-learn topic regardless of project outcome.
    </li>
    <br><li>
        <b>Cloud source control & CI/CD</b><span><h4>GitHub</h4></span>
        It's established, versatile, and now a member of the Microsoft product family. GitHub offers easy-to-use source control, extensive CI/CD capabilities via GitHub Actions, project documentation facilities, and project task management tools. Other services may provide better individual functionality, but GitHub provides a one-stop shop for many. I would rather learn one platform well, than be mediocre at several.
    </li>
    <br><li>
        <b>Cloud hosting provider</b><span><h4>AWS</h4></span>
        Given my evident preference for other Microsoft solutions, you'd be forgiven for thinking I'd pick Azure. Azure is probably worth looking at in the long term, but I have more familiarity with the AWS platform, and have a few Serverless tutorials that I would like to try soon that are all based on AWS infrastructure. Winner: AWS. 
    </li>
    <br><li>
        <b>Container solution</b><span><h4>Docker</h4></span>
        The only possible contender given the constraint that required usign Docker. It helps that the latest iteration of Docker For Windows has tight integration with WSL2. 
    </li>
    <br><li>
        <b>IDE</b><span><h4>VSCode</h4></span>
        A free, full-feared IDE with great official Microsoft language support plugins. Better yet, it is fully integrated with WSL2 and can be directly invoked from the venv CLI. Plus I like how it looks.
    </li>
    <br><li>
        <b>Testing framework</b><span><h4>Pytest</h4></span>
        I'm not super familiar with Python testing frameworks. Regardless, I need an automated testing solution in order to leverage CI/CD effectively, and Pytest works in a way that I find more natural than the Python standard library's unittest module. 
    </li>
    <br><li>
        <b>Database</b><span><h4>SQLite3</h4></span>
        I'll probably revist this at a later point when my Docker container management skills are better. From a getting started point-of-view, however, it's just too easy to use SQLite3 to consider using anything else. It is effortless to install, cross-platform, and exceedingly easy to manage (given that the whole database is just a single file). <br><br>Sqlite3 is not as robust as other database solutions when it comes to handling concurrent connections (i.e. <a href="https://en.wikipedia.org/wiki/Comparison_of_relational_database_management_systems">it can't</a>), but this is not a deal-breaker for my initial needs.
    </li>
    <br><li>
        <b>ORM</b><span><h4>*-SQLAlchemy</h4></span>
        When one talks web frameworks and databases, ORMs always seem to eventually show up. To be honest, I really dislike ORMs; every time I try to learn SQLAlchemy, I find I become confused and lose motivation to continue my project. I swore this time would be different and I would finally become a skilled ORM user <i>(foreshadowing alert: it didn't go well this time either)</i>.<br>I'll be documenting my opinions regarding flask-sqlalchemy and SQLAlchemy more in later articles, but for the purposes of this section, I'll simply say that I started with flask-sqlalchemy and then immediately began backsliding as I tried to build a coherent, scalable application.
    </li>
    <br><li>
        <b>Web framework</b><span><h4>Flask</h4></span>
        Given that this series is about creating a Flask solution, I chose Flask. Surprise!!<br><br>As per the <a href="https://www.jetbrains.com/lp/python-developers-survey-2019/">2019 Jetbrains Developer survey</a>, Flask is the #1 web framework - slightly ahead of Django and miles ahead of the rest of the competition. Given its market dominance and (theoretical) ease of initial deployment, Flask is the obvious selection.
    </li> 
</ul>

Previous: [Why this project?](./01-why-create-this-project.html)
Next: Setting up WSL2 on Windows
