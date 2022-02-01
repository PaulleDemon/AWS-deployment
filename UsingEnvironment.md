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

Now to use go to settings.py copy all the sen