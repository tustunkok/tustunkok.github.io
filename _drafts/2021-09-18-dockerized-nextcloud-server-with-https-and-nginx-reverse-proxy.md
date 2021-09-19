---
layout: post
title:  "Dockerized Nextcloud Server with HTTPS and Nginx Reverse Proxy"
date:   2021-09-18 14:11:10 +0300
categories: ['tutorial', 'notes-to-myself', 'vps']
author: Tolga Üstünkök
---

# Introduction
Data is an essential virtual matter to move forward in today's world. We create,
interpret, and store all kinds of data in different forms every day. Depending
on the objective, either creation, interpretation, or storage gains importance
over the others. In this article, I will try to explain a storage option for all
kinds of data.

Of course, there are multiple solutions for such a seemingly simple problem at
first sight. The commercial ones are fairly popular such as; [Google
Drive](https://www.google.com/drive/), [Dropbox](https://dropbox.com),
[OneDrive](https://www.microsoft.com/en-ww/microsoft-365/onedrive/online-cloud-storage),
and [Box](https://www.box.com/home) are just a few of them. All of these
commercial cloud storage services work great despite their drawbacks. Sometimes
those drawbacks are just too much for their users to stay and use the services.
Maybe the worst of them, in my opinion, is Dropbox. Its starting plan provides
too little cloud storage space and a restricted simultaneous device connection
policy of up to three devices. As an opponent of Dropbox, Google Drive works
much better for an average user. It comes with 15GB of cloud storage
(unfortunately shared with the email account). However, it is super cheap of 5TL
for 100GB. These features are definitely an improvement over Dropbox.

Things start to change when you accumulate multiple hundreds of gigabytes of
films, books, and other staff and run out of space. The cost of the commercial
cloud storage units might start to hurt a little bit at this point. Fortunately,
there are great open-source cloud data storage platforms like
[ownCloud](https://owncloud.com/) and [Nextcloud](https://nextcloud.com/).
Nextcloud actually is a fork of ownCloud, and it seems more popular than
ownCloud.

## Nextcloud
Nextcloud is a [PHP](https://www.php.net/)-based cloud storage and collaboration
software. It allows its users to put, share, and work on the same or different
data. In addition to that, one can considerably increase its functionality with
lots of 3rd party applications. Nextcloud supports multiple devices with a
strong and very active community, which is a critical detail when it comes to
open-source projects.

Nextcloud comes with lots of different installation options. Enterprise and
non-self-hosted solutions are paid services, and self-hosted solutions are, of
course, free. The most important option among the other installation options,
Nextcloud has its official [Docker Image](https://hub.docker.com/_/nextcloud) in
[DockerHub](https://hub.docker.com/). Therefore, testing and preparing an
instance up and running is extremely easy. To simplify things further, Nextcloud
also offers a [Docker Compose](https://github.com/nextcloud/docker)
installation, which includes a database (MariaDB), a caching service (Redis), a
cronjob service (cron), and Nextcloud's itself. One can effortlessly get a
Nextcloud server up and running with just a few modifications on the
configurations. This is the main topic of the whole article. I hope you enjoy
and learn from it. Have a good reading.

![Overall Architecture](/assets/images/overall-architecture.svg)
