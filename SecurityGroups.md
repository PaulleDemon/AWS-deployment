# Security Groups

A security group acts as a virtual firewall for your EC2 instances to control the incoming traffic and outgoing traffic. You can control the incoming traffic by setting inbound rules and the outgoing traffic by setting the Outbound rules.

By default, there is a security group called as Default security group. You can also create your custom security group


## creating custom security group:

Go to EC2 console -> Scroll down to Security groups in the side navigation panel -> security group

Your security groups will appear here. You can create a new security group, modify inbound/outbound rules for existing security groups from here.
![security group](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/security-groups/security_group.jpg)

1. Click on create security group.

![Create security group](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/security-groups/security-group-create.png)

Give it a security group name, give an optional description. You can choose a default VPC.

When you click on add rule below are common things you will have to fill for both

### Common functions for inbound and outbound
The Type contains many Types like custom TCP, All traffic, All TCP, HTTP, HTTPS, PostgresSQL.

Some of the predefined Types such as PostgresSQL will automatically fill the port number to 5432 and protocol to TCP. HTTPS will assign 443 as port and protocol to TCP.

In source, you can choose who should be allowed.
Sources include Anywhere-IPv4, Anywhere-IPv6, custom and My IP.

Now be careful while choosing My Ip. If you are connected through a modem it's likely that your IP address might keep changing. So you might have to update my IP in the secuity group each time.


You can add as many rules as you want in a security group.

### Inbound rules:

This is what controls the incoming traffic. Your rules define who should be allowed access and who shouldn't be.

### Outbound rules:

This controls the outgoing traffic. This rule checks on the outgoing access.

You can now assign this to your EC2 instance or even modify the security groups of EC2, RDS S3 buckets, etc.
