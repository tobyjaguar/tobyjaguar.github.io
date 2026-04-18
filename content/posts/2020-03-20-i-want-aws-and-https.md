---
title: "I Want AWS and Https"
date: 2020-03-20
---

## I Want AWS and Https
*...but how?*

![cake](/assets/posts/i-want-aws-and-https/_caitlyn-de-wild-sxIVqaksdfE.jpg)
`(Photo by Caitlyn de Wild on Unsplash)`

It has been a spell since I have written here, mainly because
I have been building out infrastructure for
<a href="https://www.smartpiggies.com/" target="_blank" rel="noopener noreferrer">SmartPiggies</a>.
And in doing so I wanted to create another post for posterity.

Jump to `Here is the good stuff` if you want to jump to the `good` stuff.

We are moving most of our services over to AWS and I had the great fortune
of navigating that rabbit hole. The initial challenge was to move
one of our apps over to an AWS server, but of course we want https
to be available. In previous cases we have used
<a href="https://certbot.eff.org/" target="_blank" rel="noopener noreferrer">cerbot</a>
to generate and provider the ssl certificate, as well as manage the
Nginx settings for port 443. This was very easy to implement on the
Ubuntu 16.04 servers that I usually run. When looking to move over to
AWS servers of course there is a twist.

Rather than spinning up an Ubuntu server (like I should have done), I went
with the first instance suggestion, an Amazon Linux 2 server. These servers
are a Red Hat Enterprise Linux (RHEL) flavor, and figuring out the transition
took a bit of work, so I am going to detail the solution I used to get
a cert from Amazon and integrate to support https on an app server. This is
mainly to serve as a document to remind me how to do it, but if it saves
anyone else some time then that is great.

So I began with an Amazon Linux 2 instance. No need to install persistent
iptables because they are not available, however you can install `iptables-services`
with yum. The security setting will be handled by security groups, so iptables and
firewall settings are less of an issue here (if this or any other information is
inaccurate, please for the love of all that is holy let me know so I can update this).

Once the instance is up, I installed nginx via `amazon-linux-extras` and setup
the sites-available directory with the app server settings. The app server is listening
on port 80 for connections, but we want port 443 to work for https connections. If
you use Amazon's DNS servers this whole process is easier, as you use Route 53 to
create records for your subdomains and aliases for ssl certificates. The rub here
is that we are using DNS servers from a different company, and can't change quite yet
as they currently serve the main site (this is annoying and particular to our situation
but for the purposes here, this is the challenge to surmount).

The goal then is to get port 443 forwarding to the app server, but Nginx is only
configured for port 80 at the moment, and if you add a 443 server in the Nginx
config for the app, it will fail because there is no certificate on the instance.
Here we can use Amazon Certificate Manager to provide us with an SSL Certificate for
all our subdomains (and root if needed). If you request a cert from Amazon you
will need to go through the routine process of proving ownership of the domain.
Amazon has two ways to do this, one by adding a DNS challenge, and the other is
via email verification. I could not get the DNS verification to work, so if anyone
has clarification on that I would love to hear it. I did get the email verification
to work, and was provided a cert for `*.examplesite.com` (where `examplesite` is
the site name).

## Here is the good stuff
After I had the cert I needed to figure out how to apply it. I could not figure
this out. I did find a helpful
<a href="https://www.youtube.com/watch?v=Sr2Mq9Gegew" target="_blank" rel="noopener noreferrer">video</a>
on youtube that explained a bit of the structural components of how to set this
up. There is one trick at the end here for my specific situation, so check back
if you watch that video, but I will also detail the process here. To use an
Amazon cert, it needs to be applied to a load balancer, and the load balancer
then needs to point to an instance. So after the cert is issued, I added a
load balancer to our account. This load balancer gets assigned the issued cert,
and then the load balancer targets the instance. There is a bit of setup here
that needs to be teased out. When you create the load balancer, I chose the
`Application Load Balancer`, you need to:
  * name it (something appropriate like app-LB or LBApp01, whatever)
  * select internet facing
  * select address type (here ipv4)
  * add listeners (more on this)
  * choose availability zones

### Listeners
The listeners will route to a specific port, and the default listener is
http port 80. You can add another listener for https port 443 here as well.

### Availability Zones
Amazon require you to choose two zones, one should obviously be the zone
the instance is in. The other, does not seem to matter.

### Configure Security Settings
Here you can assign the issued certificate. I used the default settings after
choosing the certificate.

### Configure Security groups
Here is one tricky bit in the setup. This certificate path begins with the load
balancer, which the certificate is assigned. Then the load balancer routes
traffic to one or more instances serving the application. So there are two
security groups that need to be set up, one for the load balancer and one for
the instances. Here is one magic bit, the source of the security group for the
instance needs to be the load balancer security group. Adam, from the video
above, suggests that this is the preferred setup for the internet traffic, but
I couldn't get the routing to work without setting the source for the instance
security group to the load balancer security group. To do this, you copy the
name of the load balancer security group, and paste it into the source for port
80 and port 443 in the instance security group.

To reiterate, you will have two security groups, one SG-App-LB, the security
group for the load balancer, and SG-App-Webapp, the security group for the
web application (again naming as appropriate). The set up for the load balancer
security group is project dependent but one assumes the load balancer has port
80 and port 443 open for http and https. The instance security group may also
have port 80 (http) and 443 (https) open and perhaps port 22 for ssh. For the
instance security group the input source for port 80 and 443 is the name of the
load balancer security group. This set up can be done after the load balancer
is configured, because we haven't yet saved the new load balancer.

### Configure Routing
Here create a new target group or an existing one. Basically there can be two
groups one for port 80 and one for port 443, however, the target group for 443
will forward to port 80 on the instance. The SSL connection
is made with the load balancer, remembering we do not have a certificate on
the instance. For the health check you can put a file name rather than
leaving it blank, as was suggested in the video, e.g. index.html or index.php

### Register Targets
Here is where you assign the instance to the load balancer. This should be
clickable in the instances window below, and added with the `Add to registered`
button

Once the load balancer is created, has an assigned cert, and has a registered
instance, there is one more thing to do. We must assign the 443 listener to
forward to port 80 rather than 443. Adam in the video above makes a point to
change this configuration `to` port 443 target group. But I believe it
will default to your http port 80 group. Either way confirm that the 443 listener
is forwarding to your port 80 target group. Again, this is because our Nginx
server on our instance isn't listening to port 443/https traffic.

Phew, that was a lot. The last thing to do is setup the Nginx config, and there
is yet another trick here. In the Nginx server settings for the app, say at
`/etc/nginx/sites-available/examplesite.com` you need to add the header
forwarding command block as follows:

```nginx
proxy_set_header X-Forwarded-Proto $scheme;
  if ( $http_x_forwarded_proto != 'https' ) {
    return 301 https://$host$request_uri;
  }
```

From what I've read it is some Nginx redirect logic in the header that allows
this setup to work.

All of the above is currently working on one of our apps, so it does seem
complicated, contrived, but successful. I had a bit of time on the 443 listener.
I had it set up for forward to port 443 on the instance, but the instance doesn't
listen to 443, it only redirects port 80 traffic. This resulted in a 502 error
until I changed the 443 listener to forward to port 80 on the app instance.

I hope this helps someone else, but more likely me the next time I have to do
this.

*disclaimer: these musings are offered, at best, as educational, and at worst for entertainment purposes. Do not take action on the descriptions above, as they contain risks, and are not intended as financial advice. Do not do anything above.*
