---
title: "onboarding"
linkTitle: "onboarding"
weight: 4
description: >
---

## Getting your apps connected to your users

The below picture gives a high level overview of nextensio architecture

![](/architecture/onboarding/architecture.jpg)

The picture shows 

* A user trying to get to some application backend either in Data Center A or B. The user device 
runs a light weight nextensio software called "Agent" that authenticates/validates the device
* Nextensio cloud gateways providing geo-proximal, redundant and loadbalanced acess to data centers
* Data centers running a light weight nextensio software called client that dials into nextensio gateways.
Note that data centers DO NOT have to punch holes in any firewalls or add any special security rules since
the connector initiates a connection from inside to outside
* Customer's application backends for two apps App-X and App-Y. The connector in picture provides 
connectivity to these two apps, so these two apps are considered part of an "App Group" that the connector serves
* A Nextensio controller which is the management plane where customer configures and manages users and policies
etc.., it also acts as the place from which all the telemetry information can be viewed and where all the 
telemetry based intelligent actions can be taken

## Becoming a Nextensio customer, steps involved

Let us first walk through how a new customer is bootstrapped, which should give a different perspective 
of the elements named above in the architecture diagram. Then we will deep dive into the different elements
in the architecture diagram.

### STEP1: Signing up 

Customer signs up online a controller.nextensio.net "Sign Up" page providing a name for the customer and
an admin email id. The admin will get an email notification and has to activate the account. Meanwhile 
Nextensio allocates resources for the new customer on its gateways. The list of gateways and their resources
can be modified later by the customer.

Front page             |  Signup Page
:-------------------------:|:-------------------------:
![](/architecture/onboarding/signup_1.jpg) | ![](/architecture/onboarding/signup_2.jpg)

### STEP2: Create users

In controller.nextensio.net, customer admin creates the users who are going to use nextensio services by 
providing their email ids. They get emails and they activate the accounts. Customer admin points them to 
the agent software the users need to install on their device (agents available for iOs, MacOs, Android, 
Windows, Linux). User installs the software and uses their email id and password and gets connected to the 
nearest / geo-proximal gateway.

Add user             |  Download user agent image
:-------------------------:|:-------------------------:
![](/architecture/onboarding/user_add.jpg) | ![](/architecture/onboarding/images.jpg)

For initial bootstrapping, the comment about "You have no attributes for users" can be ignored, 
it will be explained in more detail in section [Policies and Attributes](/architecture/policyattr.html). Also note 
that the first admin user who signed up will automatically show up in the users list.

NOTE: Nextensio will soon support federation with customer's Identity Provider and then the user creation
will not involve password etc.. because thats already handled by the IDP

### STEP3: Create AppGroups 

As seen in the architecture diagram, an AppGroup is just a logical way of organizing application backends 
in the data center. Its upto customer to decide what apps to group together - one way of grouping can be
based on apps that have similar security requirements. 

An AppGroup is also created in controller.nextensio.net with just another username (email id) and password ! 
And customer also specifies the URLs for the services/hosts that are members of the appgroup. For example 
if the customer domain is awesomecustomer.com, App-X might have the sub domain appx.awesomecustomer.com and 
App-Y might be appy.awesomecustomer.com

Add AppGroup             
:-------------------------:
![](/architecture/onboarding/appgroup_add.jpg)

For initial bootstrapping, the comment about "You have no attributes for AppGroup" can be ignored, it will be 
explained in more detail in section [Policies and Attributes](/architecture/policyattr.html)

* The Appgroup ID is an email id. 
* The Appgroup Name is just a descriptive string
* The AppGroup Compute Pods defaults to 1, customer can install multiple connectors
in their data center for the same appGroup - this can be for scale (load balancing) or just redundancy. The
number of compute pod configured here corresponds 1:1 to the number of connectors customer wants to run
for the same appGroup. The default value of 1, and having one connector is a good start!
* Services: in our example, customer will enter  
  appx.awesomecustomer.com,appy.awesomecustomer.com

### STEP4: Define properties of the services/hosts

The appGroup created above has member host/service URLs appx.awesomecustomer.com and appy.awesomecustomer.com. 
Each of these services might have independent properties. For example say appx and appy are both to be used
by engineering department, but appx is to be accessibly only by managers. Such properties can be defined 
in controller.nextensio.net for each of the host/service URLs, at the least create an entry with no properties 
defined, properties can be added/modified later.

Add Hosts             
:-------------------------:
![](/architecture/onboarding/host_add.jpg)

So in the above page, we would add TWO hosts - appx.awesomecustomer.com and appx.awesomecustomer.com - all the
attributes and tags etc.. can be just left alone, thats sufficient for bootstrapping, it will be explained in
more detail in section [Policies and Attributes](/architecture/policyattr.html)

### STEP5: Run connector in data center

Download the connector sofware and run it the DC on any server that can reach appx.awesomecustomer.com and
appy.awesomecustomer.com. The connector is also authenticated/validated by logging in with the username/password
that was created when creating the appGroup. The same connector software is used for any number of appGroups
customer has defined, there will need to be one connector launched per app group, the act of authenticating
the connector also makes the connector aware of what services/hosts its serving. 

And with these steps a new Nextensio customer is up and running, the users can access App-X and App-Y in the
data center. Once bootstrapped, there are powerful features that can be added on, which we will talk about in
future sections

Connector Image             
:-------------------------:
![](/architecture/onboarding/images.jpg)

Every connector connects to one nextensio gateway. The connector is linux binary that can be run independently
or packaged as a docker container or inside a kubernetes cluster etc.. - however customer chooses. The connector
is launched as "connector -gateway gatewaydosfo3.nextensio.net" as an example and then it will prompt for a 
user name and password - which is the data entered when creating AppGroups (section above). The list of nextensio
gateways can be seen below

#### Gateway details

Gateway Configuration page             |  List of available gateways
:-------------------------:|:-------------------------:
![](/architecture/onboarding/gateway_config.png) | ![](/architecture/onboarding/gateway_list.jpg)

* image: Nextensio is a fully managed solution, Nextensio takes care of upgrading the gateways. This gives an
option to specify a different image, we recommend not to change the value here. 
* Ingress (user) compute pods: If customer has thousands of users, there will be more compute requirements to handle
traffic from all those users. The compute units can be tweaked here. The default value of 1 is a fine start for
bootstrapping

The gatewayname gatewaydosfo3.nextensio.net is an example name for one of the nextensio gateways. Customer can
decide which gateway to connect to from a connector, or have multiple connectors for the appgroup each connecting
to different gateways etc.. - that is a decision customer can choose based on the geo-proximity of the gateways
to the data centers. 

Say customer chooses to connect the connector to gatewayA and customer's user agent direct the user traffic to
gatewayB - that will work just fine, Nextensio architecture will send traffic from gatewayB to gatewayA automatically

## Security Highlights

### Nextensio does NOT want customer keys, we CANNOT see customer data

None of the steps above discussed about needing customer's security/encryption keys. This is not an accident,
this is intentional in the architecture. Everything that nextensio sends from user devices to gateways and 
from gateways to connectors are all of course SSL encrypted with Nextensio's own keys which Nextensio manages.
But we dont want customer's keys, we dont need to peek into customer data to do anything that we do. We CANT
peek into customer's encrypted data

### Data Center is completely darkened

Connectors establish connection inside to outside, so there is no need to tinker with any firewall/security rules!

### We did not ask for any ip addressing details

We did not talk about ip addresses even once till now - that is intentional too. We do not care about how 
exactly appx.awesomecustomer.com is deployed, what is its subnet etc.. Customer can choose whatever 
ip addressing / subnetting / routing is required within the data center with the complete peace of mind 
that no external entity cares about it - so if the ip addressing has to change in future, the change is
restricted purely to customer's data center, there are no external ripple effects

## Next 

Having got an introduction to what Nextensio does, let us see all the flexible policies that Nextensio
provides a customer - [Policies and Attributes](/architecture/policyattr.html)
