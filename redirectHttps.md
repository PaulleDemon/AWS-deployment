# Redirecting traffic from Http to Https:

If you don't have a SSL certificate for your domin you can request one from the certificate manager in the AWS console. There are lot of tutorials so I won't be doing it

Chances are that you don't want your users to connect through HTTP you instead want them to be connected using HTTPS:

So for that go to EC2 console then in the left nav panel scroll down to load balancer -> click on listeners tab

![Load balancer](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/HttpsRedirect/https-redirect.jpg)

If you don't see HTTPS:443 under listeners then click on "add listeners" 

then under protocol click on HTTPS port no 443:
Then under default action dropdown click on "forward" to: select your target group where your EC2 resides. 

Now scroll down and select your SSL certificate:

and click on "add"

![https](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/HttpsRedirect/addlistener.jpg)

Now go to listeners and select the HTTP listener and click on edit:

From the "Default action" drop-down select "Redirect" to ->  select HTTPS protocol and port 443 then save changes.

![https-redirect](https://github.com/PaulleDemon/AWS-deployment/blob/master/images/HttpsRedirect/redirect_to_Https.jpg)

Thats it save the changes and now all http protocols will be redirected to https.
