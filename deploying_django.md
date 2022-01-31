# Deploying Django to AWS using ElasticBeanstalk:

There are many services provided by AWS and it can become challenging in the beginning to figure out which of them to use. So to make deploying web application easier AWS put together some of the common services that is used while deploying web app and brought it under Elastic beanstalk.

When using Elastic beanstalk we first create an Application in Elastic beanstalk, upload our project in the form of an application bundle, the Elastic beanstalk then takes care of creating our environment and creates configurations required to run our code.

We will be using the same to deploy our web application.

## There are multiple ways to create an application (some are stated below): 
1. Using the AWS console.
2. Using the command line interface (recommended)

Before proceeding I suggest you download EB CLI by following the instruction from here: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-advanced.html

This will help us run important commands in future. 


Before you deploy I am assuming your project directory structure looks like something shown below:

```
.
└── sampleproject
    ├── app1
    ├── app2
    ├── sampleproject
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   ├── wsgi.py
    │   └── asgi.py
    ├── app3
    └── ...
```

### Setup:

Before deploying there are some more things we need to add to our project.

Create a directory named `.ebextensions` inside our project folder and add the following script inside the `.ebextensions` folder. You can name the file as `django.config` as an example. (note it should have .config extension)
```
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: sampleproject.wsgi:application
```
Replace the sampleproject with the name of your project. 

This setting, WSGIPath, specifies the location of the WSGI script that Elastic Beanstalk uses to start your application.

Now if you haven't created a `requirements.txt` create it and use `pip freeze > requirements.txt` to copy the package names from your local virtual environment. Elastic beanstalk uses this file to determine which packages to install on EC2 instance running your application

So now your directory should look like this:

```
.
└── sampleproject
    ├── .ebextensions
    │   └── django.config
    ├── app1
    ├── app2
    ├── sampleproject
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   ├── wsgi.py
    │   └── asgi.py
    ├── app3
    └── .gitignore
```
> Note you can choose to add a file called `.ebignore` in which you can type in all the files and folders that you don't want to be uploaded when using command line interface to upload. If you have `.gitignore` you won't require `.ebignore`, but if you do add `.ebignore` your .gitignore won't be read and only the files and directories specifed in `.ebignore` will be ignored.

## 1. Creating application using the aws console.
This is one of the easiest way to set up your django project up and running.

Go to AWS console and search for ElasticBeanstalk:

![Create Application](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/CreateEBApplication.jpg)

1. Click on "Create a new application" on the top right hand corner next to actions.

![Type application name](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/createEbApp2.jpg)
2. Type in an application name and description(optional) and then click on create.

I'll name it sample application and give a sample description.

Thats it your new application has been created.

But its not done yet. You would need to create an environment to deploy and run our project. 

![Create environment](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/create-environment.jpg)

3. Now click on "create a new environment". You will be presented with the below screen.

![select env tire](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/select-env-tire.jpg)

4. Choose Web server environment and click on select.
5. You will now be presented with the following screen.

![Environment info](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/environment-info.jpg)

Here you can change the name of the environment or you can keep the default. You can create as many environments as you want but all their names must be unique.

You can choose to enter a Domain or leave it blank, if you leave it blank the name will be auto-generated.

Now under the platform section choose Python other dropdowns will be auto-filled. 

You can now choose to upload your code by clicking on upload your code and choose the project from your local computer(max upload size: 512MB).

Or you can even choose to leave that and later upload the code thorugh ebcli.

6. Now click on "configure more options". 

**Note: If you are deploying django application that uses channels for websockets you need to pay close attention to the below.**

Now you will be presented with the below screen:

![Configuring environment](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/configuring-eb-env.jpg)

7. under presets click on custom. Then under Load balancer section click on edit.

If you are not using channels or websockets. You can go ahead with classic load balancer. 

**Webscockets**:
If you are using websockets click on Application load balancer. Only Application load balancer allows websocket connections. 

Now go to the bottom of the page and click on save.


> Note: you will not be able to change the load balancer later. So if you are using websockets use Application load balancer. 

>Note: If you ever require to change the load balancer later you will have to create a new environment and copy your existing environment settings.


> Note: Don't click on edit database under Database section, as we will be creating database from RDS console. If you are ever required to modify database settings you will find it difficult to modify. So don't create the data base when creating the environment.

8. Now if you are happy with the settings you can click in create environment at the bottom of the page.

> This operation can take several minutes so please wait till the oprtation is complete.
<<<<<<< HEAD

## Using EB CLI (Recommended way):
=======
>>>>>>> be5058702b7c3686cd988e20c8d1971176ec420e
