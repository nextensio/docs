---
title: "Onboarding"
linkTitle: "Onboarding"
weight: 4
description: >
---

## Getting your apps connected to your users

The below picture gives a high level overview of nextensio architecture

![](/architecture/onboarding/architecture.jpg)

The picture shows 

* A user trying to get to some application backend either in Data Center A or B. The user device 
runs a light weight nextensio software called "Agent" that authenticates/validates the device
* Nextensio cloud gateways providing geo-proximal, redundant and loadbalanced access to data centers
* Data centers A and B running a light weight nextensio client software called Connector that dials
into nextensio gateways.
Note that data centers DO NOT have to punch holes in any firewalls or add any special security rules since
the connector initiates a connection from inside to outside
* Customer's application backends for two apps App-X and App-Y. Each connector in the picture provides 
connectivity to these two apps, and so these two apps are considered part of an "App Group" that the
connector serves. The connector takes the name of the AppGroup. To distinguish the connectors in the two datacenters, we will name one as AppGroup-DCA and the other as AppGroup-DCB. 
* A Nextensio controller which is the management plane where customer configures and manages users, policies
etc.., it also acts as the place from which all the telemetry information can be viewed and where all the 
telemetry based intelligent actions can be taken

To summarize, nextensio components are
* Agents (for user devices) - identified by user IDs
* Connectors (for data centers) - identified by AppGroup IDs
* Gateways (dispersed in the cloud) - agents and connectors connect to these
* Controller (in the cloud) - to configure and manage


## Becoming a Nextensio customer - steps involved

Let us first walk through how a new customer is bootstrapped, which should give a different perspective 
of the elements named above in the architecture diagram. Then we will deep dive into the different elements
in the architecture diagram. The steps involved are as follows:

1. Sign up as a nextensio customer
2. Create users
3. Define and create AppGroups. Each AppGroup will map to a Connector.
4. Define and create attributes
5. Define and create policies
6. Run the Connectors in the data center(s)

### STEP1: Signing up 

Customer signs up online at controller.nextensio.net "Sign Up" page providing a name for the customer and
an admin email id. The admin will get an email notification and has to activate the account. Meanwhile, 
Nextensio allocates resources for the new customer on its gateways. The list of gateways and their resources
can be modified later by the customer.

Front page             |  Signup Page
:-------------------------:|:-------------------------:
![](/architecture/onboarding/signup_1.jpg) | ![](/architecture/onboarding/signup_2.jpg)

### STEP2: Create users

In controller.nextensio.net, customer admin creates the users who are going to use nextensio services by 
providing their email ids. They get emails and they activate the accounts. Customer admin points them to 
the agent software the users need to install on their device (agents available for iOS, MacOs, Android, 
Windows, Linux). User installs the software and uses their email id and password to get connected to the 
nearest / geo-proximal gateway.

Add user             |  Download user agent image
:-------------------------:|:-------------------------:
![](/architecture/onboarding/user_add.jpg) | ![](/architecture/onboarding/images.jpg)

For initial bootstrapping, the comment about "You have no attributes for users" can be ignored. 
It is covered below in step 4 and also explained in more detail in section
[Policies and Attributes](/architecture/policyattr.html).
Also note that the first admin user who signed up will automatically show up in the users list.

NOTE: Nextensio will soon support federation with customer's Identity Provider and then the user creation
will not involve password etc.. because that's already handled by the IDP

### STEP3: Define and create AppGroups 

As seen in the architecture diagram, an AppGroup is just a logical way of organizing application backends 
in the data center and assigning them to a connector. Its upto the customer to decide what apps to group
together but apps grouped together should have similar security requirements. An AppGroup is mapped to a
Connector, hence this step basically defines the Connectors.

An AppGroup is also created in controller.nextensio.net with just another username (in email id format)
and password ! Customer also specifies the URLs for the applications that are members of the AppGroup.
For example, if the customer domain is awesomecustomer.com, App-X might have the URL appx.awesomecustomer.com
and App-Y might be appy.awesomecustomer.com.

Add AppGroup             
:-------------------------:
![](/architecture/onboarding/appgroup_add.jpg)

For initial bootstrapping, the comment about "You have no attributes for AppGroup" can be ignored. It will be 
covered in step 4 below and also explained in more detail in section [Policies and Attributes](/architecture/policyattr.html)

* The AppGroup ID is an email id. 
* The AppGroup Name is just a descriptive string
* The AppGroup Compute Pods (default 1). This represents the number of connectors per AppGroup ID. There can be multiple connectors for scale (load balancing) and/or redundancy. The default value of one connector is a good start!
* Services: in our example, customer will enter the application URLs appx.awesomecustomer.com, appy.awesomecustomer.com

Note that we will have to enter two records - one for AppGroup-DCA and one for AppGroup-DCB. This helps
identify each connector uniquely, and thereby apply access policies per connector.
We also have the option of using a single AppGroup ID and setting AppGroup Compute Pods to 2 if there is
no need to distinguish the connectors and controlling traffic to each connector.


### STEP4: Define and create attributes

Nextensio routes user traffic based on application URLs to AppGroups (or connectors). It allows having
multiple instances of any application and selecting a specific instance based on a routing policy.
Once an instance is selected (and there may be just one to select from), nextensio then determines the
AppGroup for that instance and sends the user's traffic to that AppGroup connector.
In this whole process, nextensio also provides two stages for access control - one  at the application
level via the Routing policy itself, and next at the AppGroup level via a separate Access policy.
Access control may be implemented at the first stage, or at the second stage, or at both stages,
depending on requirements.
The routing and access-control policies are all based on attributes, hence one needs to define and
create the attributes involved -
* User attributes - a set of attributes that apply to all users. Values will vary from user to user.
* App attributes - a set of attributes that apply to all applications. Values will vary from application to application and play a role in access control and routing.
* AppGroup attributes - a set of attributes that apply to all AppGroups. Values will vary from one AppGroup ID to another and play a role in access-control to a connector, and thereby, to a datacenter.

Out of these, the user attributes are mandatory. The other attributes are available depending on the
operational mode selected. Nextensio provides two modes - an 'Easy' mode and an 'Expert' mode. By default,
every new customer starts in the 'Easy' mode. In the
'Easy' mode, user attributes are the only attributes that need to be defined and maintained. The other
attributes are not available. Mode selection can be changed via the "Settings" option in the left-hand
panel.
* In the 'Easy' mode, policies are created by defining and maintaining rules in an easy assisted way.
* In the 'Expert' mode, the App and AppGroup attributes can be defined and maintained. This mode also
disables the 'Easy' mode rules and allows policies to be directly edited and maintained.

More details on this in a later section on policies and attributes.

The AppGroups created above have application URLs appx.awesomecustomer.com and appy.awesomecustomer.com as members. 
Let's say for example that App-X and App-Y in DCA are to be used only by engineering team, while App-X
and App-Y in DCB are to be accessible only by managers. We would first define two attributes per user, one to
represent the department, and one to indicate if user is a manager.
For the AppGroups, we could define two attributes per AppGroup ID, one to represent the team(s) allowed and one to indicate whether managers are allowed or not.
These attributes are created via controller.nextensio.net for each user, App and AppGroup.
We discuss this case in more detail in the Attributes and Policies section.

Note that the customer defines the attributes and creates them. Nextensio provides an attribute editor to
define the attributes and assign them to users, Apps, or AppGroups. Nextensio does not control or need to
know what the attributes are - they are defined and created exclusively by customers.

Add Apps
:-------------------------:
![](/architecture/onboarding/host_add.jpg)

So in the above page, we would add TWO applications - appx.awesomecustomer.com and appy.awesomecustomer.com.
The attributes can be left alone for now since adding the applications is sufficient for bootstrapping.
We can similarly skip the AppGroup attributes for bootstrapping.
The full capabilities will be explained in more detail in section [Policies and Attributes](/architecture/policyattr.html)

### STEP5: Define and create policies

This step is optional. By default, all user traffic is permitted without any routing support.
Note that multiple instances of a service are still possible even without routing - the instances
will be selected based on round-robin load-balancing.
Also, the default 'Easy' mode does not require dealing directly with policies. Instead, policies are
created via higher level rules. We will therefore skip this step. The types of policies and their full capabilities
will be explained in a later section.


### STEP6: Run connectors in data centers

At a high level, this consists of downloading the connector sofware to run it in DCA and DCB on any servers
that can reach appx.awesomecustomer.com and appy.awesomecustomer.com. The connector is provided with an authentication
key and the gateweay to connect to. The same connector software is used for any number of AppGroups
customer has defined. There will need to be one connector launched per AppGroup ID (there can be more as
covered above under 'Add AppGroup') . The act of authenticating the connector also makes the connector aware
of what applications it's serving. 
 
And with these steps, a new Nextensio customer is up and running; the users can access App-X and App-Y in the
two data centers. Once bootstrapped, there are powerful features that can be added on, which we will talk about in
future sections

The connector image can be downloaded from the Images page. Every connector connects to one nextensio gateway. The
connector is a linux binary that can be run independently or packaged as a docker container
or inside a kubernetes cluster etc., - however customer chooses. 

Connector Image             
:-------------------------:
![](/architecture/onboarding/images.jpg)



Before launching a connector, the customer needs to do two things :
* obtain an authentication key. 
* select a gateway from a drop-down list for the connector to connect to. 


Connector Authentication Key
:-------------------------:
![](/architecture/onboarding/connector_key.jpg)
![](/architecture/onboarding/connector_key_copy.jpg)


On the AppGroup configuration page, pick the AppGroup ID for which the connector needs to be launched and click on the
"key" icon located towards the right hand edge of the page. A small window will pop up displaying the key with a "Copy"
button on the lower right edge. Click this button to copy the key and then save it in a file. This file with the key
then needs to be transferred to the server where the connector will run. The recommended default location is
/opt/nextensio/connector.key, but the customer can put it anywhere else if desired.

Next, select a gateway from the drop-down list as shown below for 'List of available gateways'. The drop-down list
should be presenting gateways based on geo-location.

If the file containing the authentication key is in the default location, run
* "./connector -gateway \<gateway name\>"

If the file with the authentication key is in a different location, run
* "./connector -key \<key file location\> -gateway \<gateway name\>"

NOTE:

1. The above steps assume that the "connector" binary is stored in some directory of customers choice (it can be anywhere,
no specific requirement), and that customer is in that directory when typing "./connector" in the commands above

2. The connector can run as a normal user, it does NOT need root privileges

3. The connector logs are printed onto the console, it can be redirected/saved into a log location of customer's choice,
or it can just be left to log into the console

#### Gateway details

Gateway Configuration page             |  List of available gateways
:-------------------------:|:-------------------------:
![](/architecture/onboarding/gateway_config.png) | ![](/architecture/onboarding/gateway_list.jpg)

* image: Nextensio is a fully managed solution. Nextensio takes care of upgrading the gateways. This
gives an option to specify a different image. We recommend not to change the value here. 
* Ingress (user) compute pods: If customer has thousands of users, there will be more compute requirements
to handle traffic from all those users. The compute units can be tweaked here. The default value of 1 is
a fine start for bootstrapping

The gateway name gatewaydosfo3.nextensio.net is an example name for one of the nextensio gateways. Customer
can decide which gateway to connect to from a connector, or have multiple connectors for the AppGroup each 
connecting to a different gateway, etc.. - that is a decision customer can choose based on the geo-proximity of
the gateways to the data centers. 

Say customer chooses to connect the connector to gatewayA and customer's user agent directs the user traffic to
gatewayB - that will work just fine, Nextensio architecture will send traffic from gatewayB to gatewayA automatically

## Security Highlights

### Nextensio does NOT want customer keys, we CANNOT see customer data

None of the steps above discussed about needing customer's security/encryption keys. This is not an accident,
this is intentional in the architecture. Everything that nextensio sends from user devices to gateways and 
from gateways to connectors are all of course SSL encrypted with Nextensio's own keys which Nextensio manages.
But we dont want customer's keys as we dont need to peek into customer data to do anything that we do. We
CANNOT peek into customer's encrypted data.

### Data Center is completely darkened

Connectors establish connection inside to outside, so there is no need to tinker with any firewall/security
rules! There is no need to advertise your private applications/services to the world through DNS - keep it all within your
data center.

### Whatever happened to IP addresses ?

IP addresses have not been mentioned even once till now - that is not accidental ! Nextensio does not care
about IP addresses and how exactly appx.awesomecustomer.com or appy.awesomecustomer.com are deployed, what the subnets are, how they are
accessed from outside the data center, etc. Customer can choose whatever IP addressing / subnetting / routing
is required within each data center with the complete peace of mind that no external entity cares about them.
So if the IP addressing has to change in future, the change is restricted purely to customer's data center -
there are no external ripple effects

## Next 

Having got an introduction to what Nextensio does, let us see all the flexible policies that Nextensio
provides a customer - [Policies and Attributes](/architecture/policyattr.html)
