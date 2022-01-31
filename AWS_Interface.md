# Introduction to AWS console interface

If you are new to AWS you are very much likely to dislike the AWS interface.

In this blog we'll only be focusing on navigating in AWS interface, we'll also talk about some of the commonly used services.

Once you create an account on AWS you will mostly be working on their console.

You should be able find their console just by typing AWS console in browser.

![AWS console](images\aws_console.jpg)

<sub>This is how the console looks as of Jan-31-2022</sub>

Here you will be able to see the recently vistied items. You should be able to find their services just by typing the name of the service in the serch-bar located in the header.

## Commonly used services: 

1. **Elastic Beanstalk**: This is one of their service which makes it easier of deploying our web application. It combines many of the other services and helps you to easily create database, ec2 instance, loadbalancer etc. while still giving you a good control of the resources you use.

2. **RDS (Relational database service)** : 
    This is a relational database service offered by AWS. This is likely where you will be creating your database.

3. **S3(simple storage service)**: You will likely be using this service mostly to servce static files and media files directly from amazon.

4. **EC2(Elastic computer cloud)**: This is the service which provides you with the hardware necessary to run your application on cloud.

5. **Route 53**: 
from AWS:  https://aws.amazon.com/route53/
> Amazon Route 53 is a highly available and scalable cloud Domain Name System (DNS) web service. Amazon Route 53 effectively connects user requests to infrastructure running in AWS – such as Amazon EC2 instances, Elastic Load Balancing load balancers, or Amazon S3 buckets.

6. **Certificate manager**: This provides you SSL/TSL certificate you would like to use in aws, after validating your domain ownership.

7. **IAM(Identity and access management)**: If you are working as a team or you have employee's you want to give access to aws services. You will use the root account(your account) that has access to all the aws services to set-up an account for employee using IAM so that you have control over what they have access to in the AWS services.

8. **ElstiCache** This is used to start an in-memory database such as Redis. Useful when you have websocket and need a in-memory database(databases in RAM).

9. **Billing**: This is self-explainatoty

## Some abbreviation you should know:

1. **eb**: Elastic beanstalk
2. **sg**: Security group


## Navigating in AWS Console: 

### search:

Search is easy you can just type in the service you want and services click a service you intend to use.

![search](images\search.jpg)

### Elastic beanstalk:

If you had typed Elastic beanstalk, you will be presented with the below page:

![elastic_beanstalk](images\elastic_branstalk.jpg)

Now what you can create a new application by clicking on Application on the left navigation bar. You can also quickly visit you environments from the left navigation panel. An application can have many environment. Each environment should have unique name. You will know about this as you start with deployment. 