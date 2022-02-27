# Connecting to RDS(Relational Database Service):

If you are going to be working in production you definitely want to ditch the SQLite database that you have been working with and use a database such as Postgre SQL or MySQL.

RDS provides you with a simple and easy setup to connect to an external database. This database won't be inside our project directory but outside our application. We will connect to the database using the endpoint that is provided by AWS.

We will be using the AWS console to set up our RDS.
 
## Setting up RDS:

Now go to AWS console and search for RDS

If you are currently in the RDS dashboard(see the left nav panel)

Before creating a Database make sure you are in the same region as the region you selected when creating your application using `eb init` or it's in the same region as the environment region.
![region](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/RDS/region.jpg)

In my case it's Us-west-2(oregon). If it's not in the same region click on the drop-down select the correct region.

you can scroll down to Create database and click on create database.

![dashboard-create](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/RDS/rds-dashboard-create.jpg)

If you are currently in databases you can simply click on create database if you see that.

![Create database](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/RDS/create-database.jpg)

1. Choose Standard create under database creation method.
2. Choose your Engine type, I'll be going with postgresSQL 12.8.
3. Choose Template. I'll be choosing free tire. You can choose production or dev/test if you don't have free tire(free database usage for 1st year).

Scroll down to settings.
![settings](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/RDS/db-settings.jpg)

Fill in database identifier. You can choose a name of your choice. 

Choose a password and username you can remember. I'll be leaving the username as the default(Postgres).

4. Now scroll down and under connectivity section -> Public access -> click on Yes.

You won't ideally need to keep public access to yes. But if you want to access your database from your computer or from PgAdmin on your computer, you will have to keep this yes. You can change this later using [security groups](https://github.com/PaulleDemon/AWS-deployment/blob/master/SecurityGroups.md). 

Don't change anything else. You can scroll down and click on create-database. Your database creation can take a few minutes to create.

Once created the status will be changed to available(view this in databases panel).

Now click on your database and copy the endpoint

![Endpoint](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/RDS/database-endpoint.jpg)

Don't share this endpoint.

### Connecting to pgAdmin:

You can skip this if you don't want to access your database through pg-admin.

Open your pgAdmin, I am using pgAdmin4.

Now in pg-admin:

1. object(on the top header) -> create -> server group -> give a name for the server group.
2. Right-click on your server group -> create -> server.
3. A create server dialog should pop up. Specify a name for the server, then in the same dialog click on "connection" tab on the top.
4. Now paste the endpoint you copied to the hostname/address. Give the user name and password you created when creating the database. (Make sure you are in port 5432. If you had chosen another port when creating the database then change the port in the pgadmin too).
5. That's it you now have your database connected to pgadmin. You can now view your tables directly from here.

### Errors:
If you get timeout errors even after having a proper internet connection. It's likely you haven't given public access database. You can change this later in the security group of the database.


> Note: Public access to the database is quite insecure. If someone has your endpoint, username and password. They will be able to connect to your database from any part of the world. We will later update the security group to allow access only to particular IP addresses in [`security groups`](https://github.com/PaulleDemon/AWS-deployment/blob/master/SecurityGroups.md).

Now that your pgadmin is up and running lets now setup the database in our project:

## Setting up RDS in our project:

First add `psycopg2-binary==2.9.2` to `requirements.txt`.

Open your settings.py and under databases change the below accordingly.

```python
DATABASES = {
            'default': {
                'ENGINE': 'django.db.backends.postgresql',
                'NAME': 'postgres',
                'USER': '<Username>',
                'PASSWORD': '<Password>',
                'HOST': '<Host>',
                'PORT': '5432'
            }
        }
```
Fill in the username password and host. Please don't commit this to pubic repositories yet. There are other ways to hide your password, host and port using `.env` file which we will be discussing later.

> Tip: If you want to continue using SQLite as your devlopment database. You can try something as shown below in settings.py
>```python
> if DEBUG:
>    DATABASES = {
>        'default': {
>            'ENGINE': 'django.db.backends.sqlite3',
>            'NAME': BASE_DIR / 'db.sqlite3',
>        }
>    }
> else:
>    DATABASES = {
>            'default': {
>                'ENGINE': 'django.db.backends.postgresql',
>                'NAME': 'postgres',
>                'USER': '<Username>',
>                'PASSWORD': '<Password>',
>                'HOST': '<Host>',
>                'PORT': '5432'
>            }
>        }
>```

That's it now you can save the `settings.py` and in your EB CLI `eb deploy` (make sure your eb cli's path is in the project directory eg: `(eb-env) c:\\path\PhotoProject>`)


## Testing connection on EC2 instance (DEBUGGING):

If you ever want to check if your project is being able to connect to the database.

using eb cli connect to your EC2 instance using

```
eb ssh
```
It will then ask for a passphrase. Give the passphrase you set up during ssh creation. If you didn't set up ssh you can create it using `eb ssh --setup`.

now navigate to your project directory in the Ec2 using:
```
cd /var/app/current/
```

activate your virtual environment using
```
source /var/app/venv/*/bin/activate
```

> Note: these paths might change in the future if AWS team decides to change them.

Now to check if it is connected just use: 
```
python manage.py check --database default
```

If its successful you should get 
```
System check identified no issues (0 silenced).
```

### Errors

If you get 
```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
psycopg2.OperationalError: could not connect to server: No such file or directory
    Is the server running locally and accepting
    connections on Unix domain socket "/var/run/postgresql/.s.PGSQL.5432"?
``` 
It means that the database was not added correctly or you haven't added hostname. Try rechecking the hostname.

In python, you can check the hostname by using the below code in your shell

```python
>>> from django.conf import settings
>>> settings.DATABASES['default']['HOST']
>>> 'awed.aws.com'
>>> settings.DATABASES['default']['PORT']  
>>> '5432'  
```


## Migrations:

If had set public access you should be simply able to run :
```
python manage.py migrate
```
directly from your development environment. (make sure if you have set the right database in `settings.py`) 

You can also log in to EC2 instance using ssh and perform migration(not recommended).

If the database is not accessible from your development environment follow the recommended way for migration as stated below.

### Recommened migration steps:

In your `.ebextensions` folder create a file called `db-migrate.config`(the name could be whatever you want).

Then paste the following command
```
container_commands:
  01_migrate:
    command: "source /var/app/venv/*/bin/activate && python3 manage.py migrate"
    leader_only: true
option_settings:
  aws:elasticbeanstalk:application:environment:
    DJANGO_SETTINGS_MODULE: PhotoProject.settings
```

In the above replace the `PhotoProject` with your application name where the settings.py resides.

This will run `python manage.py migrate` every time you deploy on EC2 instance.


### Additional information for django users.

loaddata and dumpdata can be useful when you need to quickly load and unload data from your database from a specifed format such as JSON, XML etc. 

**dump data**
If you ever need to serialize the data from the database to JSON or any other format you can use the following commands provided by django

`python -Xutf8 manage.py dumpdata > mydata.json --indent 4`

The above command will serialize all the data from the database to JSON format with 4 indent, the Xutf8 specifies the encoding.

You can also be more specific on which apps you would like to serialize by specifying the appe name eg:

`python -Xutf8 manage.py dumpdata app1 > mydata.json --indent 4`

The above will serialize models only from the app1.

> Note: If you get the error `CommandError: Unable to serialize database: cursor "_django_curs_xxxxxx_66" does not exist` it usually means that the table in the database and model doesn't match.
> Django uses your models to serialize data, so if you had made any changes to your model revert back and the dumpdata again.

**Load data**

You can use the below command to provide an initial data to the database.

`python manage.py loaddata mydata.json`

The loaddata will deserialize the data and insert it to your database.

>Note: If you already have some data in your data base, then make sure to comment out post_save methods, if you are using that to create additional fields. Otherwise it might sometimes create duplicate elements and your loaddata will fail.


Additional reference:

`dumpdata`: https://docs.djangoproject.com/en/4.0/ref/django-admin/#dumpdata

`loaddata`: https://docs.djangoproject.com/en/4.0/ref/django-admin/#loaddata