---
layout: post
title:  "A Deeplearning Course Management System with JupyterHub and NVIDIA CUDA"
categories: programming
---

# A Deeplearning Course Management System with JupyterHub and NVIDIA CUDA
## Introduction
The most exciting advancements in machine learning mostly made in the area of neural networks. This situation increases the importance of all branches of the neural networks in both academy and industry. As a result, teaching these topics take attention in both sectors. However, the teaching process might be challenging when your audience gets grows. People from the software industry have some solutions to these challenges.

For those who are interested in ML by using Python and R languages, might be heard of the name Jupyter Notebook or shortly the notebook. Although the Notebook is a great tool to publish an article, it is not a collaborative tool. If you have a class of 50 students and you want each of them has their own notebooks on a server, you need to open a total of 50 ports for each of the Notebook. Besides, each Notebook has to have an authentication method to meet security requirements. These issues lead to the creation of JupyterHub. It is a proxy server that runs on top of the Notebooks. The users/students first authenticate in the Hub. Then, a Notebooks is supplied for them. JupyterHub provides separate home directories and Notebook sessions for each authenticated user. This is a very useful feature since it provides some level of confidentiality.

In this article, I try to explain the installation steps of JupyterHub on top of an Ubuntu 18.04 server. In addition to that, I also introduce some useful extensions for the Notebook. Finally, I will show how to work this setup with an installed GPU card, namely NVIDIA GTX 1080.

## Installation
There are 3 options to install JupyterHub. The first and recommended one is Zero to JupyterHub for Kubernetes (ZJHK). ZJHK can be quickly installed on a Kubernetes cluster. It is highly scalable and able to maintain a large number of users. The second one is The Littlest JupyterHub (TLJH). TLJH is for 1-50 users. It is not scalable since you can install it on a single machine. It provides some features which makes it easily configurable. The final one is the manual one. You can install JupyterHub manually via `PIP` or `conda`.

In this section, I only try to explain the second one, TLJH. The installation is super simple. You can find the installation instructions in the following URL [The Littlest JupyterHub](https://tljh.jupyter.org/en/latest/install/custom-server.html). Simply, open a terminal and paste the commands below.

```
$ apt-get install python3 git curl

$ curl https://raw.githubusercontent.com/jupyterhub/the-littlest-jupyterhub/master/bootstrap/bootstrap.py | sudo -E python3 - --admin <admin-user-name>
```

The second command starts the installation process. When you see the "Done." word, the installation is completed. The installer creates some links to the *systemd* to start the JupyterHub when the system turns on. At this point, JupyterHub is running on port 80. You can access it from the http://127.0.0.1/. The password you entered in your first login will be valid for all of your other logins.

As an admin, you can manage the other users and sessions from the Admin menu at the top bar. For example, you can add or remove users. Moreover, you are a sudo user and can install packages for all users in the system either with PIP or conda. First, you need to open a terminal. You can open a terminal from New -> Terminal. Then, enter a `sudo` command. Some sample commands are given below.

```
$ sudo -E conda install -c conda-forge gdal
$ sudo -E pip install there
```

## NVIDIA Graphics Card Setup
From now on we have a working proxy for the Notebooks. I assume that if you are interested in deep-learning, you have a GPU. Currently, the most popular ML APIs support Nvidia GPUs. But not all of them. If you want to use your GPU in your deep-learning task, it must be a CUDA-enabled card. If you meet these requirements, you can continue reading. Otherwise, you should pass this section.

First of all, the latest proprietary Nvidia drivers should be installed on the system. You can install either from a package manager or manually. I will show you the install steps with a package manager. You can follow the instructions below. These instructions are for Ubuntu distributions.


1. Add the graphics driver PPA and update the sources list.
```
$ sudo add-apt-repository ppa:graphics-drivers
$ sudo apt update
```

2. Install the latest Nvidia driver.
```
$ sudo apt install nvidia-410-driver
```

Now, you should reboot your machine.

3. If you want to remove the installed driver and return to the open-source nouveau drivers.
```
$ sudo apt purge nvidia*
```

After these steps, when you write `nvidia-smi` to the shell, you should see something like:

```
$ nvidia-smi
Sun Dec 16 13:10:03 2018
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 410.78       Driver Version: 410.78       CUDA Version: 10.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 1080    Off  | 00000000:01:00.0 Off |                  N/A |
| 27%   28C    P5    20W / 200W |      0MiB /  8117MiB |      2%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```