# Introduction to AWS console interface

If you are new to AWS you are very much likely to find it difficult to navigate through the AWS interface.

In this blog we'll only be focusing on navigating in AWS interface, we'll also talk about some of the commonly used services.

Once you create an account on AWS you will mostly be working on their console.

You should be able to find their console just by typing AWS console in the browser.

![AWS console](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/AWSinterface/aws_console.jpg)

<sub>This is how the console looks as of Jan-31-2022</sub>

Here you will be able to see the recently visited items. You should be able to find their services just by typing the name of the service in the search bar located in the header.

## Commonly used services: 

1. **Elastic Beanstalk**: This is one of their service which makes it easier of deploying our web application. It combines many of the other services and helps you to easily create a database, ec2 instance, loadbalancer, etc. while still giving you a good control of the resources you use.

2. **RDS (Relational database service)** : 
    This is a relational database service offered by AWS. This is likely where you will be creating your database.

3. **S3(simple storage service)**: You will likely be using this service mostly to service static files and media files directly from amazon.

4. **EC2(Elastic computer cloud)**: This is the service that provides you with the hardware necessary to run your application on the cloud.

5. **Route 53**: 
from AWS:  https://aws.amazon.com/route53/
> Amazon Route 53 is a highly available and scalable cloud Domain Name System (DNS) web service. Amazon Route 53 effectively connects user requests to infrastructure running in AWS – such as Amazon EC2 instances, Elastic Load Balancing load balancers, or Amazon S3 buckets.

6. **Certificate manager**: This provides you SSL/TSL certificate you would like to use in AWS, after validating your domain ownership.

7. **IAM(Identity and access management)**: If you are working as a team or you have employee's you want to give access to AWS services. You will use the root account(your account) that has access to all the AWS services to setup an account for an employee using IAM so that you have control over what they have access to in the AWS services.

8. **ElastiCache** This is used to start an in-memory database such as Redis. Useful when you have a websocket and need an in-memory database(databases in RAM).

9. **Billing**: This is self-explanatory

## Some abbreviations you should know:

1. **eb**: Elastic beanstalk
2. **sg**: Security group


## Navigating in AWS Console: 

### search:

Search is easy you can just type in the service you want and services click a service you intend to use.

![search](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/AWSinterface/search.jpg)

### Elastic beanstalk:

If you had typed Elastic beanstalk in the search bar, you would be presented with the below page:

![elastic_beanstalk](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/AWSinterface/elastic_branstalk.jpg)

Now, you can create a new application by clicking on Application on the left navigation bar. You can also quickly visit your environments from the left navigation panel. An application can have many environments. Each environment should have a unique name. You will know about this as you start with deployment. 

### EC2:

This is a very important page and you will often come back here so please note the items in the left navigation panel.
  
![EC2 Console](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/AWSinterface/ec2_console.jpg):

>Since not all the items are not visible I'll be writing it down important items that you will be using below

**The important items in the left navigation panels are as follows**

```
.
├── EC2 Dashboard
├── EC2 Global View
├── Events
├── ...
├── Instances:
│   ├── Instances (**important** Your current instances that you have created)
│   ├── Instance Type (List of all the instance types like: t1.micro, t2nano, a1.xlarge)
│   └── ...
├── Images:
│   └── ...
├── Elastic Block Store:
│   └── ...
├── Network & Security:
│   ├── Security Groups(**important** A security group acts as a virtual firewall for your EC2 instances to control inbound and outbound traffic)
│   │
│   └── Key Pairs (A Key pair, consists of a public key and a private key, is a set of security credentials that you use to prove your identity when connecting to an Amazon EC2 instance.)
├
├── Load Balancing:
│   ├── Load Balancers(**important** Displays all your loadbalancers)
│   └── Target Groups(A target group tells a load balancer where to direct traffic to: EC2 instances, fixed IP addresses; or AWS Lambda functions, amongst others.)
├
└── Auto Scaling
    └── ...
```


### RDS Console:

The most important item in the left panel is Databases highlighted in orange color

![RDS console](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/AWSinterface/RDS_dashboard.jpg)

### S3: 

The most important item in the left panel is the buckets themselves.

![S3 Bucket](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/AWSinterface/s3.jpg)

The others aren't that difficult so I'll leave it here.
