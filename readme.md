# Deploying web application to AWS

Deploying your project to AWS can be very difficult and frustrating if you are new to deploying a project. Neither is AWS user-interface good to work with nor is their documentation.

I recently had to deploy django application which also makes use of websockets and it took me around 8 days to get it properly up and running. Most of the errors occurred because I didn't understand their system well.

In this blog I wish not only help you deploy your django project but also help you debug some of the common errors that I faced.

While I'll be using django, most of the steps followed will remain same accross different frameworks and programming languages.

> If think some steps are missing or is incorrect please create a new issue on this github repository or create a new pull request.

### Table of contents:

1. Introuduction to AWS interface
2. Deploying our django application
3. Using RDS to create database
4. Using Environment variables to hide our sensitive info's
5. Using S3 storage to store static and media files.
6. Redirecting Http to Https
7. Deploying django channels application
8. some debugging tips.
