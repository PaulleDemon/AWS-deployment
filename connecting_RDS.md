# Connecting to RDS(Relational Database Service):

If you are going to be working in production you definitely want ditch SQLite database that you have been working with and use a database such as Postgre SQL or MySQL.

RDS provides you with simple and easy setup to connect to an external database. This database won't be inside out ptoject directory but outside our application. We will connect to the database using the endpoint that is provided by the AWS.

We will be using the AWS console to set up our RDS.
 
## Setting up RDS:

Now go to AWS console and search for RDS

If you are currently in RDS dashboard(see the left nav panel)

you can scroll down to Create database and click on create database.

![dashboard-create](images\rds-dashboard-create.jpg)

If you are currently in databases you can simply click on create database if you see that.

![Create database](images\create-database.jpg)

1. Choose Standard create under database creation method.
2. Choose your Engine type, I'll be going with postgresSQL 12.8.
3. Choose Template. I'll be choosing free tire. You can choose production or dev/test if you don't have free tire(free database usage for 1st year).

Scroll down to settings.
![settings](images\db-settings.jpg)

Fill in database identifier. You can choose a name of your choice. Choose a password and username you can remember.

4. Now scroll down and under connectivity section -> Public access -> click on Yes.

You won't ideally need to keep public access to yes. But if you want to access your database from your computer or from PgAdmin on your computer, you will have to keep this yes. You can change this later. 

Don't change anything else. You can scroll down and click on create-database. Your database creation can take few minutes to cretae.

Once created the status will be changed to available(view this in databases panel). 