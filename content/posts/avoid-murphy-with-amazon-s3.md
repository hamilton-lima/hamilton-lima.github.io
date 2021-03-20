---
title: 'Avoid murphy with Amazon S3'
date: Wed, 02 Nov 2016 02:21:15 +0000
draft: false
tags: ['aws', 'aws cli', 'backup', 'create user', 'murphy', 'mysqldump', 's3']
---

![photo-1462045504115-6c1d931f07d1](/images/2016/11/photo-1462045504115-6c1d931f07d1.jpg) Backups are like life insurance, you allways will be sorry if you don´t have when you need it. That's why I am sharing this step by step way to backup your application or site or wherever you have in production.

### Preparing the vault, a.k.a. Amazon S3

I decided to store my backups on Amazon S3 service, because is [cheap](https://aws.amazon.com/s3/pricing/), around USD0,03 per GB, and very simple to store and to retrieve the data. My first step to try to mount an remote amazon s3 storage as a linux folder at the server, lost some hours trying to make it work, but luckly found enlighment from one comment at stackoverflow: DON'T USE IT :) seriously run from it! After been healed from the mount sickness, I just downloaded the [Amazon Webservice command line tools](http://docs.aws.amazon.com/cli/latest/reference/s3/index.html#cli-aws-s3) and everything run smoothly! But as I allways say: Nothing is simple, but everything is possible. The process has several small steps to follow, most of then you only do it once, So this are the steps you need to do at amazon to prepare your backup storage:

*   Create you amazon S3 account - [https://aws.amazon.com/free](https://aws.amazon.com/free)
*   [Create a bucket](https://console.aws.amazon.com/s3/home?region=us-east-1) to save your data
*   [Create a user](https://console.aws.amazon.com/iam/home?region=us-east-1#users) to be access your bucket
*   Save the new user access key and secret, you will need it to setup your server
*   Give the new user the proper rights to your bucket

Most of the steps are just click click and click. Give the proper rights to your new user require more setup. The idea is to [create a policy](https://console.aws.amazon.com/iam/home?region=us-east-1#policies) and attach an user to it, and the policy has a well defined JSON format, yes a JSON format, [with lots and lots of options](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html). You can do cool stuff like filter by IP, give rights during some period of time and several other cool features. I used the most simple rights: right to read the bucket and the right to create objects in the bucket, this is the policy description I used:

```
{
    "Version": "2012-10-17",
    "Statement": \[
        {
            "Action": \[
                "s3:ListBucket",
                "s3:GetBucketLocation"
            \],
            "Effect": "Allow",
            "Resource": "arn:aws:s3:::bucketname"
        },
        {
            "Action": "s3:\*",
            "Effect": "Allow",
            "Resource": "arn:aws:s3:::bucketname/\*"
        }
    \]
}
```

I took me some time to realize that we need two statements in the policy description: Allow ListBucket and GetBucketLocation to the bucket and other to allow all actions to the bucket files, attention to the diference: bucketname vs bucketname/\*

### Installing AWS CLI

The installation process is just install, add the access key and secret and test your setup, here are the steps:
```
apt install awscli
aws configure
cat ~/.aws/credentials
```
After that you can use the aws command line interface, here are some samples:

aws s3 ls s3://bucketname

list the existing files in the bucket

aws s3 cp myfolder s3://bucketname/myfolder --recursive

copy myfolder recursively to s3

see all commands here : [http://docs.aws.amazon.com/cli/latest/reference/s3/index.html#cli-aws-s3](http://docs.aws.amazon.com/cli/latest/reference/s3/index.html#cli-aws-s3)

### The backup script

Now that we have aws installed and the vault ready, let's talk about the backup script, first what if should do:

*   save database dump with the timestamp on the file name
*   copy all the files from the nodejs application
*   copy all the files from wordpress
*   copy all the files from the static content
*   upload the content to Amazon S3

So the following script is doing all of it:
```
#!/bin/sh
# get the current timestamp
NOW="$(date +"%Y%m%d\_%H%M%S\_%N")"
TODAY="$(date +"%Y%m%d")"

# create the backup folder
mkdir /tmp/bkp\_$NOW
mkdir /tmp/$TODAY

# dump database data
mysqldump --complete-insert --no-set-names --host=127.0.0.1 --user=backup\_uzr --password=XXXX wordpress > /tmp/bkp\_$NOW/wordpress.sql
mysqldump --complete-insert --no-set-names --host=127.0.0.1 --user=backup\_uzr --password=XXXX beepify > /tmp/bkp\_$NOW/beepify.sql

# copy folders to backup folder
cp -r /var/www/html /tmp/bkp\_$NOW
cp -r /var/www/portfolio /tmp/bkp\_$NOW
cp -r /var/www/vanhackaton-video-editor /tmp/bkp\_$NOW

# zip each file or folder to zip\_ folder
cd /tmp/bkp\_$NOW/
for dir in $(ls); do tar cvzf /tmp/$TODAY/${dir}\_$NOW.tar.gz ${dir}; done

# copy zipped backup files and folders to amazon S3
aws s3 cp /tmp/$TODAY s3://bucketname/$TODAY --recursive

# remove the temporary folders
echo "removing /tmp/bkp\_$NOW and /tmp/$TODAY"
rm -rf /tmp/bkp\_$NOW
rm -rf /tmp/$TODAY
```

Now let's comment some important parts of this script:
```
TODAY="$(date +"%Y%m%d")"
```

It creates an environment variable with the result of the execution of the command date with the Ymd format, the equivalent is done on NOW.
```
mysqldump --complete-insert --no-set-names --host=127.0.0.1 --user=backup\_uzr --password=XXXX wordpress > /tmp/bkp\_$NOW/wordpress.sql
```
Note that we are using the password in the command line, so there one security issue here, in order to minimize this I created an read only user at the mysql database, with the following commands, that should be executed at msyql console

```
GRANT SELECT, LOCK TABLES ON \*.\* TO backup\_uzr@127.0.0.1 IDENTIFIED BY 'XXXX'; 
GRANT SELECT, LOCK TABLES ON \*.\* TO backup\_uzr@localhost IDENTIFIED BY 'XXXX'; 
flush privileges;
```

After all the files are copied in different folders at /tmp/bkp\_$NOW we run this magic command
```
for dir in $(ls); do tar cvzf /tmp/$TODAY/${dir}\_$NOW.tar.gz ${dir}; done
```

That can be split as follow:

*   iterate over the result list from the command ls
*   for each element on that list set the value to the variable dir
*   execute a tar command creating a tar.gz file at the /tmp/$TODAY folder zipping the content of the variable dir

After that run the aws cli command to push the results to the S3 In order to schedule the script use the command: crontab -e and add the line to run every dy at 04:42 AM
```
42 4 \* \* \* /root/my-nice-backup-script.sh
```
Have fun backuping up your sites!