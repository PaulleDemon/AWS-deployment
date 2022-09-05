# Deploying Django to AWS using ElasticBeanstalk (EB):

There are many services provided by AWS and it can become challenging, in the beginning, to figure out which of them to use. So to make deploying web applications easier AWS put together some of the common services that are used while deploying a web app and brought it under Elastic beanstalk.

When using Elastic beanstalk we first create an Application in Elastic beanstalk, upload our project in the form of an application bundle, the Elastic beanstalk then takes care of creating our environment and creates configurations required to run our code.

We will be using the same to deploy our web application.

> Note: Before you move on please set up an Access by logging in as root user from the IAM console. This will be used when you are using EB CLI to allow EB CLI (command line interface) to manage the access. You can follow this guide to set this up: https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys
> or you can refer to any other tutorial.

## Deployment flow:

1. Create an Application in Elastic beanstalk.
2. Create an environment under the above-created application.
3. Upload your application.

## There are multiple ways to deploy an application (some are stated below): 
1. Using the AWS console.
2. Using the command-line interface (recommended)

Before proceeding I suggest you download EB CLI by following the instruction from here: https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-advanced.html

This will help us run important commands in the future.

I'll be using eb-cli to deploy but will also show you how to use AWS console,

The project I'll be deploying is called PhotoApp. It is basically a simple web app that allows users to upload photos.

Here is the link to that project you can refer to this at later point in time: [PhotoApp](https://github.com/PaulleDemon/DjangoPhotoApp)

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

### Setup(Common for both AWS console and eb-cli):

Before deploying there are some more things we need to add to our project.

Create a directory named `.ebextensions` inside our project folder and add the following script inside the `.ebextensions` folder. You can name the file as `django.config` as an example. (note it should have .config extension)
```
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: sampleproject.wsgi:application
```
Replace the sampleproject with the name of your project. 

This setting, WSGIPath, specifies the location of the WSGI script that Elastic Beanstalk uses to start your application.

Now if you haven't created a `requirements.txt` create it and use `pip freeze > requirements.txt` to copy the package names from your local virtual environment. Elastic Beanstalk uses this file to determine which packages to install on EC2 instance running your application

Make sure that these are not added to `.gitignore`

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
    ├── requirements.txt
    └── .gitignore
```
**Setting up static files for production:**
As soon as you set `DEBUG=False`, django will no longer serve your static files(css, js files). So before you send it to AWS there are a couple of things you need to configure in settings.py.

1. set `STATIC_ROOT = BASE_DIR.joinpath('static')`
2. set `STATIC_URL = '/static/'`
3. set `STATICFILES_DIRS = []`, in the list mention all the paths to your static files that are outside static folder in your apps; as by default django looks into static folder inside your django app you don't need to mention it in this list. 

Whats happening? Well all that the above steps does is that django will collect and place all the static files into a folder mentioned in `STATIC_ROOT = BASE_DIR.joinpath('static')`, so that it doesn't have to keep searhing every folder for your static files in your project during production.

Now with that done you still need to add one more thing to `django.config` file
```
aws:elasticbeanstalk:environment:proxy:staticfiles:
    /static: static
    value: static/
```

So now your `django.config` file should look as such.
```
option_settings:
  aws:elasticbeanstalk:environment:proxy:staticfiles:
    /static: static
    value: static/
    
  aws:elasticbeanstalk:container:python:
    WSGIPath: sampleproject.wsgi:application
```


> Note you can choose to add a file called `.ebignore` in which you can type in all the files and folders that you don't want to be uploaded when using the command-line interface to upload. If you have `.gitignore` you won't require `.ebignore`, but if you do add `.ebignore` your `.gitignore` won't be read and only the files and directories specified in `.ebignore` will be ignored.

## 1. Creating an application using the AWS console.
This is one of the easiest way to set your Django project up and running.

Go to AWS console and search for ElasticBeanstalk:

![Create Application](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/CreateEBApplication.jpg)

1. Click on "Create a new application" on the top right-hand corner next to actions.

![Type application name](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/createEbApp2.jpg)
2. Type in an application name and description(optional) and then click on create.

I'll name it sample application and give a sample description.

That's it, your new application has been created.

But it's not done yet. You would need to create an environment to deploy and run our project. 

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

Or you can even choose to leave that and later upload the code through eb-cli.

6. Now click on "configure more options". 

**Note: If you are deploying a Django application that uses channels for WebSockets you need to pay close attention to the below.**

Now you will be presented with the below screen:

![Configuring environment](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/configuring-eb-env.jpg)

7. under presets click on custom. Then under the Load balancer section click on edit.

If you are not using channels or WebSockets. You can go ahead with the classic load balancer. 
<hr>

**Webscockets**:
If you are using WebSockets click on Application load balancer. Only Application load balancer allows WebSocket connections. 

Now go to the bottom of the page and click on save.
 <hr>

> Note: you will not be able to change the load balancer later. So if you are using WebSockets use Application load balancer. 

>Note: If you are ever required to change the load balancer later you will have to create a new environment and copy your existing environment settings.


> Note: Don't click on edit database under Database section, as we will be creating a database from the RDS console. If you are ever required to modify database settings you will find it difficult to modify. So don't create the database when creating the environment.

> Note: for productions, I suggest you add your local (sqlite3/any other) database to `.gitignore` and not upload it EB environment.

8. Now if you are happy with the settings you can click in create environment at the bottom of the page.


## 2. Using EB CLI to deploy the application.


To use this you must have EB CLI installed as stated above and added the path to environment variables. Please follow the steps as provided in the guide to install EB CLI.

### generating access key and configuring AWS CLI:

first go to IAM console by searching for it in the search bar. Now click on users, click on your account name > under summary click on "security credentials" tab > click on create access key.

Now follow the steps in this documentation to configure your account with cli: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html

----

Now open the command prompt or terminal and change the current directory to the project directory.

In Windows
```sh
cd `path_to_project _directory`
```
now type `eb` and EB CLI should pop up. If it doesn't open in a new window, no problem. Make sure you have deactivated the virtual environment you are currently working in when using the EB CLI or EB commands.

### 1. Creating an application:

```sh
eb init
```

or 

```
eb init -p python-3.8 PhotoProject
```
In the above replace the python version and PhotoApp with a version and name of your choice. This just creates an application named PhotoApp in AWS Elastic Beanstalk.

The 2nd option will directly create an application without prompting you for more options (this one is quicker).

>I'll go with 1<sup>st</sup> option `eb init`


If you went with the first option you will be prompted to select a region you can leave this at default or choose a region.

![region-selection](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/eb-init.jpg)

> Note: In the future, You will have to keep all the services such as RDS, S3 under the same region for it to work properly.

I'll leave this at default which is option-3 (US west (Oregon))

Next, you will be asked to choose an application or create a new

```sh
Select an application to use
1) sampleapp
2) [ Create new Application ]
(default is 2): 2
```
select `2`

If you haven't previously set up provided a secret key and access key you will be asked to give it. If you haven't set this up please follow their guide to set up 

Now you will be asked to give a name to the application
```
Enter Application Name
(default is "PhotoProject"):
```
You can choose to give a name or choose to leave it as default.

Now the application is created.

```
Enter Application Name
(default is "PhotoProject"):
Application PhotoProject has been created.

It appears you are using Python. Is this correct?
(Y/n):
Select a platform branch.
1) Python 3.8 running on 64bit Amazon Linux 2
2) Python 3.7 running on 64bit Amazon Linux 2
3) Python 3.6 running on 64bit Amazon Linux (Deprecated)
(default is 1):
```
choose the version, I'll be going with the first option

```
Do you wish to continue with CodeCommit? (Y/n): n
Do you want to set up SSH for your instances?
(Y/n): y

Select a keypair.
1) existing-pair
2) [ Create new KeyPair ]
(default is 1):
```

In the above, we choose `n` for codecommit. You can choose `y`.
>AWS CodeCommit is a fully managed service for securely hosting private Git repositories. CodeCommit now supports pull requests, which allows repository users to review, comment upon, and interactively iterate on code changes. Used as a collaboration tool between team members, pull requests help you to review potential changes to a CodeCommit repository before merging those changes into the repository
> above is taken from docs: https://aws.amazon.com/blogs/devops/using-aws-codecommit-pull-requests-to-request-code-reviews-and-discuss-code/


We choose `y` for SSH
> The Secure Shell Protocol (SSH) is a cryptographic network protocol for operating network services securely over an unsecured network.

We will be using SSH to connect to our EC2 instance so we must give `y`. If you haven't set this you can later set this using `eb ssh --setup`

The output for the above command will look something like this.
```
Type a keypair name.
(Default is aws-eb): <keypairname>
Generating public/private RSA key pair.
Enter passphrase (empty for no passphrase):
Enter the same passphrase again:
Your identification has been saved in C:\path\.ssh\<keypairname>.
Your public key has been saved in  C:\path\.ssh\<keypairname>.pub.
The key fingerprint is:
SHA***:G**4A****9******3ps***
The key's randomart image is:
+---[RSA ***]----+

+----[SHA***]-----+
Enter passphrase:
WARNING: Uploaded SSH public key for "***" into EC2 for region *****.
```
You can choose `\<keypairname\>` of your choice.
Enter a passphrase you can remember. you can also choose to save the passphrase, everything including fingerprint in a file(shouldn't be uploaded to a public repo).

### 2. Creating an environment:

Environment will create an instance of EC2 where your project will run.

To set this up type 
```
eb create app-env
```
You can replace "app-env" with a name of your choice. This simply creates an environment with the specified name and uploads the project zip file to the S3 buckets service (it's a simple storage service provided by AWS). You can also host your media files here and media files will directly be served from here. 

This might take from a few seconds up to a few minutes to complete. After completion, you should see something as shown below at the end.

```
2022-01-31 16:09:00    INFO    Instance deployment successfully generated a 'Procfile'.
2022-01-31 16:09:03    INFO    Instance deployment completed successfully.
2022-01-31 16:10:06    INFO    Successfully launched environment: app-env

```


Now head back to AWS Console and type elastic beanstalk -> left panel Applications -> choose the application you just created in my case it was called PhotoProject.
Now click on the environment in my case it was called app-env.

Now if it was successfully deployed you should see the health ok and a green checkmark.

![eb-successful](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/eb-successful.jpg)

Now What I have highlighted in red is your domain. Click on it and you will see.

![disallowed host](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/disallowed_host.jpg).

Now all you have to do is head back to your settings.py in your project and under
ALLOWED HOST type in the domain.

In my case, it was 'app-env.eba-zprspr6p.us-west-2.elasticbeanstalk.com'
```
DEBUG = True

ALLOWED_HOSTS = ['app-env.eba-zprspr6p.us-west-2.elasticbeanstalk.com']
```
> Note: In production don't forget to set `DEBUG=FALSE`

Now save it again head back to your EB CLI and type 
```sh
eb deploy
```
If you make any changes to your application you can use `eb deploy` to deploy the application.

You can also upload only the staged changes by using
```
eb deploy --staged
```

you can stage changes by using:
```git
git add .
```

Now you should be able to see your application output


### Unsuccessful deployment:

<br>

![eb-unsuccessful](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/degraded-health.jpg)

If you see this that means that there was an error while uploading or the health check is failing with a 4xx error. Follow the debugging errors below to debug the errors
### Debugging errors:

If you see health as degraded you should head over to logs in the left navigation panel.

![Logs](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/logs.jpg)

Here click on request logs and click on last 100 lines.

Now download the logs:

When you view that page:
You should notice the title:
Eg: 
![log-title](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/log-title.jpg)

The above is logs for eb-engine.

Now scroll down till you find 
```
----------------------------------------
/var/log/web.stdout.log
----------------------------------------
```

Under this, you should be able to see what the error is.


If you want to connect to Linux AWS servers use. You can also debug from there:

```
eb ssh
```
You should enter your passphrase and be able to login. You can now type Linux commands 

To activate your environment in Linux-2
```
source /var/app/venv/*/bin/activate
```
to change our project directory use:
```
cd /var/app/current/
```

Note the above paths might change if AWS team decides to change them in the future. You can always find the new paths in the migrations documentation they release.

The above is helpful when you want to find out what's going wrong.
You can also use 
```
ls /var/app/current/
```
to list all the files and folders under the current directory.


Errors:
### ModuleNotFoundError: No module named 'application':

If you see this error that means that django.config file was not uploaded successfully

You can confirm this by heading over to the configuration tab from the left panel.

![configuration](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/configuration.jpg).

Now under software click on edit:

Now under the Container options Section, you can see the WSGI path if WSGI Path contains only application and not "PhotoProject.wsgi:application" then it means that the django.config file was not uploaded correctly.

1. You can try to stage the changes using: 
```
git add .
```
and using 
```
eb deploy --staged
```
2. Make sure you haven't add .ebextensions to git ignore.

If it still doesn't work only as a last resort 
you can head elastic beanstalk configurations -> software -> edit -> WSGI path fill the path like so: PhotoProject.wsgi:application  

### ModuleNotFoundError: No module named 'XYZ':
If you find module not found then make sure you have included them in `requirements.txt`

### Failing health checks:
If you don't see any errors in the log file then head over to the link that was underlined in red in the above image(the image where the health was green).

Click on the link if you can see your app running or the Django's debug error page then it means that your upload was successful 

You should head over to the environment page and check out the recent events. 

There are three types of Type:
1. INFO - it's just info and there are no errors
2. WARN - A warning.
3. ERROR - an error occurred

These types will be visible under the Type column under Recent events section as shown below.

![recent-events](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/recent-events.jpg)

If you don't see any error click on show all.

If under detail you see x% (eg: 50%) of health checks are failing with 4xx. This means that only the health checks are failing.

By default, the health checks are performed to the `/` endpoint. So when AWS makes get requests to `http://myhost/` your application should return 200 Ok or anything in the 200 range.

If your application is not returning 200 then you can create a dedicated health check endpoint. 

Eg:
urls.py 
```
urlpatterns = [
    path('health/', views.health_check),
    ...
]
```

views.py:
```
from django.http import HttpResponse
def health_check(request):
    return HttpResponse(status=200)
```

Now head over to the environment configuration in ealsticbeanstalk -> left panel -> Configuration -> scroll down -> Load balancer.

Now under load balancer click on edit.

Scroll down to health check:

![health-check](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/health-check.jpg)
Now under health check path provide the path to your health check eg:`/health` as shown above.

then scroll down and click on apply and this could take several minutes. You can head over to events and check if the health check is failing
