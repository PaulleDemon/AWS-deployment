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


### Some usefule commands:
