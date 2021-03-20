---
title: 'publish playframework to production'
date: Wed, 19 Oct 2016 23:06:29 +0000
draft: false
tags: ['beepify.io', 'dist', 'nginx', 'playframework', 'production', 'selfhackaton', 'server']
---

Running an playframework application locally is very easy, you just go to the folder, and run
```
activator
run
```

But how to put all of the necessary files at a server? end of the day we are talking about lots of files and dependencies. Luckly, [playframework has a command and instructions](https://www.playframework.com/documentation/2.5.x/Deploying) for that, its called

```
dist
```

That creates an zip file with all the necessary libraries and config files in order to run the application at the server. The name of the file will be based on the version that is setup in the build.sbt file, for example: version := "1.0", would result in a file named: beepify\\target\\universal\\beepify-1.1.zip After the dist command finishes, just upload the file to the server. Simple solution is create an user at the server, [connect using SFTP/ SSH](https://filezilla-project.org/) and send the file to the /tmp folder. After the upload install Java and unzip the file to the folder where it will live, I /var/www/beepify-1-1, here are the commands I used after logging on the server as root \[yes I know that shouldn´t log as root :)\] :

```
sudo apt-get install default-jdk
sudo apt-get install unzip
cd /tmp
unzip beepify-1.1.zip
mv beepify-1.1 /var/www
cd /var/www/beepify-1.1/bin
nohup ./beepify &
```

The last command, the one with nohup, captures the console outputs from the application in a file called nohup.out and the & at the end make sure that you can close the console without closing the application. When you start the application the file RUNNING\_PID is created at the application folder with the process id of the running application, If the application crashes you need to remove this files before you start it again. Other important configuration is to setup nginx to redirect /beepify to the 9000 port where the application is running. but there is one little issue that I had to deal with. A simple reverse proxy of with /beepify, would result in a call to host:9000/beepify/ at my application, in order to makeit work, I would need to change all my routes, adding the application name, so the routes would be /beepify/products for example instead of /products. That wasn´t acceptable, let´s say I want to change the exposed url for any reason, that would be hardcoded in the application. let´s we need to publish more than one url and split traffic for example? so fixed preffix names are not a good idea. I rather created an rule in the proxy to remove from the urls the preffix /beepify when redirecting to the application at port 9000. This is the location definition I used on /etc/nginx/nginx.conf location

```
location /beepify {
   rewrite ^/beepify(/.\*)$ $1 break;
   proxy\_pass http://127.0.0.1:9000/;
}
```

See more rules here: [Creating NGINX rewrite rules](https://www.nginx.com/blog/creating-nginx-rewrite-rules/)