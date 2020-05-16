---
layout: post
title:  "How to Create a Containerized Multi-User JupyterHub Research Server?"
date:   2020-05-16 13:12:34 +0300
categories: ['tutorial', 'notes-to-myself', 'vps']
author: Tolga Üstünkök
---

# Introduction
Development servers with high computation powers are important for research
groups. Because of that, there are multiple solutions from various companies. 
Google has its [Colab][colab-link] environment as well as the 
[Kaggle][kaggle-link] community. Those environments supply processing power 
including [GPUs][gpu-wikipedia] and [TPUs][tpu-wikipedia] to their users.

The main problem with these environments is to isolate each users' environment 
from each other via a containerized solution. At this point, you may be heard of
the name [Docker][docker-link]/[Kubernetes][kubernetes-link].
Most of those solutions use Docker under the hood. If you have any experience 
with, for example, Docker, the chances that you might see the current Docker 
version text in the sidebar is very high.

Isolating environments from each other is one point. However, another point is 
the access management part of those isolated environments. In Kaggle, access 
management to the containers is embedded in the site itself. In Colab, this 
process is handled by Google Accounts. Thus, if you want to set up a site like 
those, you have to handle the access management by yourself.

## JupyterHub
[JupyterHub][jupyterhub-link] is an open-source multi-user version of the 
notebook designed for companies, classrooms and research labs. It is designed to
manage user access to notebook servers via several authentication mechanisms. 
Essentially, JupyterHub is a web server running as a proxy to other isolated 
Jupyter Notebooks/Jupyter Labs used by different users. The following image is 
taken from the [official documentation][jupyterhubdocs-link] of JupyterHub and 
explains the working principle of it.

![JHW](https://jupyterhub.readthedocs.io/en/stable/_images/jhub-fluxogram.jpeg)

### Authentication
JupyterHub provides some basic user management and administrator features. For 
example, you can whitelist some users or blacklist them, assign special 
permissions to them, [etc][jupyterhubauth-link].

For the authentication, you use one of the following methods:
- Local Authenticator
- OAuthenticator
- Dummy Authenticator

**Local Authenticator** manages users on the local system. If you add a new user
to JupyterHub, the user is created for the system with the command `adduser`. 
If you do not want to change the system users, you should not use this 
authenticator.

**OAuthenticator** is a way to get access to protected data from an application.
It is safer and more secure than asking users to log in with passwords. If you 
are planning to open JupyterHub to the public, this may be the right choice.

**Dummy Authenticator** is the simplest type of authenticator. It allows you to 
set any username-password pair. You can also set a global password and all users
who know the correct global password can successfully login to the JupyterHub.

### Notebook Spawners
JupyterHub's main purpose it to spawning single-user notebooks for the 
authenticated users. A single-user notebook is just an instance of 
`jupyter notebook`. Jupyter notebooks are available in many formats. One can 
install it via package managers (i.e. `conda`, `pacman`) or via `pip`. However, 
if you do not want to fill your machine with random junk, you can run notebooks 
on a Docker image. You can find official docker stacks in the 
[Github][dockerstacks-link].

For spawning notebook servers, JupyterHub provides the following methods:
- Native Spawner
- Docker Spawner

**Native Spawner** spawns notebooks from the installed version of the Jupyter 
Notebook. This version uses the libraries and frameworks from the main system. 
To add new libraries or frameworks, you have to install them into the main 
system.

**Docker Spawner** spawns notebooks from a prepared Docker image. If you need to
add libraries or frameworks to the notebooks, you have to add them to the 
respected `Dockerfile` and rebuild the Docker image. This spawner type has the 
main focus of this post.

# Customizing the JupyterHub Docker Image
JupyterHub has an [official Docker image][jupyterhubdockerimg-link]. This image 
contains only the JupyterHub service itself. The notebook, authentication 
mechanisms and/or any configuration is not included. One has to derive a new 
image from this official image. The correct way of doing this is, of course, 
preparing a new `Dockerfile`.

## Writing the Dockerfile for JupyterHub
The first thing to do in the `Dockerfile` is to setting the base image. In this 
case, base image is the official JupyterHub image. Add the following command as 
the first line of the `Dockerfile` of your custom JupyterHub image.

~~~ docker
FROM jupyterhub/jupyterhub
~~~

Depending on the choice of spawners and authenticators, you have to install some
packages. The following snippet installs `dockerspawner` and `oauthenticator` to
spawn notebooks from a notebook image and using Github accounts for 
authenticaton.

~~~ docker
RUN pip install dockerspawner oauthenticator
~~~

The final part is to copying the configuration file for the JupyterHub to the 
inside of the image by using the following command.

~~~ docker
COPY jupyterhub_config.py .
~~~

This command assumes that the `jupyterhub_config.py` file is in the same 
directory with the `Dockerfile`. The overall view of the `Dockerfile` should 
look likes to the following snippet.

~~~ docker
FROM jupyterhub/jupyterhub
RUN pip install dockerspawner oauthenticator
COPY jupyterhub_config.py .
~~~

When you build the image defined in the `Dockerfile` with the following command,
you will have a ready-to-run JupyterHub image for the next phases.

~~~ bash
$ docker build --rm -t jupyterhub .
~~~

As a refresher, the final dot(.) means the *current directory*. Here the tag of 
the prepared image is `jupyterhub`. If you run the following command, you can 
see the image.

~~~ bash
$ docker image ls
~~~

Output looks like this:

~~~ bash
REPOSITORY              TAG           IMAGE ID        CREATED             SIZE
jupyterhub              latest        cb9bfc0fc293    14 hours ago        935MB
salda-special           latest        4847a31b61b7    6 days ago          9.85GB
tensorflow-notebook     latest        7fe345b290f6    2 weeks ago         8.68GB
scipy-notebook          latest        5f5f71c0d6b9    2 weeks ago         6.89GB
minimal-notebook        latest        f9df51d25171    2 weeks ago         5.91GB
base-notebook           latest        328581e70c7b    2 weeks ago         4.25GB
pclink                  0.1.0-alpha   44ca70efcd30    2 months ago        660MB
nextcloud               latest        7ec24835eb2d    2 months ago        724MB
nginx                   1             6678c7c2e56c    2 months ago        127MB
mariadb                 10.4          1fd0e719c495    2 months ago        356MB
mariadb                 10.4.12       1fd0e719c495    2 months ago        356MB
python                  3-alpine      537dfdf79ddc    2 months ago        107MB
ubuntu                  18.04         72300a873c2c    2 months ago        64.2MB
ubuntu                  latest        a2a15febcdf3    9 months ago        64.2MB
jupyterhub/jupyterhub   latest        64d82994fd55    12 months ago       932MB
openproject/community   8             99757bbbc2a4    14 months ago       1.59GB
~~~

# Customizing the Docker Stack
Luckily, Jupyter project provides official docker stacks for variousb purposes. 
If you follow this [link][dockerstacks-link] you can see many `*-notebook` 
images in the repository. These notebooks are in a hierarchical order. One can 
express this order as a tree structure.

- base-notebook
    - minimal-notebook
        - r-notebook
        - scipy-notebook
            - datascience-notebook
            - pyspark-notebook
                - all-spark-notebook
            - tensorflow-notebook

All notebooks are either directly or indirectly dervived from `base-notebook`. 
This notebook directs the underlying system to install all the necessary 
packages to run a Jupyter (Notebook | Lab).

If you need any additional packages or libraries, you can add it to the any 
`Dockerfile` you want. However, if you changed any node other than a leaf, you 
have to rebuild all the other dependent images to take your changes in effect.

For example, I generally make a brand new image from the `tensorflow-notebook` 
and add whatever libraries or packages that I want to install. By doing this, 
the original `Dockerfiles` stays untouched and the risk of failure is minimized.

[colab-link]: https://colab.research.google.com/
[kaggle-link]: https://kaggle.com/
[gpu-wikipedia]: https://en.wikipedia.org/wiki/Graphics_processing_unit
[tpu-wikipedia]: https://en.wikipedia.org/wiki/Tensor_processing_unit
[docker-link]: https://www.docker.com/
[kubernetes-link]: https://kubernetes.io/
[jupyterhub-link]: https://jupyter.org/hub
[jupyterhubdocs-link]: https://jupyterhub.readthedocs.io/en/stable/
[jupyterhubauth-link]: https://jupyterhub.readthedocs.io/en/stable/getting-started/authenticators-users-basics.html
[dockerstacks-link]: https://github.com/jupyter/docker-stacks
[jupyterhubdockerimg-link]: https://hub.docker.com/r/jupyterhub/jupyterhub/
