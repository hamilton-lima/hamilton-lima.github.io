---
title: 'Manage your machine with Jenkins'
date: Tue, 20 Dec 2016 19:40:32 +0000
draft: false
tags: ['automation', 'backup', 'jenkins', 'Uncategorized']
---

On these days when Zuckberg is feeling like iron man, [@see building Jarvis](https://www.facebook.com/notes/mark-zuckerberg/building-jarvis/10154361492931634), I want to share a more modest automation :) Manage your machine with [Jenkins](https://jenkins.io/) the powerfull build tool. Jenkins was created to automate builds and any sort of workflows after developers send their code to the server, so it's able to automate virtually anything, and that what I did used it on my own machine to automate some tasks:

*   Remove some help files that the login script creates every single day on my desktop (yes I don't like any icons on my desktop)
*   Make a [differential backup](https://en.wikipedia.org/wiki/Differential_backup) from my documents folder

The steps are pretty simple: Install Jenkins, add projects & test, schedule, done.

### The installation

Jenkins installs as a service at windows, using only 4Mb of memory, starts and stops in some seconds, and offers a nice web admin interface. Just follow the install steps, copy the token data from the install file, and you are good to go. As we will use [apache ant](http://ant.apache.org/) to write our scripts we will need to install it as well. Just [download the the zip version](http://ant.apache.org/bindownload.cgi), unzip and add the ant **bin** folder to the path. mine is at : C:\\app\\apache-ant-1.9.7\\bin

### Add projects

For each of my automation needs I will add a separated Project on Jenkins, and each one will have a build step to run a ant script. So I will use one ant script for each task I want to acomplish. Use the option **add new Item**, choose **FreeStyle project** and click ok. ![](/images/2016/12/jenkins-add-new-job.png) Now we setup what the job will do on **Build** option, choose **Add build Step** and pick **Invoke Ant** option. After that enter the ant file name on the option Build file. ![](/images/2016/12/jenkins-set-ant-build-file.png) This is the ant file I am using for the backup

```
<project name="backup" default="main" basedir=".">
   <description>full backup</description>

   <target name="main">
      <copy todir="g:\\backup2016\\Documents" 
         verbose="yes" failonerror="false" 
         preservelastmodified="yes" force="yes">

         <fileset dir="C:\\Users\\hamilton\\Documents"/>
      </copy>
   </target>

</project>
```
See more details on ant tasks here: [ant.apache.org/manual/tasksoverview.html](https://ant.apache.org/manual/tasksoverview.html)

> If you don't add the **failonerror=false** and the backup starts when you using a file, the entire backup process will fail, using this it will fail only for your file.

After that is saved you can test your job by navigating to the main jenkins interface and clicking to schdule a build for your new job. ![](/images/2016/12/jenkins-schedule-now.png) Every run of the job, will be a link to the job run details, as you can see in the last failure column of the image with #1. When you click on the the details choose the console output option, that will show the results of the ant execution. ![](/images/2016/12/jenkins-console-output.png)

### Schedule your jobs

Click on the job, choose the **Configure** option and **Buid triggers**, after that select **Build periodically**, and enter the magic crontab style string. If you click on the little help button there is very detailed help on how to write the scheduling string, for each hour I am using:
```
H \* \* \* \*
```

Just save and you good to go.