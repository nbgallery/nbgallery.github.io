# nbgallery

[nbgallery](https://github.com/nbgallery/nbgallery) is an enterprise Jupyter Notebook sharing and collaboration platform with a goal of making it easier for data scientists and analysts to create, collaborate on, and run code-based analytics.

![nbgallery screenshot](https://cloud.githubusercontent.com/assets/8132519/23445445/9f48c65e-fdf8-11e6-8ef0-d9cb7942b870.png)

## Our Story

As huge fans of iPython/Jupyter, we wanted to be able to easily share and collaborate on Jupyter Notebooks across a large, distributed enterprise.  Specifically, we wanted to empower “citizen data scientists” - those who have the aptitude, curiosity, and creativity to explore data but lack the formal education and technical background of data scientists or computer programmers.

Significantly, most “citizen data scientists” have likely never touched a command line, which has represented a relatively high barrier to entry of running Jupyter Notebooks.  Users or groups within an enterprise would first need to stand up their own Jupyter or JupyterHub instance, which required a background in Linux system administration, or at least some familiarity with a command line.  And after that, they would need a version control system like github to share and collaborate on code-based Jupyter notebooks.

We wanted to make that easier.  For starters, we felt that required the use of command line git to save and checkout code-based notebooks put Jupyter beyond the reach of those who didn’t at least have some software engineering background.  What if we could simplify the sharing and executing of Jupyter Notebooks and allow users from a wide variety of skill sets and backgrounds to benefit from, and collaborate on cutting-edge analytics written in languages like Python and R?

To do this we created nbgallery, which acts as a web-based visual middleman between the user and a remote git repo.  It provides simplified access for non-technical users to search, discover, use and collaborate on code-based analytics.

nbgallery allows for a Jupyter execution environment that is independent from the notebook sharing platform.  Bidirectional integration allows users to run a notebook from an nbgallery server into an independent Jupyter instance, or in reverse, save a notebook from a Jupyter instance to an nbgallery server.  

Since in many cases the data used within a notebook may be sensitive, the integration features automatically strip all notebook output before the notebook can be shared in nbgallery.  This ensures output stays isolated to the user running the notebook, while ensuring the broadest possible sharing of the analytic itself.

We built and maintain a Jupyter Docker image that contains all of the integration capabilities to interact with an nbgallery server.  However we’ve also developed a Python package (called [jupyter-nbgallery](https://pypi.python.org/pypi/jupyter-nbgallery)) which adds nbgallery integration into any existing Jupyter or JupyterHub server.  This allows those who are happy with their existing Jupyter computer environment to still integrate with an nbgallery server.

We built nbgallery with the intent to release it to the open source community.  During its development we made sure it was possible for organizations to adapt it to their own needs, since we ourselves needed customized capabilities to address many of our unique requirements.  Our solution was an extension framework for nbgallery that allows for a generalized open source release, but also allows for customized plug-ins to support an organization’s unique requirements.

As one example, we developed and use an nbgallery extension that requires users to include control markings for each notebook, which are later used for more fine-grained security access.  We also support multiple authentication methods; our internal deployment of nbgallery integrates with a proprietary user authentication service, whereas the open source release includes support for more standard username/password and OAuth-based authentication.

nbgallery has been under active development since November of 2015, and has been in regular use since early 2016.  A big push for our efforts in 2017 is to improve the “discoverability” of notebooks in nbgallery.  We have two areas of focus here - the first is in a recommendation system that can operate on code-based notebooks written in multiple languages.  The second is a system to measure a notebook’s “health” which can identify which notebooks have gone “stale” and no longer run as expected.  Combined, these efforts can help improve the ability for a user to easily discover and use the most relevant notebook to their needs.

Ultimately, the value we see for this effort is empowering users from a wide-variety of backgrounds to create their own analytic solutions.  While users without computer programming experience might initially balk at the thought of writing or executing code-based analytics, we found Jupyter's web-based Notebook to be a very approachable platform.  And nbgallery and its intuitive integration with Jupyter makes it easier for non-technical users across an enterprise to discover and use Notebook-based analytics of interest - allowing them to mix and match pieces of code for their own purpose from other well-vetted notebooks.

# Jupyter-Docker

One of our use-cases involves the use of personal ephemeral (i.e. short-lived) compute environments.  As a result, the time and speed it takes to install Jupyter within that environment was a very important factor to us.  To address this we have developed a minimal [Jupyter Docker image (<250MBs)](https://hub.docker.com/r/nbgallery/jupyter-alpine/) based on Alpine Linux.  This minimal image offers a dozen language kernels, most of which are installed dynamically when the user tries to open a new notebook in that language.  This avoids potentially ballooning the size of the image by trying to “bake in” each and every language kernel.

To maintain such a small image we also couldn’t include every possible language library that an analytic author might need.  Since many notebooks do require language libraries, we’ve created capabilities for both [Python](https://github.com/nbgallery/ipydeps) and [Ruby](https://github.com/nbgallery/iruby-dependencies) to allow language and OS dependencies to be installed on the fly when a user runs the notebook.  This means that users operating in these ephemeral environments do not have to know how to install packages from the command line in order to successfully execute the code within a nbgallery-hosted notebook.
