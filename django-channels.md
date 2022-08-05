# Django-channels:

If you are a beginner please make sure you have visited the uploading Django. There are important concepts you need to know before proceeding.

Make sure you are using an Application load balancer and not a classic loadbalancer.
Classic Load balancers don't support WebSockets.

You can check your loadbalancer type under LoadBalancer from configurations in the environment.

If you are using classic loadbalancer you will have to create a new environment with an application loadbalancer as there is no way you can upgrade from classic loadbalancer to application loadbalancer as of Feb-2-2022.

## setup

1. First add the following to your `requiremet.txt`:
```
gunicorn==20.0.4
daphne==3.0.1
supervisor==4.2.2
```

> Daphne is a HTTP, HTTP2 and WebSocket protocol server for ASGI and ASGI-HTTP, developed to power Django Channels. we'll be using this to run our ASGI application

> The Gunicorn "Green Unicorn" is a Python Web Server Gateway Interface HTTP server. 

2. create a file named `Procfile` and add the following:
```
web: gunicorn --bind :8000 --workers 3 --threads 2 <project>.wsgi:application
websocket: daphne -b 0.0.0.0 -p 5000 <project>.asgi:application
```
replace the project with your project name.

Now your structure should look below.
```
.
└── sampleproject
    ├── .ebextensions
    │   └── django.config
    ├── app1
    ├── app2
    ├── sampleproject
    │   ├── __init__.py
    │   ├── settings.py
    │   ├── urls.py
    │   ├── wsgi.py
    │   └── asgi.py
    ├── app3
    ├── Procfile #<------------here
    └── .gitignore
``` 


3. If you have an appication loadbalancer. Go to environment -> loadbalancer -> edit:

4. If you want to use HTTPS and have an SSL certificate then click on add listener and fill in the port number to be 443 choose your SSL certificate and choose any SSL policy from the dropdown.

![HTTPS](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/ElastiCache/create-https-loadbalancer.png)

5. Then go ahead and add processes and match the below settings. (make sure you enable stickiness in port 5000)
![process](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/ElastiCache/loadbalancer-config.jpg)

Your health check might fail with "100% of GET request failing with 4xx". That's why I gave a different health check path that returns 200 Ok status.

6. Now under Rules match the below (if you aren't using HTTPS then change the listener port to 80) the process is the name you gave, this will make sure to redirect all the path that has `/ws/` is redirected to the websocket process :
![rules](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/ElastiCache/rules.jpg)

>Note: If you are using HTTPS protocol then you should use wss protocol eg: wss://ws/xys otherwise use ws://ws/pqr

### Way 2:
If you haven't created your environment yet you can just add the following in `01_loadbalancer.config` under `.ebextensions`
Note you should also follow till step 2 from the above.
```
option_settings:
  aws:elbv2:listener:443:
    ListenerEnabled: 'true'
    SSLCertificateArns: arn:<SSL certificate arn>
    Protocol: HTTPS
    Rules: ws
  aws:elbv2:listenerrule:ws:
    PathPatterns: /ws/*
    Process: websocket
    Priority: 1
  aws:elasticbeanstalk:environment:process:websocket:
    Port: '5000'
    Protocol: HTTP
```

If you don't want to use HTTPS you can remove the
```
aws:elbv2:listener:443:
    ListenerEnabled: 'true'
    SSLCertificateArns: arn:<SSL certificate arn>
    Protocol: HTTPS
    Rules: ws
```

If you are using HTTPS and have an SSL certificate replace the SSL certificate arn with your SSL certificate ARN.


## Connecting Redis:

It's highly likely you want an in-memory database such as Redis.

### 1. Creating Redis through AWS console.

1. Search for ElastiCache and in elasticcache dashboard click on create.
![elastic cache](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/ElastiCache/elasticache.jpg)

2. Let cluster engine be Redis. Now under Redis settings give your cluster a name and select a node type(note: only `cache.t2.micro` and `cache.t3.micro` comes under free tire (if you have a free tire account)) 
![redis create](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/ElastiCache/redis-create.png)

> If you are eligible for free tire make sure you have set the node type to `cache.t2.micro` or `cache.t3.micro`. Set number of replicas to 0 disable multi A-Z, disable automatic backups.

Now click on create.

Once created copy the primary endpoint and in settings.py:

```py
...
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            "hosts": [('<Primary endpoint>', 6379)],
        },
    },
}
...
```

You might have to go to the Redis security group and allow the EC2 instance to access it.

You will Also have to create a [security group](https://github.com/PaulleDemon/AWS-deployment/blob/master/SecurityGroups.md) and associate it with the ec2 instance.

1. Go to Ec2 > security groups > click on create security group.

2. Now give it a name say, <project_name>_redis, add a small description.(optional)

3. Now under inbound rules, click on add inbound rule.

4. set type to `custom TCP`, port to 6379 (redis port), set source to custom and search for your environment security group and set it to that.

5. Go to the bottom of this page to see how to test redis connection.

### 2. Connecting through config file(recommended)

create a file called `elastic_cache.config` in `.ebextensions` folder and paste the below code:

```
Resources:
  MyCacheSecurityGroup:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupDescription: "Lock cache down to webserver access only"
      SecurityGroupIngress :
        - IpProtocol : "tcp"
          FromPort :
            Fn::GetOptionSetting:
              OptionName : "CachePort"
              DefaultValue: "6379"
          ToPort :
            Fn::GetOptionSetting:
              OptionName : "CachePort"
              DefaultValue: "6379"
          SourceSecurityGroupName:
            Ref: "AWSEBSecurityGroup"
  MyElastiCache:
    Type: "AWS::ElastiCache::CacheCluster"
    Properties:
      CacheNodeType:
        Fn::GetOptionSetting:
          OptionName : "CacheNodeType"
          DefaultValue : "cache.t2.micro"
      NumCacheNodes:
        Fn::GetOptionSetting:
          OptionName : "NumCacheNodes"
          DefaultValue : "1"
      Engine:
        Fn::GetOptionSetting:
          OptionName : "Engine"
          DefaultValue : "redis"
      VpcSecurityGroupIds:
        -
          Fn::GetAtt:
            - MyCacheSecurityGroup
            - GroupId

Outputs:
  ElastiCache:
    Description: "ID of ElastiCache Cache Cluster with Redis Engine"
    Value :
      Ref : "MyElastiCache"
```

Now as shown above copy the primary endpoint and replace it under host.


### Debugging tips:

To test the connection first ssh into your instance.

then type 
```
source /var/app/venv/*/bin/activate
cd /var/app/current/
```
now type `python manage.py shell` and give the following command

```
>>> import channels.layers
>>> from asgiref.sync import async_to_sync
>>> channel_layer = channels.layers.get_channel_layer()
>>> async_to_sync(channel_layer.send)('test_channel', {'foo': 'bar'})
>>> async_to_sync(channel_layer.receive)('test_channel')
>>> {'foo': 'bar'} # <---------- you should receive this as output if everything went well
```

If you receive connection timeout, its most likely related to your security group.

Daphne will run your WebSockets connection on port 5000. you can manually start this by going to your current directory activating your environment in your EC2 instance and running 
```
daphne -b 0.0.0.0 -p 5000 projectname.asgi:application
```
replace projectname with your project name.

full steps:
```sh
eb ssh
Enter your passphrase:

---- Elastic Beanstalk----

source /var/app/venv/*/bin/activate
cd /var/app/current/
daphne -b 0.0.0.0 -p 5000 projectname.asgi:application
```
