---
path: /content/blog
date: 2021-12-31T00:36:17.177Z
title: flask giga tutorial
description: first
---

Welcome! You are about to start on a journey toward deploying a working web application with Python, Flask, and Postgres. When you have finished, this application will be live on the web and you will be able to access it from your phone or computer anywhere you have internet access.

Both the structure and content of this book are shamelessly based on Miguel Grinberg's [Flask Mega-Tutorial](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world). While the focus of this book is on exploring and explaining the surplus of non-obvious tasks required to get a Flask application onto the internet, it should be treated as something of an extended application of the lesson in Chapter 17 of the Mega-Tutorial that provides much more context for the many, many steps of deploying.

In the first part of this book, we will move the present blank page to, at the end, having a website, with a domain name, served over https on nginx and gunicorn, on a cheap remote linux server provided by Digital Ocean.

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

### Finding an Ubuntu Server

If you would like to follow along with me, you will need to find a server to work on. As of the end of 2021, [Digital Ocean](https://www.digitalocean.com/) will provide you a Ubuntu 20.04 server for around $5 per month.  (My experience: in writing the book I made frequent changes to my server and the total cost for the month was under $4.)

[do server configuration tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)
