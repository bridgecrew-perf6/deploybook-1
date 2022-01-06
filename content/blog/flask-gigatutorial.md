---
path: /content/blog
date: 2021-12-31T00:36:17.177Z
title: flask giga tutorial
description: in-depth deployment tutorial for Flask, Python, Postgres, Digital Ocean, Nginx, Gunicorn and more!
---

TODO: formatting

- [ ] change slash comments to #
- [ ] if using program or package name as a command use _italics_, if using as an adjective or process use SSH in caps


      TODO

- [ ] how do I want to handle initial flask steps
- [ ] how do I want to handle text editors on remote _vim_ and _nano_
- [ ] wrap in introspection
- [ ] set up linux computer and follow along with my own work making notes (I can do this with)
- [ ] oswap link & discussion of security: leave it to the experts (but don't ignore)
      [owasp top10](https://owasp.org/www-project-top-ten/)
- [ ] does debug server matter to this type of flask deployment?
- [ ] don't forget about having to enter the passphrase (how are you supposed to securely handle these password?)
- [ ] what is my package called: deploy-linux? I am going to just be stuck with that unless I am shown that I am otherwise wrong

TODO: nginx:

- go back to digitalocean docs on configuring nginx?
- what helped me figure out the enable reload situation?
- order of
- [ ] So first we deploy with nginx with http active, then domain name, then https, then reconfigured nginx with certbot
- configure and check logs for nginx

TODO: login via SSH, score: 4/5

- [ ] what is ux of reader logging into DO first time? will they need a password where does it come from?

TODO: passwordless login:

- [ ]
  TODO: HTTPS

### Hello World

Welcome! You are about to start on a journey toward deploying a working web application with Python, Flask, and Postgres. When you have finished, this application will be live on the web and you will be able to access it from your phone or computer - anywhere you have internet access.

Both the structure and content of this book are shamelessly based on Miguel Grinberg's [Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world). While the focus of this book is on exploring and explaining the surplus of non-obvious tasks required to get a Flask application onto the internet, it should be treated as something of an extended application of the lesson in Chapter 17 of the Mega-Tutorial that provides much more context for the many, many steps of deploying.

NEEDS LOVE: In the first part of this book, XXXXXXXXXXXXXXXXXXXXXxxxxx to, at the end, having a website, with a domain name, served over https on nginx and gunicorn, on a cheap remote linux server provided by Digital Ocean. We will be saving discussion of in-depth security, docker, nftables.

assumptions: I am writing this with as few assumptions as possible about your experience with programming or deploying code to websites.

However, there are two requirements: first, the following you will have to pay digital ocean, you may have to pay for dns,

Term of art slurry:
postgres
https
let's encrypt
git & github

### Roadmap

familiarize with flask/git/venv locally, configuring nginx; (note on introspecting remote processs), https, autorenew with cron, set up dns,

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
# should return something like
# "OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020"

# Alternately, you can get a list of all packages installed
# by using: dpkg --list.  Scrolling down you some ways should show
# you packages called "openssh-client" and "openssh-server".
```

Using _ssh_ with your server's IP address from the command-line, you will now be able to log into your remote server.

```
$ ssh root@<your-server-ip-address>
```

### Passwordless Login

Instead of continuting to log in as root (that is, the "root" in ssh root@your-server-ip-address), you will be configuring your server to log you in without a password. Instead of using a password, you will use public key authentication in order to verify your identity to your remote server. This method is both more convenient and more secure. (Public key authentication is one of the most important inventions of all-time and I recommend learning at least a little about it).

To begin, you should come up with a new username that you would like to use as we proceed for logging into your server. For the purpose of illustration in the following examples, I will be using the new username of "gigaflask" but you are welcome to substitute your own username. Just make sure you are able to refer to it.

```
$ adduser -gecos gigaflask
# creates new user "gigaflask", -gecos flag disables requirement
# to provide information such as name and phone number
# for GECOS field in password file

$ usermod -aG sudo gigaflask
# grants superuser privileges to user "gigaflask"

$ su gigaflask
# "su" stands for "switch user", so command tells current command-line session to
# switch current user from root to gigaflask.
```

With the new user "gigaflask" created, the next step is to configure public key authentication. Once this is configured you will no longer have to type a password when logging in to your server from ssh.

Note: Configuring public key authentication is one of the most intimidating steps of configuring a deployment. The process involves manipulating weird, long numbers with terminal commands with which you may not have familiarity. Furthermore, the terms "key", "authentication", and "private" raise the anxiety level; if I do this wrong will hackers get into my server? can I fix this if I mess up? Please rest assured however that nothing you are doing in this step of the deployment is irrevocable. Stripped of the terms of art, the process itself is fundamentally a simple one.

I will walk through the basic steps of generating a public key on your local computer and then configuring your remote server to accept this form of authentication. You may want to read through all the steps before beginning. After we have gone through the steps I will provide some ways to look into the components of public key authentication to determine whether you are configured correctly.

If you have been following along with this project, you will still be logged into the remote after creating the new user "gigaflask". Since the next step requires that you create a private key on your local computer, you will need to open a second terminal window (and do not log into the remote server).

In the second terminal window, we will be checking the contents of the _~/.ssh_ directory to determine whether or not you ALREADY have a private key.

```
$ ls ~/.ssh  # lists contents of ~/.ssh folder

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

### Installing the Application on the Remote Server

In this step we will deplou our application source code to the remote server. To download (or "clone") the application source code to your remote server, make sure that you are logged via _ssh_ to the "gigaflask" account on the remote server.

```
$ git clone MAKE SURE TO GET V0.1 IN THERE
$ cd deploy-linux
$ git checkout MAKE SURE i HAVE THE RIGHT VERSION
```

These commands download the code and installs it on the remote server. Since we using a minimal Flask application to demonstrate deployments rather than the functionality of Flask, we are using only a tiny packet of Flask code to run our application. Thus you should make sure to checkout the version named INSERT HERE //ALSO TALK ABOUT MY VERSION V YOUR VERSION OF CODE ON GIT IF YOU DID SOMETHING DIFFERENT IN THE FAMILIARIZING YOURSELF WITH FLASK CHAPTER ABOVE.

In the next set of commands, we will be activating a virtual environment and installing the dependencies list in the _requirements.txt_ file you have just cloned from the Github repository.

```
$ python3 -m venv venv //activate the virtual environment
$ source venv/bin/activate
$ (venv) $ pip install -r requirements.txt  //installed contents of requirements.txt into the virtual environment
```

Additionally, we are going to install a further package from pip for our production deployment. The _gunicorn_ package (short for, green unicorn, I am told) is a production web server for Python applications.

```
(venv) $ pip install gunicorn

```

### Setting up Gunicorn and Supervisor

When you start your Flask server with _flask run_, you are using the simple development server that comes with Flask. While useful in development, this server is not designed with the functionality and performance needed for a production server. Instead we will ust _gunicorn_, a Python webserver, widely used for serving in production.

If you want to simply start _gunicorn_, you can use the following command.

```
(venv) $ gunicorn -b localhost:8000 -w 4 [WHICH ONE IS THIS: WHICH ONE IS THIS]

```

//-b tells gunicorn on which port to listen for requests, here port 8000.
//-w configures how many workers gunicorn will run, here we have configured 4. While having more workers will increase the number of clients your application can handle, if you have more workers than can be handled by the amount of RAM on your remote server you will experience performance issues and you will then need to adjust down the number of workers you provide.

The _app:app_ argument tells _gunicorn_ how to load the application instance. \*\*\* WHY CAN'T I GET THIS TO MAP TO GRINBERG'S CODE V. MY DEPLOY-LINUX CODE?

The main drawback against running _gunicorn_ from the command-line is that if your remote server crashes, you will need to manually restart your application. Instead, we are going to use the _supervisor_ package, which we have already installed to ensure that our Flask application starts when the remote server is rebooted.

_supervisor_ uses a configuration file that configures what programs to wathc and how to restart them when it is necessary. _supervisor_ stores its configuration file in _/etc/supervisor/conf.d/_. We are going to make a configuration file called _deploy.conf_ and store the following code.

```
[program:deploy]
command=/home/gigaflask/deploy-linux/venv/bin/gunicorn -b localhost:8000 -w 4 app:app
directory=/home/gigaflask/deploy-linux
user=gigaflask
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
```

If you used a username other than gigaflask when you configured your username for your remote server, you should substitute that username for "gigaflask" in the above code. Also, if you cloned your own version of the minimal Flask app for this deployment, you will you use that folder name in place of "deploy-linux".

//need an edit bad, this is a direct copy paste
The command, directory and user settings tell supervisor how to run the application. The autostart and autorestart set up automatic restarts due to the computer starting up, or crashes. The stopasgroup and killasgroup options ensure that when supervisor needs to stop the application to restart it, it also reaches the child processes of the top-level gunicorn process.

Once you have saved the changes to your new _supervisor_ configuration file, you can restart the supervisor service with the following command.

```
$ sudo supervisorctl reload
```

Now your application should be running under the watchful eye of the _supervisor_ package.

///how to introspect whether any of this is working? I think I have good notes and practices on this

Further Reading:

-[git for beginners](http://ryanflorence.com/git-for-beginners/)

### Domain Name and DNS

Because we are going to use our domain name in the configuration of nginx, our production server. The next step is to set up our domain name and link our DigitalOcean server to our new domain. There are many sites from which you can obtain a domain name, both paid and free. While I do not endorse any site, I have experience with [Namecheap](www.namecheap.com) so if you'd like to follow along, we will be using that site.

You will be required to pay Namecheap for a domain name. The costs for this domain name starts around $5.  I was able to get a domain in the name of my business for about $8 a year (at least for the first year).

### Configuring Nginx: SSL Certificate

Your Flask application is now served by _gunicorn_ on its own private 8000 port according to the configuration code you wrote into the _supervisor_ configuration file in the prior passage. To continue your deployment, you will next need to expose your application to the wider world by enabling another web server, called _nginx_, to allow access via ports 80 and 443. If you remember these are the same ports that you configured your firewall to open.

We are going to configure port 80 to forward all traffic to port 443; port 443 being the channel for encrypted traffic under HTTPS. In order to use HTTPS, you will need an SSL certificate. Later, we will walk through the process of obtaining a REAL SSL certificate from the certificated authority Let's Encrypt but for now we will use a _self-signed SSL certificate_. This is sufficient for our purposes of practicing the steps of deployment. However, please be aware that with a self-signed certificate that your browser will warn you and any users of the site that the certificate is not trusted.

In any case, the following command will create an SSL certificate for your project. You should execute this function from the root directory of your Flask project on your remote server. Be aware: the following command is going to ask you for information about your idenity and application that will be included in the SSL certificate. Web browsers will show this information to users if they request to see it (by, for example, clicking on the little lock icon in the far left of the Chrome Omnibar).

```
$ mkdir certs //make a new directory called "certs"
$ openssl req -new -newkey rsa:4096 -day 365 \
- nodes -x509 -keyout certs/key.pem -out certs/cert.pem
```

This command results in to files being created in your project's root directory called _key.pem_ and another in the folder just created called _certs_ called _cert.pem_. To verify the existence of these files, you can check with:

```
$ ls -a  //you should see a file called key.pem and a sub-directory called certs.
```

### Configuring Nginx: Nginx Config File

In order to serve your project with nginx, you will need to create an nginx configuration file.

The directory for the configuration for your DigitalOcean installation is _/etc/nginx/sites-enabled/_. Since nginx at installation installs a test site in this folder, the first step we are going to take is to remove this file, which we do not need. This is done with the following command:

```
$ sudo rm /etc/nginx/sites-enabled/default
```

The next step is to configure nginx. If this is your first time deploying to a remote server, this is perhaps the most difficult step. The configuration file is written in a language that is likely unfamiliar to you, there are numerous location in the config file where you must substitute your paths and package names, and - unless your website works perfectly after you save this file for the first time - it can feel difficult to get feedback from nginx about which line is causing the problem.

To mitigate the pain of this circumstance, I am first going to show you the text of the configuration file that you should use together with comments about what each section of the configuration file is configuring. Later, I will give you some simple ways to introspect the nginx server's status on your remote so that if something is not working you can narrow down the causes. Finally, I will provide additional sources that I found useful during my first deploy and in my research for this book.

For the first step, you are going to write the configuration in the directory referenced above _/etc/nginx/sites-enabled_ in a file called _deploy-linux_. The content of the file should be as follows:
{your_domain}

<i>/etc/nginx/sites-enabled/deploy-linux</i>

```
server {
        listen 80;
        listen [::]:80;

        root /var/www/{your_domain}/html;
        index index.html index.htm index.nginx-debian.html;

        server_name {your_domain}.com www.{your_domain}.com;

        location / {
        # forward application requests to the gunicorn server/manually added by me; not certbot
        proxy_pass http://localhost:8000;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

```

//fix the comments in the config
nginx -t thing, what is that put that in

After you have saved this file, you will need to reload the configuration file in order to activate it:

```
$ sudo service nginx reload
```

Your application is now deployed. But it is not time for congratulations just yet. To check the deployment, you can enter the IP address of your server or, if you have configured your domain name with Namecheap and DigitalOcean, you can enter the domain name in browser address bar. Since you only have a self-signed certificate, you should expect a warning from your browswer which you will need to click through.

In a following chapter, we will learn how to configure a proper SSL certificate for free with Let's Encrypt. However, first, I would like to cover some ways to troubleshoot your nginx configuration if you were not able to deploy and view your application.

### Troubleshooting Nginx Configuration

If you find that your configuration is not serving your application properly, the following commands may be useful:

```
$ sudo apt-get install nginx //command to install nginx
$ sudo service nginx status //check status of nginx server, will show green "active" if ok.
$ systemctl service nginx //same as above
$ sudo systemctl stop nginx
$ sudo systemctl start nginx
$ sudo systemctl restart nginx
$ sudo systemctl reload nginx //use if you are only making configuration changes
$ sudo systemctl disable nginx  //disable nginx from starting at boot
$ sudo systemctl enable nginx //re-enable nginx to start at boot

//have nginx check configuration file
$ sudo nginx -t //should return "nginx: ... syntax is ok" if not
you probably need to double check your configuration file.

//firewall commands

$ sudo ufw status //check status of firewall
$ sudo ufw app list //check open ports:
```

### Nginx Further Reading

-[How to Install Nginx on Ubuntu 20.04](<https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04#step-5-%E2%80%93-setting-up-server-blocks-(recommended)>)

-[Nginx Docs](http://nginx.org/en/docs/beginners_guide.html#:~:text=By%20default%2C%20the%20configuration%20file,%2Flocal%2Fetc%2Fnginx%20.) very useful but far from easy reading

### Pushing Updates to Your Remote Server

Since our minimal Flask application is so limited in function, it is likely you will want to add features in the code. If you do make such changes, you can use _git pull_ from the remote server to download the new commits made since the prior deployment.

However, simply downloading the code to the remote is not sufficient to upgrade, the server processes will continue running the old code stored in memory.

Instead, you will need to stop the current server and force the server to read the new code and then restart the server. Most upgrades that you will do to a site will include action like updating the database but to push simple changes like styling or minor changes in code, you can use:

```
(venv) $ git pull  //downloads new code version
(venv) $ sudo supervisorctl stop deploy-linux //stops the server, sub your app name as appropriate
(venv) \$ sudo supervisorctl start deploy-linux //starts new server
```
