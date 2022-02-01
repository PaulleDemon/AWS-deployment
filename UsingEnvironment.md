# Setting you allowed host and other sensitive information in environment variables.

If you are working in production or even in development you don't want to expose your host name, allowed hosts, passwords etc, by placing them inside your settings.

## Hiding your sensitive information in development

In development you will create a `.env` file where you will store all the important passowrds, endpoints to database etc. You will add this file to `.gitignore` and will not commit this file or put this up in a public repository.

The `.env` file should look something like the below
```
RDS_HOST=xyz.region-xyq.aws.database.com
RDS_PASSWORD=xyzwie129
RDS_PORT=5432

AWS_KEY=wewefwef
AWS_SECRET_KEY=wefwfef
```
You can change the names of the keys and values as per your requirements

> Note: This `.env` file should be in the same directory level as other app directories.
eg:
>```
>.
>└── sampleproject
>    ├── .ebextensions
>    ├── .env   <-------- here
>    ├── app1
>    ├── sampleproject
>    │   ├── __init__.py
>    │   ├── settings.py
>    │   ├── urls.py
>    │   ├── wsgi.py
>    │   └── asgi.py
>    ├── app2
>    └── ...
>```

Now to use go to settings.py copy all the sensitive information such as RDS Host, password, username etc, and add it to `.env` file in the above shown format.

Now use pip install and install `django-dotenv` and add it to `requirements.txt`
```
pip install django-dotenv
```

Now open manage.py file and add the following line under main `dotenv.read_dotenv()` dont forget to import dotenv

```python
import os
import sys
import dotenv

def main():
    """Run administrative tasks."""
    dotenv.read_dotenv() # <----------------- here
    os.environ.setdefault('DJANGO_SETTINGS_MODULE', '<myprojectsetings>.settings')
    try:
        from django.core.management import execute_from_command_line
    ...
```

now go to settings and replace all the sensitive information with `os.getenv(KEY)` or `os.environ.get(KEY)`. `os.environ` return a dictonary of key-value pairs `os.getenv` is simply a wrapper around `os.environ.get(KEY)`  Here key is the name you assigned on the Left hand side of the = in `.env` file

.env
```
RDS_HOST=aws.wedkwed.com
```
So here RDS_HOST is your key.

## Setting environment variables in production:

When you are uploading to production you are less likely to upload your `.env` file, you will instead set your RDS_HOST, RDS_PASSWORD and other sensitive information as environment variable. 

There are multiple ways to set the envinronment variable. I'll be showing you two ways.

### 1. First Way

The first way is to set environment variable directly in the EC2 instance. 
You can do so by logging into to your EC2 instance using `eb ssh`

Then you can set the environment variables using the linux commands.
Eg:
```sh
export Key1=value1 Key2=value2 KeyN=valueN
```

I personally haven't used this method myself.

### 2. Second Way:

GO to ElasticBeanstalk on AWS console -> Configurations -> Software -> Edit

Now scroll down until you see "Environment properties":

![Environment properties](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/environment-properties.jpg)

Now you can start filling the name and value as I have shown above. (Please use your own host name, password etc)

Once you have set all the information you can click on Apply.

It will take some time to apply. If everything went right your status should be ok. If something went wrong during the process of updating. It will reset back to previous configuration. You can check the events under Recent events.


You can also set this using EB CLI using the below commmand.
```
 eb setenv Key1=value1 Key2=value2 KeyN=valueN
```
### Note:

If you are using the second method its likely that your environment variables are not set as you expect.

So add the below code in your `settings.py`:

```python
import os
import ast
import subprocess
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

...

# SECURITY WARNING: don't run with debug turned to True in production!
DEBUG = False

def get_environ_vars(): # This is because the environment variables are set in this path in elasticbeanstalk
    # This method returns a dictionary of key value pairs
    completed_process = subprocess.run(
        ['/opt/elasticbeanstalk/bin/get-config', 'environment'],
        stdout=subprocess.PIPE,
        text=True,
        check=True
    )

    return ast.literal_eval(completed_process.stdout)


ENV_VARS = None

if not DEBUG:
    
    ENV_VARS = get_environ_vars()

...
```
Source for the above code: https://stackoverflow.com/a/64528198/15993687

> Note: `get_enviton_vars()` returns dictinoary of key value pairs so you can use any dictinoary methods such as `.get()` on it

Example usage in `settings.py`:

```python
DATABASES = {
>            'default': {
>                'ENGINE': 'django.db.backends.postgresql',
>                'NAME': 'postgres',
>                'USER': ENV_VARS['USER'],
>                'PASSWORD': ENV_VARS['RDS_PASSWORD],
>                'HOST': ENV_VARS['RDS_HOST'],
>                'PORT': ENV_VARS['RDS_PORT']
>            }
>        }

```

> Note: The above is needed only if the environment variable is not set as expected. Otherwise use `os.getenv()`
