---
title: 'beepify.io, server side release 1 - part 1'
date: Mon, 17 Oct 2016 19:10:05 +0000
draft: false
tags: ['beepify.io', 'database', 'mysql', 'playframework', 'selfhackaton', 'setup']
---

These are the server side release 1 original plan, that is complete with some minor changes highlighted on each line

*   Install playframework environment - [not in the original plan](https://hamiltonlima.com/blog/beepify-io-implementation-plan/)
*   Create the models
*   Read current location latitude, longitude
*   Add sample data to the database - changed to use a service to create data instead of SQL to add data
*   Create services
*   Get data receive product code, latitude and longitude
*   Return all prices

It´s impressive how environment setup takes time, usually we forget to include this step when planning projects, but believe me this takes of time in a project, let´s go over each step I did on this implementation.

### Playframework install

![2016-10-17_1626](/images/2016/10/2016-10-17_1626.png)

The installation is very straight forward: [download the package](http://playframework.com/download), save to a easy to remember folder, add the bin folder to the path and you are ALMOST done.

Next step create the project 

I say almost, because the first time you create a project, the activator will download all the necessary libraries... seriously this takes time :) and if you are creating your project inside a dropbox folder as I did, remember to disable dropbox while activator is downloading the files.

### How to create the project

*   at the console, at windows: windows / run / cmd
*   cd to the root folder of your projects
*   run: activator new
*   I pick the project type: play-java
*   goto the project folder, in this example: cd beepify
*   run the project: activator run, ... then wait forever to the download complete.

While the download was working I use the time to watch this nice overview on playframework: [https://www.youtube.com/watch?v=bLrmnjPQsZc](http://www.youtube.com/watch?v=bLrmnjPQsZc)

### IDE integration - Eclipse

As I use Eclipse since forever, I really need to make sure I can edit Java code on it. The instructions are very clear on the [documentation](https://www.playframework.com/documentation/2.5.x/IDE), but there is one point that I should highlight:

> COMPILE the project BEFORE running the eclipse command

You will need to add Scala integration so it´s easier to edit the templates, that are by default written in scala, just install this plugin http://scala-ide.org/download/current.html. As I´m familiar with all the option from the plugin I just then all, it worked fine.

If you know how to install plugins in eclipse from urls, here it is: http://download.scala-ide.org/sdk/lithium/e44/scala211/stable/site

Now that you have your eclipse ready to import the project, add the following changes BEFORE you compile the project

on file: project/plugins.sbt

addSbtPlugin("com.typesafe.sbt" % "sbt-play-ebean" % "3.0.2")  
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "4.0.0")

on file: build.sbt

enablePlugins(PlayEbean)

Run the activator at the project folder, and in the activator console run : 

*   reload
*   compile
*   eclipse

After this finished go to eclipse and import the project to the Eclipse workspace

### Prepare the database

Play apps can run with in memory databases, but I want to start with a more real example using mysql as database, the first thing to do is to create the database and the user that your application will use to store the data, I put together this script that I save in the docs folder of the project.

```
CREATE USER 'beepify\_user\_db'@'%' IDENTIFIED BY 'fFRbUPmS9BSHggR';
CREATE USER 'beepify\_user\_db'@'localhost' IDENTIFIED BY 'fFRbUPmS9BSHggR';

CREATE SCHEMA db\_beepify DEFAULT CHARACTER SET utf8;
GRANT ALL PRIVILEGES ON db\_beepify.\* TO 'beepify\_user\_db'@'%' WITH GRANT OPTION;
GRANT ALL PRIVILEGES ON db\_beepify.\* TO 'beepify\_user\_db'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

You can generate nice and [random passwords here](https://www.random.org/passwords/?num=5&len=15&format=html&rnd=new), and the above username, database names and passwords are not the ones I used :)

Change play configuration to use the new database

**change to match the following at conf/application.conf**

```
play.db {
  config = "db"
  default = "default"
... 

db {
    default.driver=com.mysql.jdbc.Driver
    default.url="jdbc:mysql://127.0.0.1:3306/db\_beepify?characterEncoding=UTF-8"
    default.username=beepify\_user\_db
    default.password="fFRbUPmS9BSHggR"

...
```

ebean.default="models.\*"

> Important comment on this configuration: ebean.default="models.\*" will indicate to Ebean framework, that is the ORM framework installed by default with Playframework, that the package models is used to store all of your models

**add to build.sbt**

```
libraryDependencies += "mysql" % "mysql-connector-java" % "5.1.36"
libraryDependencies += evolutions

enablePlugins(PlayEbean)
```

With these configurations in place you are ready to build the application.