# Using S3(Simple storage service) to save media files and other files:

If your users are going to be uploading images such as profile etc, you can also serve static files from S3 but in this we will focus on how to serve media files. You should definitely use S3 to store them and serve them directly from S3 instead of django serving those files.

### To set up S3 follow the below instructions:

1. Install the following pacakges and put them in `requirements.txt` file:
```
pip install boto3
pip install django-storages
```

`requirements.txt`
```
boto3==1.20.26
django-storages==1.12.3
```

add storages in installed apps:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    'storages', # <----- here   
    'my-app'
]
```

django-storages is a library to manage storage backends like Amazon S3, OneDrive etc.

boto3 is a public API client to access the AWS resources such as AWS S3. 

2. Go to S3 by searching for it on AWS console:

3. Click on create bucket

![Create-bucket](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/S3/S3-buckets-create.jpg)

4. Now give the bucket a name of your choice and uncheck block all public access. (make sure the region you are in is the same as in the EC2 instance)

5. Now scroll down and click on create bucket.

### Creating IAM Role:

You ideally don't want your django-app to be able to access all the services by AWS for security reasons. So you create an account which has access only to the S3 buckets from IAM role.

Now search for IAM in the searchbar:

1.Click on Users from the left nav-bar and click on "Add users".

![IAM](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/S3/IAM-users.jpg)

2. Type in a user name and click on Access-key: programmatic access:
![programmatic-access](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/S3/programmatic-access.jpg)

3. Click on next and and under "Add user to group"
click on create group and search for S3 full access, check the check-box.
![Create-group](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/S3/create-group.jpg)

4. Click on review and if everything seems correct then you can click on create user.

5. Take note of all the information including User, Access Key and the Secret Access key. We will be using this to give django permisson to access the S3 buckets.


Now in your `settings.py` add the below(you can add this anywhere in your settings I'll be adding it at the bottom):

```python
AWS_ACCESS_KEY_ID = os.environ.get('AWS_ACCESS_KEY_ID')
AWS_SECRET_ACCESS_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')
AWS_STORAGE_BUCKET_NAME = os.environ.get('AWS_STORAGE_BUCKET_NAME')

AWS_S3_REGION_NAME = os.environ.get('AWS_S3_Region')
AWS_QUERYSTRING_AUTH = False
AWS_S3_CUSTOM_DOMAIN = f'{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com'  
AWS_S3_OBJECT_PARAMETERS = {'CacheControl': 'max-age=86400'}
AWS_DEFAULT_ACL = 'public-read'
```

You should add the access key id etc in `.env` file and in production you should add them as environment variables.(read the Hiding sensitive information topic).

Now in the same directory as `settings.py` create a file called as `storages.py` and add the following.

```py
from storages.backends.s3boto3 import S3Boto3Storage

class MediaStore(S3Boto3Storage):
    location = 'media'
    file_overwrite = False
```

now back in `settings.py` add 

```py
DEFAULT_FILE_STORAGE = '<project_name>.storage.MediaStore'
```
replace `<project_name>` with your project name.
The above will work if the storages.py is in the same directory as `settings.py` If you have it somewhere else provide the path to that eg: `app.storages.MediaStore`.

Thats it should now be up and running.
