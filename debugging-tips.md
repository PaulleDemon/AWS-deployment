# Debugging tips:

There is high chance that you man not be able to get your application running the first time you deploy it. So here are some of the debugging tips you can use to understand the cause of your error.

### Check list

1. Ensure you have not added `.ebextensions` and `requirements.txt` to `.gitignore`

2. Check if all the endpoints to RDS, S3 are correct.

### debugging tips:

1. Check the logs in the elastic beanstalk.
![Logs](images\logs.jpg). 
You would usually need to download only the last 100 lines. In logs, Most likely your errors would lie under the `/var/log/web.stdout.log`.

If you get `ModuleNotFoundError: No module named 'application'` then its likely your wsgi path was not set correctly during deployment. Try to stage changes using `git add .` and redeploy the staged changes `eb deploy --staged`

If you get `ModuleNotFoundError: No module named 'xyz'` make sure you have added the module xyz in the `requirments.txt`(here xyz is a placeholder, this isn't and actual library).

Check if your database is connected by connecting to Ec2 instance using `eb shh`
```sh
eb ssh
Enter passphrase:

-------------- Elastic beanstalk-----------------
source /var/app/venv/*/bin/activate
cd /var/app/current/
python manage.py check --database default
```
This should  result in `System check identified no issues (0 silenced).` if there is no connection problem.


## Some commonly used commands:

### 1. EB CLI COMMANDS:

Things in square brackets are optional.
| Command                               | description                                                                                      | usage                                          |
|---------------------------------------|--------------------------------------------------------------------------------------------------|------------------------------------------------|
| eb init [-p platform applicationname] | sets default values for Eastic Beanstalk application prompting you with series of questions      | eb init, eb init -p python-3.8 ApplicationName |
| eb create [environmnet name]          | creates an environment                                                                           | eb create, eb create sample-env                |
| eb ssh --setup                        | change or create key pair(ssh) assigned to an instance                                           | eb ssh --setup                                 |
| eb ssh [environment name]             | Connects to Amazon Linux EC2 instance in your environment using SSH                              | eb ssh, eb ssh sample-env                      |
| eb deploy                             | deploys the application bundle from the initialized project directory to the running application | eb deploy                                      |
| eb deploy --staged                    | deploy staged changes                                                                            | eb deploy --staged                             |


### Some linux commands:

| Command                                                 | description                                                                         | usage                                      |
|---------------------------------------------------------|-------------------------------------------------------------------------------------|--------------------------------------------|
| cd                                                      | change directory                                                                    | cd /var/app/current/                       |
| ls                                                      | list directories and files under a specified directory                              | ls /var/app/current/                       |
| source                                                  | reads and executes the contents of a file                                           | source /var/app/venv/*/bin/activate        |
| exit                                                    | exit the shell where its running currently                                          | exit                                       |
| netstat -antpl                                          | returns listening ports along with application                                      | netstat -antpl                             |
| sudo                                                    | helps you run programs with security privileges of another user                     | sudo netstat -antpl                        |
| lsof -i:portno                                          | returns the application thats running in specific port                              | sudo lsof -i:5000                          |
| psql -h [HOST] -p[PORTNO] -U[USER NAME] [DATABASE NAME] | connect to postgres SQL (this command is available only if you have installed psql) | psql -h localhost -p 5432 -U user postgres |

Running daphne:
```
daphne -b 0.0.0.0 -p 5000 project.asgi:application
```

Checking database connection in python using manage.py:
```
python manage.py check --database default
```