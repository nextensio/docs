---
title: "attributes and policies"
linkTitle: "attributes and policies"
weight: 10
description: >
---

## Attributes 

Policies control everything in Nextensio. Lets look at some sample use cases, continuing with
the awesomecustomer.com App-X and App-Y examples from the [Architecture](/architecture) section
Once we run through the sample use cases, we will have a fair idea of what attributes and 
policies are and how to use them. 

### Attributes

#### Restrict App-X and App-Y to team "engineering"

So we created an appgroup containing appx and appy. Its a common use case to be needing to 
restrict an app usage to certain groups in an enterprise. This is how we would do it in 
nextensio 

* We will say that all users have an "attribute" called "team". Some users will have value
"engineering" for the attribute, some others can have "finance" etc..
* We will say that all appgroups have an attribute called "allowTeam" .. Our example appgroup
will have the value "engineering", some other appgroup might have the value "finance"
* We will write a policy for our appgroup that says  
  ```allow if user.team == appgroup.allowTeam```  
  Here the "user.team" is "engineering" finance etc.. based on which user is trying to access 
the appgroup

TODO ASHWIN: replace with exactly how the policy will be in Rego, with preferably just one line 

#### Restrict users with macOS < 10.15 

The user agent on the users devices informs nextensio gateways about the security/device posture
information of the device, and those are available as "inbuilt" attributes. So the policy in this
case will say ```deny if osType == "macOS" and (osVersion.Major < 10 OR osVersion.Minor < 15)```

TODO ASHWIN: replace with exactly how the policy will be in Rego, with preferably just one line 

#### Employees go to amazon, consultants go to digital ocean

Lets say our favorite appx.awesomecustomer.com and appy.awesomecustomer.com are hosted in both
digital ocean and AWS, but of course AWS is far more expensive that digital ocean. The attempt 
here is to minimize aws usage as much as possible to save on cost

So lets say the service in digital ocean is hosted as do.appx.awesomecustomer.com and 
do.appy.awesomecustomer.com and in aws its aws.appx.awesomecustomer.com and 
aws.appy.awesomecustomer.com. Of course the user does not know any of this, user is going to
access the service as appx.awesomecustomer.com or appy.awesomecustomer.com

So there are two connectors obviously - one running in AWS and one in Digital Ocean. The one for
AWS is configured with services "aws.appx.awesomecustomer.com,aws.appy.awesomecustomer.com" and
the one for Digital ocean with "do.appx.awesomecustomer.com,do.appy.awesomecustomer.com"

* We will say that all users have an attribute called "employment" with values "fulltime" or "consultant"
* We will say that all hosts will have an attribute "employmentMatch" with values either "fulltime" or "consultant"
* Will will also say that all hosts will have two "tags" - "aws" and "do  
  tag aws will have "employmentMatch" as "fulltime", tag do will have "employmentMatch" as "aws"
* We will define a policy which says  
  ```if host.tags.employmentMatch == user.employment {destination = host.tag}``` 

So here what we are telling in the policy is to pick from all the host tags, the tag that has
attribute "employmentMatch" matching the users attribute "employment". So if a full time user
accesses appx.awesomecustomer.com and matches tag "aws", nextensio will send the data over to
whichever connector advertises aws.appx.awesomecustomer.com

TODO ASHWIN: Replace the policy with the real one, hopefully not more than a handful of lines

TODO ASHWIN: Thinking about this, now I am convinced that the host attributes are very specific
to an appGroup. Different appGroups will have different set of host attributes, so enforcing that
every host have all the host attrs defined in the attribute editor wont fly I think - anyways 
thats for the future, just thinking aloud thats all

### Attribute Editor

From the above discussion, we now know that all three entities users and appGroups and Hosts have
"attributes", and those attributes can have "values". The attributes we discussed in the above
examples are listed below collected together

* user: team, osType, osVersion, employment
* appGroup: allowTeam
* host: employmentMatch

In addition, as we discussed, host as this additional entity called a "tag" to identify and differentiate
the same host/service running in different places/different versions/alpha-beta etc..

The values we saw for the attributes are mostly strings and numbers, again listing them down together from
all the examples

* user: team, employment, osType: value string, osVersion: value number
* appGroup: allowTeam: value string
* host: employmentMatch: value string

In addition, the "tag" for a host is always a string.

So the attribute editor is one single place where we define / lay down the attributes that will be 
used for users, appgroups and hosts and what type of value these attributes will contain - they can
contain single value strings, numbers, booleans or multi value (arrays) of strings, numbers or booleans

Attribute Editor             
:-------------------------:
![](/architecture/policyattr/attredit.jpg)

The above picture shows the attribute editor populated with the attributes we discussed above, with
the corresponding types (string, number). We do not have any example attribute that is multi-value (array),
hence all the attributes are created selecting the "Array" radio button to "False"

The attribute editor is not saying WHAT the value is, the attribute editor does not know the values, the
value for the same attribute of course can be different for different users. 

### Host Attributes

We show below  adding the attributes required for the AWS/Digital ocean example for host
appx.awesomecustomer.com. This section also needs an understanding of the [Routing](/architecture/routing)
section to get a complete picture

Add host             
:-------------------------:
![](/architecture/policyattr/host_add.jpg)

Edit host             
:-------------------------:
![](/architecture/policyattr/host_edit.jpg)

Add attributes for tags             
:-------------------------:
![](/architecture/policyattr/hostattr_edit.jpg)

Similarly we can add the second tag "do" with attribute employmentType set to "consultant"

### User Attributes

Add/Modify attributes             
:-------------------------:
![](/architecture/policyattr/userattr_edit.jpg)

The picture shows the attributes we discussed in the examples above, added to the "admin" user

### AppGroup Attributes

Add/Modify attributes             
:-------------------------:
![](/architecture/policyattr/appattr_edit.jpg)

The picture shows the attributes we discussed in the examples above, added to the "appxappy" appgroup

## Policies

We saw snippets of how a nextensio policy is written in the above examples. The nextensio policy language
is the Rego language which is getting fast industry acceptance and popularity as a simple intuitive and
expressive policy language. The policies are configured in the picture shown below

![](/architecture/policyattr/policy.jpg)

There can be multiple policies each used in a different context. Each policy has a name. The picture above
shows two policies AccessPolicy and RoutePolicy. AccessPolicy is what controls all the "restrict appx/appy
to engineering organization" kind of activities. RoutePolicy is what controls the "fulltime employees 
get routed to AWS, consultants to Digital Ocean" kind of policies

TODO ASHWIN: Are there only two types of policies that exist today ? What if customer wants completely 
different policies for two different appgroups. Or completely different policies for two different hosts ?
Is that still written as one big policy with "if appgroup == abc then this else that" or 
"if host == xyz then this else that" or do we allow customer to organize policies per appgroup/host ? 
Obviously I know the answer is NO :), its just a food for thought for future

### Policy Examples / Policy Library

Below we provide examples of template policies that can be cut + pasted and adapted. The template policies
cover a lot of the common scenarios and should be useful in getting off the ground fast.

TODO ASHWIN: pls fill this up

## Next 

Having understood attributes and policies, let us see how they are used in making routing decisions specifically -
[Routing](/architecture/routing/)
