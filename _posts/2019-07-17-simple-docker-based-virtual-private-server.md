---
layout: post
title:  "Simple Docker Based Virtual Private Server"
categories: ['notes-to-myself', 'vps']
author: Tolga Üstünkök
---

The aim of this document is to remind me how to get an up and running virtual private server (VPS) from scratch. Every time when I try to migrate my VPS, I find myself completely forgotten to do the job. Today, I decided to write all the steps to a document.

This document explains:

1. Creating a key pair to remote access.
2. Installing Docker.
3. Migrating a Docker container with the volume attached to it.
4. Installing OpenVPN.
5. Installing the Teamspeak 3 server.

As a note to the reader, some of the text is not my own. But you can always find the source at the links provided with them.

# Prepare a VPS Key Pair

The tutorial in the link https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604 is for Ubuntu 16.04. However, it should be same for Ubuntu 18.04, too.

First, create a key pair on the client computer.

~~~
$ ssh-keygen
~~~

By default, *ssh-keygen* will create a 2048-bit RSA key pair, which is secure enough for most use cases (you may optionally pass in the -b 4096 flag to create a larger 4096-bit key). Press *ENTER* to save the key pair into the .ssh/ subdirectory in your home directory or specify an alternate path.

Here you optionally may enter a secure passphrase, which is highly recommended. A passphrase adds an additional layer of security to prevent unauthorized users from logging in.

Copy the public key to Ubuntu Server. The quickest way to copy your public key to the Ubuntu host is to use a utility called *ssh-copy-id*. Due to its simplicity, this method is highly recommended if available.

~~~
$ ssh-copy-id username@remote_host
~~~

Next, the utility will scan your local account for the id_rsa.pub key that we created earlier. When it finds the key, it will prompt you for the password of the remote user’s account. Type in the password (your typing will not be displayed for security purposes) and press *ENTER*. The utility will connect to the account on the remote host using the password you provided. It will then copy the contents of your ~/.ssh/id_rsa.pub key into a file in the remote account’s home ~/.ssh directory called *authorized_keys*.

Disable password authentication on your server. If you were able to log into your account using SSH without a password, you have successfully configured SSH-key-based authentication to your account. However, your password-based authentication mechanism is still active, meaning that your server is still exposed to brute-force attacks. Open up the SSH daemon’s configuration file:

~~~
$ sudo nano /etc/ssh/sshd_config
~~~

Inside the file, search for a directive called PasswordAuthentication. This may be commented out. Uncomment the line and set the value to "no". This will disable your ability to log in via SSH using account passwords:

~~~
...
PasswordAuthentication no
...
~~~

Save and close the file when you are finished by pressing *CTRL + X*, then *Y* to confirm saving the file, and finally *ENTER* to exit *nano*. To actually implement these changes, we need to restart the *sshd* service.

~~~
$ sudo systemctl restart ssh
~~~

You should now have SSH-key-based authentication configured on your server, allowing you to sign in without providing an account password.

# Installing Docker on an Ubuntu 18.04 Machine

You should follow the instructions at https://docs.docker.com/install/linux/docker-ce/ubuntu/.

A quick summary is that; you should uninstall any previous installation of Docker from the machine you used with the following command:

~~~
$ sudo apt-get remove docker docker-engine docker.io containerd runc
~~~

After you make sure that all the related components of Docker are removed, install Docker CE by using the repository of your choice of the operating system. In this example, I will show you for Ubuntu systems. This means that we will use the apt package manager.

First, make sure that you updated your package index:

~~~
$ sudo apt-get update
~~~

Then, install packages to allow apt to use a repository over HTTPS:

~~~
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
~~~

Add Docker’s official GPG key:

~~~
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
~~~

Use the following command to set up the stable repository. To add the nightly or test repository, add the word nightly or test (or both) after the word stable in the commands below.

~~~
$ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable"
~~~

Install the latest version of Docker CE and *containerd*:

~~~
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
~~~

Add yourself to the docker group to use Docker without *sudo* command.

~~~
$ sudo usermod -aG docker $USER
~~~

# Migrate a Docker Container

The following tutorial is taken from https://bobcares.com/blog/move-docker-container-to-another-host/ and https://docs.docker.com/storage/volumes/#backup-restore-or-migrate-data-volumes.

When Docker containers or images are moved from one host to another using export or commit tools, the underlying data volume is not migrated.

The foolproof method is to backup and restore the data volume by providing `--volumes-from` parameter in the `docker run` command.

~~~
$ docker run --rm --volumes-from container_name \
    -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /etc/openvpn
~~~

This command mounts a /backup directory as the local directory. Then, makes a tarball from the OpenVPN configuration files and save it to there.

With the backup just created, you can restore it to the same container, or another that you made elsewhere. The first step is to download the tar file. To do that:

~~~
$ scp username@server.com:/some/remote/directory/backup.tar \
    /some/local/directory
~~~

After that, you need to restore the downloaded tar file to for example a named volume. In this case the name of the volume is `ovpn-data-volume`. To do this you have to create a named volume first.

~~~
$ docker volume create ovpn-data-volume
~~~

Then, you should restore the tar file to the newly created volume with a temporary container.

~~~
$ docker run --rm -v ovpn-data-volume:/etc/openvpn \
    -v $(pwd):/backup ubuntu bash \
    -c “cd /etc && tar xvf /backup/backup.tar --strip 1”
~~~

Above command restores the backup.tar file to the necessary places. After that, you can start the OpenVPN server as usual.

~~~
$ docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp \
    --cap-add=NET_ADMIN kylemanna/openvpn
~~~

# Installing OpenVPN Without Migrating a Volume

This section explains how to install the OpenVPN server in a Docker container without migrating a previously populated volume. The content of this section is pulled from the https://github.com/kylemanna/docker-openvpn.

Pick a name for the `$OVPN_DATA` data volume container. It’s recommended to use the `ovpn-data-` prefix to operate seamlessly with the reference systemd service. Users are encouraged to replace example with a descriptive name of their choosing.

~~~
$ OVPN_DATA="ovpn-data-example"
~~~

Initialize the `$OVPN_DATA` container that will hold the configuration files and certificates. The container will prompt for a passphrase to protect the private key used by the newly generated certificate authority.

~~~
$ docker volume create --name $OVPN_DATA
$ docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none –rm \
    kylemanna/openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
$ docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm -it \
    kylemanna/openvpn ovpn_initpki
~~~

Start the OpenVPN server process.

~~~
$ docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp \
    --cap-add=NET_ADMIN kylemanna/openvpn
~~~

Generate a client certificate without a passphrase.

~~~
$ docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none --rm -it \
    kylemanna/openvpn easyrsa build-client-full CLIENTNAME nopass
~~~

Retrieve the client configuration with embedded certificates.

~~~
$ docker run -v $OVPN_DATA:/etc/openvpn --log-driver=none –rm \
    kylemanna/openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn
~~~

# Installing Teamspeak 3

Starting a Teamspeak 3 server in a Docker container is quite simple. Docker provides an example at https://docs.docker.com/samples/library/teamspeak/. But, you can run the server with the following command.

~~~
$ docker run -d --name ts3srv -v \
    /home/user/ts3server/:/var/ts3server/ -p 9987:9987/udp \
    -p 10011:10011 -p 30033:30033 -e TS3SERVER_LICENSE=accept \
    --restart always teamspeak
~~~
