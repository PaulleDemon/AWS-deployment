# Route53

**What is DNS(domain name system)?**

Every device connected on the internet has a unique IP address, which other machines use to find the device. 
The DNS eliminates the need for us humans to memorize IP addresses, such as 192.164.2.1(IPv4) and instead help us use more friendly names such as 
[peckspace.com](peckspace.com) to browse websites.
<br>
The process of DNS resolution helps in converting  hostnames back into computer-friendly IP address such as 192.164.2.1.

## Route 53:

from AWS:  https://aws.amazon.com/route53/

*"Amazon Route 53 is a highly available and scalable cloud Domain Name System (DNS) web service. Amazon Route 53 effectively connects user requests to infrastructure running in AWS – such as Amazon EC2 instances, Elastic Load Balancing load balancers, or Amazon S3 buckets."*

## connecting Elasticbeanstalk to custom domain

Once you have purchased a domain name. Search for Route53 from the search bar.

1. From the left sidebar click on hosted zones, and then click on create hosted zone.
2. In the hosted zone configuration page, under domain name give your domain name, and give an optional description.
3. Now thats done, You will see two rows, One with Type(column type) NS(name server) and other with type SOA. Copy all the values(the last column)from the NS row.
4. Go to the website you purchased the domain from and find DNS management console and update all the NS values there to the one given by the AWS.
6. now create a record and under record name give it a subdomain name like www or about or just leave it blank. (Note: if you use *, it will act as a wild card, meaning all the subdomains will be captured)
7. Let the record type be "A-Routes traffic to an IPv4 address and some AWS resource".
8. Now turn on the Alias near to the value, You will be shown a Route traffic to dropdown.
9. From the drop-down choose "Alias to ElasticBeanstalk environment".
10. Choose the region you have your ElasticBeanstalk environment, then choose your correct environment. Now click on create record. 

Thats it will now be able to use your custom domain.

## Email configuration

If you have purchased email for your domain from some other website and have hosted your domain in Route53, 
you will not be able to send emails unless you create the records thats asked by email provider.

They usually ask you to create MX record, TXT record and couple of CNAME. 

Creating the records are same as above, you just need to change record type to MX or TXT depending what your mail provider asks you to.

Then just copy the values and create those records.


## Hosting static websites

Sometimes you might need to take a maintenance break for your main website, and you probably don't want the users to write or read your website.

In the mean time you can serve a static website from s3 bucket telling your users that the website is currently under maintenance.

To do so, follow the following steps.

1. Go to s3 bucket and create a bucket matching your domain, for example if your domain is `example.com` then name your bucket `example.com`
2. once you have created that bucket, click on the bucket and under properties tab, scroll down and enable static website hosting.
3. Upload all your static website files into that bucket
3. Specify your index document and click on save changes
4. Under bucket policy in the permissions tab make it read only by editing it and adding the following.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Resource": "arn:aws:s3:::example.com/*"
        }
    ]
}
```
Replace example.com with your bucket name.

5. Go back to Route53 and find the subdomain that matches your bucket name, in my case it was `example.com`. Click on edit values.
6. Now under route traffic to dropdown(this will appear if you have enable alias), click on "Alias to s3 website endpoint".
7. Choose the region and your bucket from the dropdown then click on save.

That it now you can server your static website from s3 bucket

