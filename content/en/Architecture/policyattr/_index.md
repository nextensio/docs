---
title: "Attributes and Policies"
linkTitle: "Attributes and Policies"
weight: 10
description: >
---

## Introduction 

Policies play an important role in Nextensio. Lets look at some sample use cases, continuing with
the awesomecustomer.com App-X and App-Y examples from the [Architecture](/architecture.html) section.
Once we run through the sample use cases, we will have a fair idea of what attributes and 
policies are and how to use them. 

A nextensio policy basically takes an input and evaluates it using rules that implement some logic, either
for routing or access control. The input consists of user attributes plus either the name of the
host/service being accessed (in case of a Route policy) or the AppGroup ID being accessed
(in case of the Access policy). The policy rules are written in a language called Rego, an industry
standard developed as part of the Open Policy Agent (OPA).

At a high level, there are two strategies for implementing any policy:
* hardcode the values to match within the rules so that the policy is self contained
* reference external data in the rules to get the values to match. The reference data may be a file with
host attribute records or a file with AppGroup attribute records

Which strategy is chosen depends entirely on which one is easier to manage, driven by the scale of
hosts and AppGroup IDs. Note that one can also use a combination of the two strategies - one strategy for
the Route policy and the other for the Access policy (or vice versa). 
It is of course also possible to maintain the reference data files for host and AppGroup attributes
but ignore them in the policies by using hardcoded values.

Each customer can decide what works best for them and change strategies if and when needed.


### Access control by Appgroup IDs (Connectors)

A data center may have one or more connectors identified by their AppGroup ID. Access into a
data center can be controlled at the connector level using an Access policy. The Access policy
may or may not use AppGroup attributes as reference data, as mentioned above.


#### Case1: Restrict access by user team and role

Let's assume data center DCA is running engineering applications and data center DCB is running
HR applications. We therefore want to restrict access to AppGroup-DCA to those in "engineering"
team, and restrict access to AppGroup-DCB to users with the role of "managers" as well as users
in the "hr" team. Note that the "engineering" team can have "managers" - they should be able to
access both data centers. For simplicity, we will assume that a user can be in just one team.
Such a case gives the gist of a common use case that involves the need to restrict app usage
to certain groups in an enterprise. This is how we would do it in nextensio.

* We will say that all users have an "attribute" called "team". Some users will have value
"engineering" for the attribute, others can have "hr", "finance", "sales", "marketing", etc.
* We will also create a user attribute called "userRole" that can have values such as "manager"
and "non-manager".
* We will say that all AppGroups have an attribute called "allowTeam". AppGroup-DCA will have
the value "engineering", AppGroup-DCB will have the value "hr".
* We will also have an attribute called "roleAllowed" for all AppGroups. AppGroup-DCB will have
the value of "manager" while AppGroup-DCA can have the empty value or a value that no user
can have.
* We can write an Access policy using each of the two strategies indicated earlier to implement
this logic:

```Table 1
             User   	     	     DCA	  DCB
"engineering"	 "non-manager"	     allow	  deny
"engineering"	 "manager"	     allow	  allow
"hr"		 "non-manager"	     deny	  allow
"hr"		 "manager"	     deny	  allow
<any-other-team> "manager"	     deny	  allow
```

Option 1 - without using AppGroup attributes

```Policies
allow = is_allowed
default is_allowed = false
is_allowed {
        user.destination == "AppGroup-DCA"
	user.team == "engineering"
}
is_allowed {
        user.destination == "AppGroup-DCB"
	user.userRole == "manager"
}
is_allowed {
        user.destination == "AppGroup-DCB"
	user.team == "hr"
}

Option 2 - using AppGroup attributes as reference data

allow = is_allowed
default is_allowed = false
is_allowed {
        some AppGroup
	# ensure we select the correct AppGroup ID (connector)
	user.destination == attributes[AppGroup].AppGroupID
	# AppGroup-DCA allowTeam = "engineering", AppGroup-DCB allowTeam = "hr"
	user.team == attributes[AppGroup].allowTeam
}
is_allowed {
        some AppGroup
	user.destination == attributes[AppGroup].AppGroupID
	# only AppGroup-DCB attribute roleAllowed = "manager"
	user.userRole == attributes[AppGroup].roleAllowed
}
```

The destination AppGroup ID is available in the AppGroupID attribute.
The 'some AppGroup' line is to iterate through multiple AppGroup IDs (we have two here) to ensure the
allowTeam or the roleAllowed attribute checks are done with the correct AppGroup ID record. 


#### Case 2: Restrict users based on device posture (o/s version) 

The user agent on the user devices informs nextensio gateways about the security/device posture
information of the device, and those are available as "inbuilt" attributes. So the policy in this
case can have this rule

```
allow = not deny
deny {
	user.osType == "macOS"
	user.osVersion < 10.15
}
```

What's interesting here is that since the deny case is very specific, the rule has to be written
for deny and the allow value derived via negation. Writing the rule for allow would involve checking
for all possible osTypes and their versions, making the rule much more complex.
Note that this is a very simple case where the osType and osVersion values are hardcoded in the
policy and applies to all AppGroup IDs. This may not be desirable in more complex cases, or where
the criteria needs to vary depending on the AppGroup ID (eg., some AppGroup IDs restricted to users of
very old macOS versions and some AppGroup IDs restricted to users of very old Windows versions).
The use of AppGroup attributes as reference data may be considered for more complex cases and the
policy written accordingly.


#### Case 3: Routing to optimize data center cost

Lets say that DCA is in AWS and DCB in Digital Ocean and both data centers run the same applications.
Assuming AWS is more expensive than digital ocean, the goal here is to minimize aws usage as much as
possible to save on cost - so restrict it to fulltime employees. Goal, therefore, is to have fulltime
employees go to AWS, consultants go to digital ocean.

So lets say the service in digital ocean is hosted as do.appx.awesomecustomer.com and 
do.appy.awesomecustomer.com and in aws its aws.appx.awesomecustomer.com and 
aws.appy.awesomecustomer.com. Of course the user does not know any of this, nor does it change the
names of applications in the data centers. User is going to access the service as appx.awesomecustomer.com
or appy.awesomecustomer.com. The prefixes of "aws" and "do" differentiate the two instances of
each service only within the nextensio network.

So there are two connectors obviously - one running in AWS and one in Digital Ocean. The one for
AWS is configured with services "aws.appx.awesomecustomer.com,aws.appy.awesomecustomer.com" and
the one for Digital Ocean with "do.appx.awesomecustomer.com,do.appy.awesomecustomer.com"

* We will say that all users have an attribute called "employment" with values "fulltime" or "consultant"
* We will say that the two hosts will each have two route "tags" - "aws" and "do"  
* We will also say that each route tag will have an associated attribute called "employmentMatch" with
a value of either "fulltime" or "consultant"
Route tag "aws"'s attribute will have "employmentMatch" as "fulltime", route tag "do"'s attribute will
have "employmentMatch" as "consultant"
* The routing policy will look something like this based on the two strategies outlined earlier to
route "fulltime" employees to AWS data center and everyone else to the DO data center:

Option 1: without using host attributes
```
default route_tag = "do"
route_tag = prefix1 {
      user.service == "appx.awesomecustomer.com"
      user.employment == "fulltime"
      prefix1 := "aws"
}
route_tag = prefix2 {
      user.service == "appy.awesomecustomer.com"
      user.employment == "fulltime"
      prefix2 := "aws"
}

Option 2: using host attributes as reference data

default route_tag = "do"
route_tag = prefix {
      some host
      some tagindex
      user.service == allhosts[host].serviceName
      user.employment == allhosts[host].tags[tagindex].employmentMatch
      prefix := allhosts[host].tags[tagindex].prefix
}
```

In option 2, what we are saying in the policy is to go through both (or any number of) hosts to first select
the correct host (service being accessed), and then for the selected host, go through all the route tags 
for the host to match attribute "employmentMatch" with the user's attribute "employment".
So if a full time user accesses appx.awesomecustomer.com and matches tag "aws", nextensio will send the
user's traffic over to whichever connector advertises aws.appx.awesomecustomer.com

TODO ASHWIN: Thinking about this, now I am convinced that the host attributes are very specific
to an appGroup. Different appGroups will have different set of host attributes, so enforcing that
every host have all the host attrs defined in the attribute editor wont fly I think - anyways 
thats for the future, just thinking aloud thats all

### Attribute Editor

From the above discussion, we now know that all three entities - users, appGroups and Hosts have
"attributes", and those attributes have "values". The attributes we discussed in the above
examples are listed below collected together

* user: team, userRole, osType, osVersion, employment
* AppGroup: allowTeam, roleAllowed
* host: employmentMatch

In addition, as we discussed, host has this additional entity called a "tag" to identify and differentiate
multiple instances of the same host/service running in different places/different versions/alpha-beta etc..

The values we saw for the attributes are mostly strings and numbers, again listing them down together from
all the examples

* user: team, employment, userRole, osType: value string, osVersion: value number
* AppGroup: allowTeam, roleAllowed: value string (can also be array of strings to allow multiple teams, for eg.)
* host: employmentMatch: value string (can also be an array of strings for multiple values)

In addition, the "tag" for a host is always a string.

So the attribute editor is one single place where we define / lay down the attributes that will be 
used for users, AppGroups and hosts and what type of value these attributes will contain - they can
contain single value strings, numbers, booleans or multi value (arrays) of strings, numbers or booleans.
The value type of any attribute needs to be consistent across all records wherever it is used.
For eg., the value type of a user attribute needs to be the same for all users. An attribute cannot
have a string value for one user and a number value for another. The attribute editor helps ensure this
consistency when configuring values.
When an attribute value is defined as an array, it is not mandatory that the attribute have multiple
values. In fact, the attribute can have a single value or no value configured (a default will be assumed
if no value is entered).


Attribute Editor             
:-------------------------:
![](/architecture/policyattr/attredit.jpg)

The above picture shows the attribute editor populated with the attributes we discussed above, with
the corresponding types (string, number). We do not have any example attribute that is multi-value (array),
hence all the attributes are created selecting the "Array" radio button to "False"

The attribute editor is not saying WHAT the value is. The attribute editor does not know the values. The
value for the same attribute of course can be different for different users or hosts or AppGroups. The
customer has to ensure that correct values are entered in order to get correct results from any policies,
and hence correct and expected behavior.
The attribute editor cannot ensure correctness of data.

### Host Attributes

We show below  adding the attributes required for the AWS/Digital ocean example for host
appx.awesomecustomer.com. This section also needs an understanding of the [Routing](/architecture/routing.html)
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
is the Rego language which is getting fast industry acceptance and popularity as a simple, intuitive and
expressive policy language. The policies are configured in the picture shown below

![](/architecture/policyattr/policy.jpg)

There are separate policies, each used in a different context. Each policy has a name. The picture above
shows two policies - AccessPolicy and RoutePolicy. AccessPolicy is what controls all the "restrict appx/appy
to engineering organization" kind of activities. RoutePolicy is what controls the "fulltime employees 
get routed to AWS, consultants to Digital Ocean" kind of policies.

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
[Routing](/architecture/routing.html)
