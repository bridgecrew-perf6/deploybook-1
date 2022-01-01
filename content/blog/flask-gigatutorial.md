---
path: /content/blog
date: 2021-12-31T00:36:17.177Z
title: flask giga tutorial
description: in-depth deployment tutorial for Flask, Python, Postgres, Digital Ocean, Nginx, Gunicorn and more!
---

### TODO

    - [ ] wrap in introspection
    - [ ] set up linux computer and follow along with my own work making notes
    - [ ] oswap link & discussion of security: leave it to the experts (but don't ignore)

Welcome! You are about to start on a journey toward deploying a working web application with Python, Flask, and Postgres. When you have finished, this application will be live on the web and you will be able to access it from your phone or computer anywhere you have internet access.

Both the structure and content of this book are shamelessly based on Miguel Grinberg's [Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world). While the focus of this book is on exploring and explaining the surplus of non-obvious tasks required to get a Flask application onto the internet, it should be treated as something of an extended application of the lesson in Chapter 17 of the Mega-Tutorial that provides much more context for the many, many steps of deploying.

In the first part of this book, we will move the present blank page to, at the end, having a website, with a domain name, served over https on nginx and gunicorn, on a cheap remote linux server provided by Digital Ocean. We will be saving discussion of in-depth security, docker, nftables.

assumptions: I am writing this with as few assumptions as possible about your experience with programming or deploying code to websites.

However, there are two requirements: first, the following you will have to pay digital ocean, you may have to pay for dns,

Term of art slurry:
nginx
gunicorn
postgres
digital ocean
https
let's encrypt
git & github

### Roadmap

familiarize with flask/git/venv locally, digital ocean, configure SSH, firewalls and ports, installing application, install production dependencies, configuring gunicorn & supervisor, configuring nginx; (note on introspecting remote processs), https, autorenew with cron, set up dns,

first order security concerns: firewall, secret key,

### Minimal Flask Application

A minimal Flask application, in this context, means a small amount of code that will allow us to demonstrate and test our configuration on the remote server.

[link to github v1 #todo](https://github.com/redmonroe/deploy-linux/tree/v0.1)

### Trying Flask Application Locally

Soon enough we will be sending the code to your remote. However, if this is your first experience with Flask, you may want to clone a copy of the minimal Flask app and experiment with it on your computer at home.

The code for this section can be had at [https://github.com/redmonroe/deploy-linux/tree/v0.1](https://github.com/redmonroe/deploy-linux/tree/v0.1).

### Finding an Ubuntu Server

If you would like to follow along with me, you will need to find a server to work on. As of the end of 2021, [Digital Ocean](https://www.digitalocean.com/) will provide you a Ubuntu 20.04 server for around $5 per month.  (My experience: in writing the book I made frequent changes to my server and the total cost for the month was under $4.)

Once you have created a new Ubuntu 20.04 server in a Digital Ocean(DO) droplet, you spend a moment to familiarize yourself with the DO administrative controls. The first step you should take to configure your server will require you to know what the IP address of your remote virtual server (what DO calls a "droplet").

```
You can find you IP address either by clicking on
[Your-Project-Name] under "Projects" in the sidebar
of the DO admin panel
OR
by clicking on "Droplets" in the sidebar of the DO admin panel.

There you will find - beneath the header "IP Address" -
the address in the form XX.XXX.XXX.XX.

Make sure you can find this address as you will
be using it in the following sections.

```

[do server configuration tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)

### Login via SSH

Since your server is headless, you will not have a desktop interface that you may be used to on your own computer. Instead you will connect to your remote server through an SSH client and send instructions to your server via the command-line. Windows Subsystem for Linux 2 provides OpenSSH by default.

```
To verify installation of OpenSSH: ssh -V
// should return something like
//OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020

Alternately, you can get a list of all packages installed
by using: dpkg --list.  Scrolling down you some ways should show you packages called "openssh-client" and "openssh-server".
```

Using ssh with your server's IP address from the command-line, you will now be able to log into your remote server.

```
$ ssh root@<your-server-ip-address>
```

//You may be prompted to enter a password. What is DO behavior at this stage?
/where are my certifications stored?
/what is the password situation & pitfalls during initial login for DO?

### PasswordLess Login

Instead of continuting to log in as root (that is, the "root" in ssh root@your-server-ip-address), you will be configuring your server to log you in without a password. Instead of using a password, you will use public key authentication in order to verify your identity to your remote server. This method is both more convenient and more secure.

To begin, you should come up with a new username that you would like to use as your username when logging into your server. For the purpose of the illustration in the following examples, I will be using the new user name of "gigaflask" but you are welcome to substitute your own username.

```
$ adduser -gecos gigaflask
//creates new user "gigaflask", -gecos flag disables requirement
to provide information such as name and phone number for GECOS field in passwd file

$ usermod -aG sudo gigaflask
//grants superuser privileges to user "gigaflask"

$ su gigaflask
//"su" stands for "switch user", so command tells current command-line session to switch current user from root to gigaflask.
```

Further Reading:

- [OpenSSH Server Documentation from Ubuntu](https://ubuntu.com/server/docs/service-openssh)
- linux tutorial (need to emphasize this)
- why is passwordless more secure?
- what is root?

### Installing and Configuring Firewall & Exposing Ports

### Install Dependencies

why are some from apt-get and some from pip?

(work in progress please hang on . . .)
