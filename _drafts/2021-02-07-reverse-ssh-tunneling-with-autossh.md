---
layout: post
title:  "Reverse SSH Tunneling with Autossh"
date:   2021-02-07 20:29:05 +0300
categories: ['tutorial', 'notes-to-myself', 'vps']
author: Tolga Üstünkök
---

## Introduction
Followings are the useful links for the construction of this post.
* https://www.everythingcli.org/ssh-tunnelling-for-fun-and-profit-local-vs-remote/

### Highlights
The most important parts from the above links are given below as quotes.

> You can relay a port from a remote server to your local machine with ssh -L,
hence called local port forwarding.

> Remote port forwarding is to make your local port available on a remote server
(ssh -R).

>It should now be pretty easy to remember: Local and remote port forwarding
always refers to where to relay the port to.

> The SSH command syntax uses the same easy to remember abbreviations: `-L`
(forward to my local machine) and `-R` (forward to my remote machine).
