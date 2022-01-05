---
path: /content/blog
date: 2021-12-31T00:36:17.177Z
title: flask giga tutorial
description: in-depth deployment tutorial for Flask, Python, Postgres, Digital Ocean, Nginx, Gunicorn and more!
---

TODO

    - [ ] get rid of other blog posts on site
    - [ ] how do I want to handle initial flask steps
    - [ ] how do I want to handle text editors on remote *vim* and *nano*
    - [ ] wrap in introspection
    - [ ] set up linux computer and follow along with my own work making notes (I can do this with)
    - [ ] oswap link & discussion of security: leave it to the experts (but don't ignore)
    [owasp top10](https://owasp.org/www-project-top-ten/)
    - [ ] does debug server matter to this type of flask deployment?

### Hello World

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

With the new user "gigaflask" created, the next step is to configure public key authentication. Once this is configured you will no longer have to type a password when logging in to your server from ssh.

Configuring public key authentication is one of the most intimidating steps of configuring a deployment. The process involves manipulating weird, long numbers with terminal commands with which you may not have familiarity. Furthermore, the terms "key", "authentication", and "private" raise the anxiety level; if I do this wrong will hackers get into my server? can I fix this if I mess up? Please rest assured however that nothing you are doing in this step of the deployment is irrevocable. Stripped of the terms of art the process itself is fundamentally a simple one.

I will walk through the basic steps of generating a public key on your local computer and then configuring your remote server to accept this form of authentication. You may want to read through all the steps before beginning. After we have gone through the steps I will provide some ways to look into the components of public key authentication to determine whether you are configured correctly.

If you have been following along with this project, you will still be logged into the remote after create the new user "gigaflask". Since the next step requires that you create a private key on your local computer, you will need to open a second terminal window (and do not log into the remote server).

In the second terminal window, we will be checking the contents of the ~/.ssh directory to determine whether or not you ALREADY have a private key.

```
$ ls ~/.ssh  //lists contents of ~/.ssh folder

```

If the result of the above command returns <i>id_rsa</i> and <i>id_rsa.pub</i> then you already have a key and can use this key to configure the remote server.

However, if the above command does not return the two files OR if you do not have an ~/.ssh directory at all, you will need to create an SSH keypair by using a utility called <i>ssh-keygen</i>.

```
$ ssh-keygen
//command to start the procedure for creating a keygen
```

//what are the questions?

After you have finished the ssh-keygen steps, you should check that you have an <i>~/.ssh</i> directory, an <i>id_rsa</i> file, and an <i>id_rsa.pub</i>file.

The <i>id_rsa.pub</i> file is your public key (notice .pub ending) and this is the key you will provide to your remote server in the next step. DigitalOcean and other third parties will use this key as a way to verify your identity. The <i>id_rsa</i> file is your private key, you will keep this file and its cryptographic contents on your computer. You should not give this key to anyone.

In the next step, we will use the <i>cat</i> command to view the contents of your public key file (<i>id_rsa.pub</i>).

```
$ cat ~/.ssh/id_rsa.pub

//if you were successful generating a
private key in the prior step you will
see a very long series of letters and
numbers that is the content of your
public rsa key followed by your laptop name.

//Example public key output:
ssh-rsa AAAAB3NzaD1fc2EAAABAQCjw....F9lXv5f/9+8YD joe@joelaptop

```

In the next step we will be copying this key to a location in the directory structure of your remote server.

Thus, first copy the public key you just generated to the clipboaord. Then, return to the original terminal window (the one logged into the remote server). Finally, issues the following command.

```
$ echo <paste-YOUR-public-key> >> ~/.ssh/authorized_keys  //the echo command displays a line of text and combined with ">>"; it "displays" the line of text into the authorized_keys file

$ chmod 600 ~/.ssh/authorized_keys //chmod stands for "change mode of access" and allows a Ubuntu/Linux user to change who and how much access a user has.  600 is an argument passed to chmod command.  It gives the owner full read and write access to the target file, here authorized_keys, while preventing any other user from accessing the file.
```

Once you have entered these commands, you will be able to log into your remote server without a password. From now on, when you log into the remote server <i>ssh</i> will identify itself to the remote server and trigger a cryptographic procedure that requires a public key. The remote server then checks that the procedure is correct and that you are verified by referrencing the public key which you have just provided.

To check work, you should first log out of both your <i>ssh</i> session and your remote session. Then you will attempt to login directly to your "gigaflask" account by entering

```
$ ssh gigaflask@<your-server-ip-address>
```

If your work has been successful you should not have to enter a password and (depending on your bash configuration) you will see <i>gigaflask@your-server-ip-address</i> at the prompt in your terminal.

### Server Security: First Steps

We are now going to take three steps to reduce the number of methods by which an attacker could gain access to your remote server. First, we are going to disable root logins via <i>ssh</i>. Second, we are going to disable login for all accounts on the remote server. Third, and finally, we are going to install a firewall on the remote server. Firewall software protects your server by blocking access to the server on ports that are not explicitly enabled by you.

In the next two steps, we are going to be making two small changes to the text in a configuration file located at _/etc/ssh/sshd_config_.

Because this file is located on your remote server, you will probably not have access to your regular text editor or IDE that you are used to using to make changes in the text. There are many ways to handle this issues; the simplest method is to simply use the _nano_ editor that is provided in (nearly) all installations of Linux.

To disable root logins via _ssh_, first you will log back into your remote server:

```
$ ssh gigaflask@your-server-ip-address
$ sudo nano /etc/ssh/sshd_config  //this will open the SSH configuration file
```

Once open you should scroll down inside the terminal-based _nano_ editor window to the line that starts with ==PermitRootLogin==. There you will change the value to ==no==.

The second change you will make is located in the same file. Once you have made this change you will have disabled all password logins for all accounts. Since you have already enabled password-less logins via public key authentication there is no need to permit password authentication on your remote server at all.

To make this change - while still in your _nano_ session inside _sshd_config_ - scroll to the line ==PasswordAuthentication==. There you will change the value to ==no==.

To complete the configuration of these two values, you will restart SSH so that the change will take effect.

```
$ sudo service ssh restart //this stops ssh and starts it again; initializing the changes.
```

The third change you will make is to install a firewall. The following commands install the Uncomplicated Firewall(ufw) and configure it to block access to all ports with the exception of port 22(ssh), port 80(http), and port 443(https) which we will explicitly enable.

```
$ sudo apt-get install -y ufw //installs ufw
$ sudo ufw allow ssh //open port 22
$ sudo ufw allow http //open port 80
$ sudo ufw allow 443/tcp //open port 443
$ sudo ufw --force enable //enable command reloads ufw and enables firewall on boot, force command disables interactive script

```

Once you have completed these steps you can check your work with:

```
    $ sudo ufw status //will show active if your install and configuration were successful.
```

Further Reading:

- [OpenSSH Server Documentation from Ubuntu](https://ubuntu.com/server/docs/service-openssh)
- [how often should I rotate my ssh keys](https://tailscale.com/blog/rotate-ssh-keys/)
- linux tutorial (need to emphasize this)
- why is passwordless more secure?
- what is root?
- what is public key authentication?
- [An Introduction to Uncomplicated Firewall](https://www.linux.com/training-tutorials/introduction-uncomplicated-firewall-ufw/)

### Install Dependencies

Since you have deployed into a Ubuntu 20.04 Server on your DigitalOcean remote server, you have a system that - as of the date of this writing - comes with Python 3.8.

In addition to Python, we are going to install additional packages that will add further functionality to our eventual deployment and also make deploying more convenient.

```
$ sudo apt-get -y update
$ sudo apt-get -y install python3 python3-venv python3-dev
$ sudo apt-get -y install supervisor nginx git
```

why are some from apt-get and some from pip?

(work in progress please hang on . . .)
