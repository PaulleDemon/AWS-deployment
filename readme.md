# Deploying a web application to AWS: Django, Django Channels

Deploying your project to AWS can be difficult and frustrating if you are new to deploying a project. Neither is AWS user interface easy to work with nor is their documentation easy to understand.

I recently had to redeploy a Django application that made use of WebSockets and it took me around 8 days to get it properly up and running. Most of the errors occurred because I didn't understand their system well.

In this blog, I wish not only to help you deploy your Django project but also to help you understand AWS better. I will also talk about some of the common errors that I faced during the deployment.

While I'll be using Django, most of the steps followed will remain the same across different frameworks and programming languages.

>Note: If think some steps are missing or is incorrect please create a new issue on this GitHub repository or create a new pull request.

> **Note:** Since the release of Amazon Linux 2023 (python 3.9 and above), amazon has removed some of the packages, you don't need those packages any longer. So can skip those in the configuration files. 

<hr>

## Help me deploy to AWS?
Since many people have been asking me to help them in deployment, I have created it as a service, If you want me to help you deploy Django channels, Please Create a [discussion](https://github.com/PaulleDemon/AWS-deployment/discussions) or
Send a message on the [Discord server](https://discord.gg/VtyfrAhD), I'll DM you there, (Standard charges apply)

**Note**: 
    For me to help you will have to provide me necessary permission through IAM access, if your company policy doesn't allow you to provide access, I have also created a video recording, you can check it out [here](https://www.buymeacoffee.com/artpaul/e/241689)

<hr>

### Table of contents:

1. [Introduction to AWS interface](https://github.com/PaulleDemon/AWS-deployment/blob/master/AWS_Interface.md)

    1. Introduction
    2. Commonly used services
    3. Some abbreviation you should know
    4. Navigating in AWS Console
        <br>
        1. Search
        2. Elastic beanstalk
        3. EC2
        4. RDS 
        5. S3

2. [Deploying our Django application](https://github.com/PaulleDemon/AWS-deployment/blob/master/deploying_django.md)
    
    1. Deployment flow
    2. Multiple ways to deploy an application
    3. Setup
    4. Creating an application using the AWS console.
    5. Using EB CLI to deploy an application
        <br>
        1. Creating an application
        2. Creating an environment
    6. Debugging errors


3. [Using RDS to create database](https://github.com/PaulleDemon/AWS-deployment/blob/master/connecting_RDS.md)

    1. Introduction
    2. setting up RDS
    3. Connecting to pgAdmin
    4. Setting up RDS in our project
    5. Testing connection on an EC2 instance (DEBUGGING)
    6. Migrations


4. [Using Environment variables to hide our sensitive info's](https://github.com/PaulleDemon/AWS-deployment/blob/master/UsingEnvironment.md)

    1. Introduction.
    2. Hiding your sensitive information in development
    3. Setting environment variables in production:


5. [Using S3 storage to store static and media files.](https://github.com/PaulleDemon/AWS-deployment/blob/master/s3buckets.md)
    1. Introduction.
    2. Setting up a project for S3 and creating a bucket.
    3. Creating IAM Role


6. [Redirecting Http to Https](https://github.com/PaulleDemon/AWS-deployment/blob/master/redirectHttps.md)

7. [Deploying Django channels application](https://github.com/PaulleDemon/AWS-deployment/blob/master/django-channels.md)

    1. Setup
    2. Connecting Redis:


8. [Using Route53 to connect to custom domain](https://github.com/PaulleDemon/AWS-deployment/blob/master/Route53.md)

    1. Connecting your custom domain to elasticbeanstalk using Route53.
    2. Configuring your custom domain email to send mails 
    3. Hosting a static website


9. [some debugging tips](https://github.com/PaulleDemon/AWS-deployment/blob/master/debugging-tips.md)
    1. Introduction
    2. Checklist
    3. Debugging tips
    4. Commonly used commands.
        1. Eb cli commands
        2. Linux commands

### Additional information:

1. [Security Groups](https://github.com/PaulleDemon/AWS-deployment/blob/master/SecurityGroups.md)
2. [IAM](https://github.com/PaulleDemon/AWS-deployment/blob/master/IAM.md) 

<hr>

References:
1. https://docs.aws.amazon.com/
2. StackOverflow answers
3. https://blog.zishanahmad.dev/how-to-deploy-django-channels-2x-on-aws-elastic-beanstalk-amazon-linux-2#comments-list
4. https://medium.com/@elspanishgeek/how-to-deploy-django-channels-2-x-on-aws-elastic-beanstalk-8621771d4ff0


**Support open-source:**

Like the article? consider a one-time donation

[Donate](https://www.buymeacoffee.com/ArtPaul)

[<img src="https://github.com/PaulleDemon/PaulleDemon/blob/main/images/buy-me-coffee.png" height="100px" width="350px">](https://www.buymeacoffee.com/ArtPaul)


**Keywords**
1. How to deploy Django to AWS?
2. How to deploy Django Channels to AWS?
3. How to deploy Django to cloud?
4. AWS deployment help.
5. Redis connection for Django.
6. How to host Django WebSockets?
7. AWS Deployment help
8. Django AWS deployment help
9. Django Channels AWS deployment help
