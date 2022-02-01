# Django-channels:

If you are a beginner please make sure you have visited the uploading django. There are important concepts you need to know before proceeding.

Make sure you are using Application load balancer and not clasic loadbalancer.
Classic Load balancers don't support websockets.

You can check your loadbalancer type under LoadBalancer from configurations in the environment.

If you are using classic loadbalancer you will have to create a new environment with a application loadbalancer as there is no way you can upgrade from classic loadbalancer to application loadbalancer as of Feb-2-2022.


1. First add the following to your `requiremet.txt`:
```
gunicorn==20.0.4
daphne==3.0.1
supervisor==4.2.2
```

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


3. If you have appication loadbalancer. Go to environment -> loadbalancer -> edit:

4. If you want to use HTTPS and have ssl certificate then click on add listner and fill in the port number to be 443 choose your ssl certificate and choose any SSL policy from the dropdown.

![HTTPS](images\create-https-loadbalancer.png)

5. Then go ahead and add processes and match the below settings.(make sure you enable stickness in port 5000)
![process](images\loadbalancer-config.jpg)

You health check might fail with 100% of get request failing with 4xx. Thats why I filled a gave a different health check path that returns 200 Ok status.

6. Now under Rules match the below (if you aren't using HTTPS then change the listener port to 80) the process is the name you gave :
![rules](images\rules.jpg)

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

If you are using HTTPS and have SSL certificate replace the SSL certificate arn with your SSL certificate ARN.


## Connecting Redis:

Its highly likely you want a in-memory database such as redis.

### 1. Creating Redis through AWS console.

1. Search for ElastiCache and in elasticcache dashboard click on create.
![elastic cache](images\elasticache.jpg)

2. Let cluster engine be redis. Now under redis settings give your cluster a name and select a node type(note: only `cache.t2.micro` and `cache.t3.micro` comes under free tire (if you have free tire account)) 
![redis create](images\redis-create.png)

If you are using under free tire make sure you have set the node type to `cache.t2.micro` or `cache.t3.micro`. set number of replicas to 0 disable multi A-Z, disable automatic backups.

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

You might have to go to redis security group and allow EC2 instance to access it.

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
    Description : "ID of ElastiCache Cache Cluster with Redis Engine"
    Value :
      Ref : "MyElastiCache"
```

Now as shown above copy the primary endpoint and replace it under host.


### Debugging tips:

Daphne will run your websockets connection on port 5000. you can manually start this by going to your current directory activating your environment in your EC2 instance and running 
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
