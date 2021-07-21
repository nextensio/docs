---
title: "routing"
linkTitle: "routing"
weight: 10
description: >
---

## Same Service, multiple datacenters

In the section on [Policies and Attributes](/architecture/policyattr), we talked
about an example service appx.awesomecustomer.com present in two different data
centers AWS and Digital Ocean, and we talked about how we can use attributes to
route users to one or the other data center using attribute match. We left out a
few key details there which we will expand on here.

### AppGroup Services Vs Hosts

AppGroup services configuration and the Host names that we configure are referring
to the same thing. So there is the example service we use in our discussions,
appx.awesomecustomer.com. Lets say that service is hosted in multiple data centers,
and with that in mind lets consider a few cases

#### All datacenters identical

So if all the data centers are just the same as far as appx.awesomecustomer.com is
concerned and there is NO reason to differentiate the data centers, its a simple 
configuration as below.

App Group Config             |  Host config
:-------------------------:|:-------------------------:
![](/architecture/routing/appgroup_config.jpg) | ![](/architecture/routing/host_config.jpg)



 appx.awesomecustomer.com is defined in both the appgroup and the host and the host 
 does not have any extra configs, it just adds the URL name appx.awesomecustomer.com and gets out

 And this same appgroup "appxappy@awesomecustomer.com" can be used from any number of 
 data centers to authenticate and connect to the nextensio clusters. Nextensio will just loadbalance
 the users across these data centers

 #### Datacenters are different

 Now if we need to differentiate services in amazon Vs services in digital ocean (for example),
 whatever be the reason for the difference (cost of access or reliability etc..) and want to
 preferably route users to either of these DCs using attributes, then the below is how we will
 configure the service

 App Group Config             |  Host config
:-------------------------:|:-------------------------:
![](/architecture/routing/appgroup_differ_config.jpg) | ![](/architecture/routing/host_differ_config.jpg)

Now that under the host config, we have defined two "tags" one "aws" (indicating amazon) and another "do"
(indicating digital ocean). 

Also note that we have TWO appgroups defined appgroup-aws@awesomecustomer.com and appgroup-do@awesomecustomer.com.
The former will be used to connect from amazon DC to nextensio gateways, and the latter to connect from
digital ocean DC. And note that in each of these appgroups, the service URLs are prefixed by "aws" or "do"

So this is how we can logically think of how routing happens

1. User comes up with attribute say "employment": "fulltime" and going to appx.awesomecustomer.com
2. Nextensio looks up the host appx.awesomecustomer.com and finds that it has two tags "aws" and "do",
nextensio picks that tag which has the matching attribute "employmentMatch": "fulltime" - in this case
it is the tag "aws". 
3. Now nextensio tries to see "which appgroup advertises a service called aws.appx.awesomecustomer.com",
and nextensio finds that its advertised by appgroup appgroup-aws@awesomecustomer.com and sends data over
the connection established by that appgroup - which is a connection from AWS data center

Note that the extra "aws" or "do" tag that we add to the URL is just nextensio internal, purely to identify
a specific appgroup that needs to be chosen for this traffic - we DO NOT  expect customer to host URLs
with those tags added. Customer will just host appx.awesomecustomer.com in BOTH amazon and digital ocean

The exact details of how step2 does the "attribute match" is defined using policies as described in the
section on [Policies](/architecture/attrpolicy)

## Next 

The next topic of interest will be how to control which user can access what resources, in 
section [Access Control](/architecture/accesscontrol)
