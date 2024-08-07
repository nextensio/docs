---
title: "Routing"
linkTitle: "Routing"
weight: 10
description: >
---

## Same Application, multiple instances

Nextensio provides flexibility when it comes to routing to multiple instances of any
application. A simple automatic way is via load balancing. When multiple connectors advertise
the same application (from two or more connectors either in the same data center, different
data centers, or a combination of both), nextensio will load balance to the connectors.

However, there may be situations where a customer wants to control how traffic flows to
each instance of the application. One example is A/B testing of an application when a new version
(alpha or beta) is to be deployed. Another is migration of any application(s) from one cloud
provider to another or from one data center to another within the same provider. Yet another is
where each instance is in a data center hosted by a different cloud provider and customer wants
to control traffic to either minimize costs or to provide a more reliable service to a subset
of users (senior executives, premium paying customers, and so on). Nextensio allows controlling
which traffic goes to which instance via a Routing policy.

The Routing policy is a way to define rules that match a user's attributes with the attributes
of the application that the user is trying to access and determine which instance of the application
should be accessed (if at all, but more on this in the Access Control section). The policy
indicates the selected instance by returning a tag that is prefixed to the original application
url to derive the actual instance. The multiple instances of an application (or service) are
therefore differentiated via a prefix tag and these tagged application names are the ones configured
as the services for the associated AppGroup ID (aka connector).

In the section on [Policies and Attributes](/docs/architecture/policyattr.html), we talked
about an example application appx.awesomecustomer.com present in two different data centers DCA and
DCB, hosted in AWS and Digital Ocean, and we talked about how we can use attributes to
route users to one or the other data center using attribute match. We left out a
few key details there which we will expand on here.

### AppGroup Services Vs Applications

AppGroup services configuration and the application URLs that we configure are referring
to the same thing. So there is the example application we use in our discussions,
appx.awesomecustomer.com. Lets say that application needs multiple instances to be hosted, 
and with that in mind lets consider a few cases.

#### Identical instances of applications

So if all the data centers are just the same as far as appx.awesomecustomer.com is
concerned and there is NO reason to differentiate the data centers, its a simple 
configuration as below.

App Group Config             |  App config
:-------------------------:|:-------------------------:
![](/docs/architecture/routing/appgroup_config.jpg) | ![](/docs/architecture/routing/host_config.jpg)


appx.awesomecustomer.com is defined in the single AppGroup ID "appxappy@awesomecustomer.com"
as the service, and the application does not have any extra configs; it just adds the URL name
appx.awesomecustomer.com and gets out. The routing policy returns an empty or null string ("")
as a default (no rules are needed in the policy).

The same AppGroup ID "appxappy@awesomecustomer.com" can be used from any number of data centers to
authenticate and connect to the nextensio clusters. Nextensio will just loadbalance the users
across these data centers. Note that when using the same AppGroup ID, a routing policy cannot be
applied. Also, the number of compute pods should be set to the number of connectors with this
AppGroup ID (the default is 1).

It is not mandatory to use the same AppGroup ID. If it is expected that a routing policy may be
required in future to control traffic to these instances, it would be prudent to use separate
AppGroup IDs per instance from the get go. So for example if we have AppGroup-DCA@awesomecustomer.com
and AppGroup-DCB@awesomecustomer.com, each AppGroup ID would still be configured with the same
service names (eg., appx@awesomecustomer.com). The compute pods value would be left at the default
in this case.


#### Differentiated instances of applications

Now if we need to differentiate applications in amazon v/s applications in digital ocean (for example),
whatever be the reason for the difference (cost of access or reliability etc..) and want to
preferably route users to either of these DCs using attributes, then the below is how we will
configure the service.

 App Group Config             |  App config
:-------------------------:|:-------------------------:
![](/docs/architecture/routing/appgroup_differ_config.jpg) | ![](/docs/architecture/routing/host_differ_config.jpg)

Note that under the App config, we have defined two "tags" - one "aws" (indicating amazon) and another "do"
(indicating digital ocean). 

Also note that we now have TWO AppGroup IDs defined - AppGroup-aws@awesomecustomer.com and
AppGroup-do@awesomecustomer.com (or AppGroup-DCA@awesomecustomer.com and AppGroup-DCB@awesomecustomer.com
as mentioned earlier).
The former will be used to connect from amazon DC to nextensio gateways, and the latter to connect from
digital ocean DC. In each of these AppGroup IDs, the application URL is prefixed by "aws" or "do".

So this is how we can logically think of how routing happens

1. User comes up with attribute say "employment": "fulltime" and going to appx.awesomecustomer.com
2. Nextensio looks up the application appx.awesomecustomer.com and finds that it has two tags "aws" and "do".
Nextensio picks that tag which has the matching attribute "employmentMatch": "fulltime" - in this case
it is the tag "aws". 
3. Now nextensio tries to see which AppGroup ID advertises a service called "aws.appx.awesomecustomer.com",
and finds that its advertised by AppGroup ID AppGroup-aws@awesomecustomer.com and sends data over
the connection established by that AppGroup ID - which is a connection from AWS data center

Note that the extra "aws" or "do" tag that we add to the URL is just nextensio internal, purely to identify
a specific AppGroup ID that needs to be chosen for this traffic - we DO NOT expect customer to host applications
with those tag added URLs. Customer will just host appx.awesomecustomer.com in BOTH amazon and digital ocean

The exact details of how step2 does the "attribute match" is defined using policies as described in the
section on [Policies](/docs/configurations/policies.html). In the 'Easy' mode, the policy is created by defining
rules for each App. In the 'Expert' mode, the policy can be written and edited directly.

## Next 

The next topic of interest will be how to control which user can access what resources, in 
section [Access Control](/docs/architecture/accesscontrol.html)
