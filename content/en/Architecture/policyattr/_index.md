---
title: "Attributes and Policies"
linkTitle: "Attributes and Policies"
weight: 10
description: >
---

## Introduction 

Policies play an important role in Nextensio. Lets look at some sample use cases, continuing with
the awesomecustomer.com App-X and App-Y examples from the [Architecture](/docs/architecture.html) section.
Once we run through the sample use cases, we will have a fair idea of what attributes and 
policies are and how to use them. 

A nextensio policy basically takes an input and evaluates it using rules that implement some logic, either
for routing or access control. The input consists of user attributes plus either the name of the
App being accessed (in case of a Route policy) or the AppGroup ID being accessed (in case of the Access
policy). The policy rules are written in a language called Rego, an industry standard developed as part
of the Open Policy Agent (OPA). Nextensio provides an 'Easy' mode that hides the OPA policy language
and instead allows creation of a policy via simple rules through a graphical interface. Each rule
consists of one or more expressions where an expression matches a specific user attribute to one or
more values.

At a high level, there are two strategies for implementing any policy:
* hardcode the values to match within the rules so that the policy is self contained
* reference external data in the rules to get the values to match. The reference data may be a file with
App attribute records or a file with AppGroup attribute records

The 'Easy' mode adopts the first strategy, doing away with the need to maintain App and AppGroup attributes
and allowing the policy to be defined via higher level rules.
The 'Expert' mode adopts the second strategy, giving the user the full flexibility of using OPA's
Rego language to write the policies and reference App and AppGroup attribute files.

Which strategy is chosen depends entirely on which one the customer is more comfortable with, driven by
familiarity with Rego and the scale of App and AppGroup IDs. We expect that customers will start with the
'Easy' mode and migrate to the 'Expert' mode later when they become more familiar with attributes and the
policy language.


### Access control by Appgroup IDs (Connectors)

A data center may have one or more connectors identified by their AppGroup ID. Access into a
data center can be controlled at the connector level using an Access policy. The Access policy
may or may not use AppGroup attributes as reference data, as mentioned above.


#### Case1: Restrict access by user team and role

Let's assume data center DCA is running engineering applications and data center DCB is running
HR applications. We therefore want to restrict access to AppGroup-DCA to those in "engineering"
team, and restrict access to AppGroup-DCB to users with the role of "manager" as well as users
in the "hr" team. Note that the "engineering" team can have "manager" - they should be able to
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
<any-other-team> "non-manager"	     deny	  deny
```

Option 1 - without using AppGroup attributes

In the 'Easy' mode, the user creates the three rules shown below
by picking the user attribute and operator for each expression from drop-down
lists, and entering the match values for each. Once all the rules are entered,
clicking a 'Generate Policy' button at the top right corner of the screen generates
the policy and asks for a confirmation whether to apply it.

```Policies
allow = is_allowed
default is_allowed = false
is_allowed {
        # Rule 1 with two expressions (or statements)
        user.destination == "AppGroup-DCA"
	user.team == "engineering"
}
is_allowed {
        # Rule 2 with two expressions (or statements)
        user.destination == "AppGroup-DCB"
	user.userRole == "manager"
}
is_allowed {
        # Rule 3 with two expressions (or statements)
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

The above option 2 is for the 'Expert' mode. It involves writing the policy directly
and using AppGroup attributes.
The destination AppGroup ID is available in the AppGroupID attribute.
The 'some AppGroup' line is to iterate through multiple AppGroup IDs (we have two here) to ensure the
allowTeam or the roleAllowed attribute checks are done with the correct AppGroup ID record. 


#### Case 2: Restrict users based on device posture (o/s version) 

The user agent on the user devices informs nextensio gateways about the security/device posture
information of the device, and those are available as "inbuilt" attributes. Such attributes are
distinguished by an underscore ('_') prefix in the attribute name.

In the 'Easy' mode, the rules would be defined as follows :

```
Rule-1
    user._osType != "macOS"

Rule-2
    user._osType == "macOS"
    user._osVersion >= 10.15
```

In the 'Expert' mode, the policy could be written more compactly as follows

```
allow = not deny
deny {
	user._osType == "macOS"
	user._osVersion < 10.15
}
```

Since the deny case is very specific, the rule can be written for deny and the allow value derived via
negation. Writing the rule for allow requires checking for other osTypes, making the policy slightly bigger.
Note that this is a very simple case where the osType and osVersion values are hardcoded in the
policy and applies to all AppGroup IDs. This may not be desirable in more complex cases, or where
the criteria needs to vary depending on the AppGroup ID (eg., some AppGroup IDs restricted to users of
very old macOS versions and some AppGroup IDs restricted to users of very old Windows versions).
The use of AppGroup attributes as reference data may be considered for more complex cases and the
policy written accordingly.


### Routing control by App


#### Case 1: Routing to optimize data center cost

Lets say that DCA is in AWS and DCB in Digital Ocean and both data centers run the same applications.
Assuming AWS is more expensive than digital ocean, the goal here is to minimize aws usage as much as
possible to save on cost - so restrict it to fulltime employees. Goal, therefore, is to have fulltime
employees go to AWS, consultants go to digital ocean.

So lets say the App in digital ocean is hosted as do.appx.awesomecustomer.com and 
do.appy.awesomecustomer.com and in aws its aws.appx.awesomecustomer.com and 
aws.appy.awesomecustomer.com. Of course the user does not know any of this, nor does it change the
names of applications in the data centers. User is going to access the App as appx.awesomecustomer.com
or appy.awesomecustomer.com. The prefixes of "aws" and "do" differentiate the two instances of
each App only within the nextensio network.

So there are two connectors obviously - one running in AWS and one in Digital Ocean. The one for
AWS is configured with services "aws.appx.awesomecustomer.com,aws.appy.awesomecustomer.com" and
the one for Digital Ocean with "do.appx.awesomecustomer.com,do.appy.awesomecustomer.com"

* We will say that all users have an attribute called "employment" with values "fulltime" or "consultant"
* We will say that the two Apps will each have two route "tags" - "aws" and "do"  
* We will also say that each route tag will have an associated attribute called "employmentMatch" with
a value of either "fulltime" or "consultant"
Route tag "aws"'s attribute will have "employmentMatch" as "fulltime", route tag "do"'s attribute will
have "employmentMatch" as "consultant"
* The routing policy will look something like this based on the two strategies outlined earlier to
route "fulltime" employees to AWS data center and everyone else to the DO data center:

Option 1: without using App attributes
```
default route_tag = "do"
route_tag = prefix1 {
      # Rule-1 statements in 'Easy' mode
      user.service == "appx.awesomecustomer.com"
      user.employment == "fulltime"
      prefix1 := "aws"
}
route_tag = prefix2 {
      # Rule-2 statements in 'Easy' mode
      user.service == "appy.awesomecustomer.com"
      user.employment == "fulltime"
      prefix2 := "aws"
}

Option 2: using App attributes as reference data ('Expert' mode)

default route_tag = "do"
route_tag = prefix {
      some host
      some tagindex
      user.service == allhosts[host].serviceName
      user.employment == allhosts[host].tags[tagindex].employmentMatch
      prefix := allhosts[host].tags[tagindex].prefix
}
```

In option 2, what we are saying in the policy is to go through both (or any number of) Apps to first select
the correct App (service being accessed), and then for the selected App, go through all the route tags 
for the App to match attribute "employmentMatch" with the user's attribute "employment".
So if a full time user accesses appx.awesomecustomer.com and matches tag "aws", nextensio will send the
user's traffic over to whichever connector advertises aws.appx.awesomecustomer.com


### Attribute Editor

From the above discussion, we now know that all three entities - users, appGroups and Apps have
"attributes", and those attributes have "values". The attributes we discussed in the above
examples are listed below collected together

* user: team, userRole, _osType, _osVersion, employment
* AppGroup: allowTeam, roleAllowed
* App: employmentMatch

In addition, as we discussed, App has this additional entity called a "tag" to identify and differentiate
multiple instances of the same application running in different places/different versions/alpha-beta etc..

The values we saw for the attributes are mostly strings and numbers, again listing them down together from
all the examples

* user: team, employment, userRole, _osType: value string, _osVersion: value number
* AppGroup: allowTeam, roleAllowed: value string (can also be array of strings to allow multiple teams, for eg.)
* App: employmentMatch: value string (can also be an array of strings for multiple values)

In addition, the "tag" for an App is always a string.

So the attribute editor is one single place where we define / lay down the attributes that will be 
used for users, AppGroups and Apps and what type of value these attributes will contain - they can
contain single value strings, numbers, booleans or multi value (arrays) of strings, or numbers.
The value type of any attribute needs to be consistent across all records wherever it is used.
For eg., the value type of a user attribute needs to be the same for all users. An attribute cannot
have a string value for one user and a number value for another. The attribute editor helps ensure this
consistency when configuring values.
When an attribute value is defined as an array, it is not mandatory that the attribute have multiple
values. In fact, the attribute can have a single value or no value configured (a default will be assumed
if no value is entered).


Attribute Editor             
:-------------------------:
![](/docs/architecture/policyattr/attredit.jpg)

The above picture shows the attribute editor populated with the attributes we discussed above, with
the corresponding types (string, number). We do not have any example attribute that is multi-value (array),
hence all the attributes are created selecting the "Array" radio button to "False"

The attribute editor is not saying WHAT the value is. The attribute editor does not know the values. The
value for the same attribute can of course be different for different users or Apps or AppGroups. The
customer has to ensure that correct values are entered in order to get correct results from any policies,
and hence correct and expected behavior.
The attribute editor cannot ensure correctness of data.

### App Attributes

We show below  adding the attributes required for the AWS/Digital ocean example for application
appx.awesomecustomer.com. This section also needs an understanding of the [Routing](/docs/architecture/routing.html)
section to get a complete picture

Add App             
:-------------------------:
![](/docs/architecture/policyattr/host_add.jpg)

Edit App             
:-------------------------:
![](/docs/architecture/policyattr/host_edit.jpg)

Add attributes for tags             
:-------------------------:
![](/docs/architecture/policyattr/hostattr_edit.jpg)

Similarly we can add the second tag "do" with attribute employmentType set to "consultant"

### User Attributes

Add/Modify attributes             
:-------------------------:
![](/docs/architecture/policyattr/userattr_edit.jpg)

The picture shows the attributes we discussed in the examples above, added to the "admin" user

### AppGroup Attributes

Add/Modify attributes             
:-------------------------:
![](/docs/architecture/policyattr/appattr_edit.jpg)

The picture shows the attributes we discussed in the examples above, added to the "appxappy" appgroup

## Policies

We saw snippets of how a nextensio policy is written in the above examples. The nextensio policy language
is the Rego language which is getting fast industry acceptance and popularity as a simple, intuitive and
expressive policy language. Nextensio makes it even simpler by providing an 'Easy' mode to write policies
using rules. A rule consists of one or more expressions, where each expression selects a user attribute,
an operator (such as ==, !=, <, > etc), and one or multiple values to match. The expressions within a rule
are logically ANDed together while multiple rules are logically ORed together. The policies generated via
rules in the 'Easy' mode of course have some restrictions and cannot utilize the full capabilities of the
Rego language. But as customers get experience with attributes and simple policies, we expect that they
will be able to migrate to the 'Expert' mode to develop more complex and powerful policies.


In 'Expert' mode, the policies are configured as shown below. Some templates to aid in writing policies are
also shown.

![](/docs/architecture/policyattr/policy.jpg)

There are separate policies, each used in a different context. Each policy has a name. The picture above
shows two policies - AccessPolicy and RoutePolicy. AccessPolicy is what controls all the "restrict appx/appy
to engineering organization" kind of activities. RoutePolicy is what controls the "fulltime employees 
get routed to AWS, consultants to Digital Ocean" kind of policies.


### Policy Templates

Below are some templates for policies that can be copy + pasted and adapted. The templates for each type
of policy provide a basic framework using one of two methods :
* Template1 is based on using all match values encapsulated within the policy itself (ie., there is no
reference configuration data required). Looked at another way, attributes and their match values that
would be entered into the AppGroup or App configuration are instead entered directly into match statements
in the corresponding policies. This template can be used for small deployments. 
* Template2 is based on using match values from configured data. By taking attribute match values from a
configuration file instead being specified directly in a policy, it enables writing policies in a compact
and generic way. This template can be used for large deployments but requires some expertise.


#### Access Policy Template1

```
package app.access
allow = is_allowed
default is_allowed = false

# Create more rules as needed if logic is rule1 OR rule2 OR rule3 OR rule4 .... for a bundle ID
# Rule # 1 for bundle ID1
is_allowed {
    input.bid == "$BUNDLE-ID1"                         # bundle ID == AppGroup ID
    # if using multiple rules for this bundle ID, ensure at least one attribute value match is
    # common in all rules and matches to true for only one rule
    # change == to >, >=, <, <= ... as needed
    input.user.$ATTRIBUTE1 == "$STRING-VALUE"           # string value
    input.user.$ATTRIBUTE2 == $INTEGER-VALUE            # integer value
    # add more expressions as needed
}
# Rule # 2 for bundle ID1
is_allowed  {
    input.bid == "$BUNDLE-ID1"                         # bundle ID == AppGroup ID
    input.user.$ATTRIBUTE1 == "$STRING-VALUE"           # string value
    input.user.$ATTRIBUTE3 == $INTEGER-VALUE            # integer value
    # add more expressions as needed using guidelines above
}

# Similarly, create one or more rules as needed for more bundle IDs
# Rule # 1 for bundle ID2
is_allowed {
    input.bid == "$BUNDLE-ID2"                         # bundle ID == AppGroup ID
    input.user.$ATTRIBUTE4 == "$STRING-VALUE"           # string value
    input.user.$ATTRIBUTE5 == $INTEGER-VALUE            # integer value
    # add more expressions as needed using guidelines above
}
```

#### Access Policy Template2

```
package app.access
allow = is_allowed
default is_allowed = false

# Create more rules as needed if logic is rule1 OR rule2 OR rule3 OR rule4 .... across all bundles
# Rule # 1
is_allowed {
    some bundle
    input.bid == data.bundles[bundle].bid     # to ensure checks below are for target bundle ID
    # if using multiple rules, ensure at least one attribute value match is common in all rules and
    # matches to true for only one rule
    # if AppGroup attribute is an array, use $APPGROUP-ATTRIBUTE[_] to match with any value
    # replace == by >=, <=, <, >, != as needed
    input.user.$ATTRIBUTE1 == data.bundles[bundle].$APPGROUP-ATTRIBUTE1
    input.user.$ATTRIBUTE2 == data.bundles[bundle].$APPGROUP-ATTRIBUTE2
    # add more match expressions as needed
}
# Rule # 2
is_allowed  {
    some bundle
    input.bid == data.bundles[bundle].bid     # to ensure checks below are for target bundle ID
    # if AppGroup attribute is an array, use $APPGROUP-ATTRIBUTE[_] to match with any value
    input.user.$ATTRIBUTE1 == data.bundles[bundle].$APPGROUP-ATTRIBUTE1
    input.user.$ATTRIBUTE3 == data.bundles[bundle].$APPGROUP-ATTRIBUTE3
    # add more expressions as needed using guidelines above
}
```

#### Route Policy Template 1

```
package user.routing
default route_tag = ""

# Create more rules as needed for route tags for each App
# Rule # 1 for route tag1 for host1
route_tag = rtag {
    input.host == "$HOST1"
    # if using multiple rules for this App route, ensure at least one attribute value match
    # is common in all rules and matches to true for only one rule
    # replace == by >=, <=, <, >, != as needed
    input.user.$ATTRIBUTE1 == "$STRING-VALUE"    # string value
    input.user.$ATTRIBUTE2 == $INTEGER-VALUE     # integer value
    # add more expressions as needed
    rtag := "$ROUTE-TAG1"
}
# Rule # 2 for route tag2 for host1
route_tag = rtag  {
    input.host == "$HOST1"
    input.user.$ATTRIBUTE1 == "$STRING-VALUE"    # string value
    input.user.$ATTRIBUTE3 == $INTEGER-VALUE     # integer value
    # add more expressions as needed using guidelines above
    rtag := "$ROUTE-TAG2"
}

# Rule # 1 for route tag3 for host2
route_tag = rtag {
    input.host == "$HOST2"
    input.user.$ATTRIBUTE1 == "$STRING-VALUE"    # string value
    input.user.$ATTRIBUTE4 == $INTEGER-VALUE     # integer value
    # add more expressions as needed using guidelines above
    rtag := "$ROUTE-TAG3"
}
# Rule # 2 for route tag4 for host2
route_tag = rtag {
    input.host == "$HOST2"
    input.user.$ATTRIBUTE1 == "$STRING-VALUE"    # string value
    input.user.$ATTRIBUTE5 == $INTEGER-VALUE     # integer value
    # add more expressions as needed using guidelines above
    rtag := "$ROUTE-TAG4"
}
```

#### Route Policy Template 2

```
package user.routing
default route_tag = ""

route_tag = rtag {
    some idx
    input.host == data.hosts[idx].host
    some route
    # if using multiple rules, ensure at least one attribute value match is common in all rules and
    # matches to true for only one rule
    # if host route attribute is an array, use $HOSTROUTE-ATTRIBUTE[_] to match with any value
    # replace == by >=, <=, <, >, != as needed
    input.user.$ATTRIBUTE1 == data.hosts[idx].routeattrs[route].$HOSTROUTE-ATTRIBUTE1
    input.user.$ATTRIBUTE2 == data.hosts[idx].routeattrs[route].$HOSTROUTE-ATTRIBUTE2
    # add more match expressions as needed
    rtag := data.hosts[idx].routeattrs[route].tag
}
```

### Policy Examples Library

The goal here is to build up a library of sample policies for different use cases so as to give a gist of
how the policies can be written. These sample policies can be selected and adapted as per the needs of each
Nextensio customer. Policies can be written in different ways, based on the style and skills of the author,
and there is no one right way to write any policy. The policies are not unlike any software program written
in your favorite programming language such as C, Go, Rust, Javascript, Python, etc even though Rego, the
policy language, is simpler and at a higher level than the other languages.

Besides giving a gist of how to write Nextensio policies, this library will also try to cover use cases
to illustrate when/where such policies can be used. We will obviously not be able to cover all possible
use cases, and these use cases will keep evolving over time.

As with the templates above, these sample policies for each use case are based on each of the two methods :
* using all match values encapsulated within the policy itself (ie., there is no reference configuration
data required). Looked at another way, attributes and their match values that would be entered into the
AppGroup or App configuration are instead entered directly into match statements in the corresponding
policies. These samples can be used for small deployments. 
* using match values from configured data. By taking attribute match values from a configuration file
instead being specified directly in a policy enables writing policies in a compact and generic way. These
samples can be used for large deployments.

These sample policies are also accompanied by the user attributes and associated values assumed. For policies
using AppGroup or App attributes configuration data, those attributes together with match values assumed
are included. Since Nextensio policies deal with attributes, they have to go hand-in-hand for full context.


#### Access Policy Samples

##### Merger and acquisition use case

An online photo editing software company, EZphoto, acquired an image processing tool company called iPixel.
The data centers of EZphoto and iPixel use overlapping IP subnet ranges. EZphoto needs a quick IT solution
to enable employees of EZphoto to access the data centers of both EZphoto and iPixel while limiting iPixel
employees to access only the iPixel data center.

The logic for this can be depicted in multiple ways. Translating the above intent, we can depict it as

```
IF USER DOMAIN == EZPHOTO.COM AND DESTINATION DOMAIN == EZPHOTO.COM
    ALLOW ACCESS
IF USER DOMAIN == EZPHOTO.COM AND DESTINATION DOMAIN == IPIXEL.COM
    ALLOW ACCESS
IF USER DOMAIN == IPIXEL.COM AND DESTINATION DOMAIN == EZPHOTO.COM
    DENY ACCESS
IF USER DOMAIN == IPIXEL.COM AND DESTINATION DOMAIN == IPIXEL.COM
    ALLOW ACCESS
```

The above can be condensed a bit to

```
IF USER DOMAIN == EZPHOTO.COM
    ALLOW ACCESS
IF USER DOMAIN == IPIXEL.COM AND DESTINATION DOMAIN == IPIXEL.COM
    ALLOW ACCESS
ELSE
    DENY ACCESS
```

This can be condensed even further to

```
IF DESTINATION DOMAIN == EZPHOTO.COM AND USER DOMAIN != EZPHOTO.COM
     DENY ACCESS
ELSE 
    ALLOW ACCESS
```

If the logic for this simple intent can be expressed in so many ways, a policy to implement that logic can
also be written in many ways. Which way is chosen will depend on the author and maybe reviewers (if there
are any), but may also depend on what other granular access restrictions EZphoto already has in place for
their users and any other new ones they would like to bring in for iPixel users.

To address the above intent, we would need at least one user attribute:
1. businessUnit - "ezphoto" or "ipixel"

Also, assume
IPixel data center is accessed via Connector called ipixel-dc1.com and 
EzPhoto data center is accessed via Connector called ezphoto-dc1.com

The access policy using template1 format could be written as follows. Note that Appgroup attributes
are not required here.

```
package app.access
allow = is_allowed

default is_allowed = false
is_allowed  {
    # Allow ezphoto employee to access either data ccenter
    input.user.businessUnit == "ezphoto"
}
is_allowed  {
    # Allow ipixel employee to access data center behind ipixel-dc1 only 
    input.bid == "ipixel-dc1.com"
    input.user.businessUnit == "ipixel"
}
```

A more elegant but slightly longer policy could look like this. This policy uses functions to make it more
readable.

```
package app.access
allow = is_allowed

default is_allowed = false
is_allowed  {
    # Allow ezphoto employee to access either data ccenter
    user_is_ezphoto_employee
}
is_allowed  {
    input.bid == "ipixel-dc1.com"
    user_is_ipixel_employee
}

default user_is_ezphoto_employee = false
user_is_ezphoto_employee {
    input.user.businessUnit == "ezphoto"
}

default user_is_ipixel_employee = false
user_is_ipixel_employee {
    input.user.businessUnit == "ipixel"
}
```

Now let's consider how the above policy would look if we were to define AppGroup attributes for our two
Connectors and assign them some match values for use in the policy. 

Assume we define the attribute buAllowed per Connector. Note that it is defined as an array of string
values so that we can match against multiple values. 

```
Connector 	 Attribute      Match values		Comments
-------------------------------------------------------------------------------------------------------
ezphoto-dc1.com	 buAllowed	["ezphoto"]		Only users with businessUnit = "ezphoto"
ipixel-dc1.com   buAllowed	["ipixel","ezphoto"]	Users with businessUnit = "ezphoto" or "ipixel"


package app.access
allow = is_allowed

default is_allowed = false
is_allowed  {
	some bundle
	# bundle is used to iterate through the multiple Connectors (AppGroups)
	input.bid == data.bundles[bundle].bid
        # match attribute used to distinguish EZphoto from iPixel employees
	input.user.businessUnit == data.bundles[bundle].buAllowed[_]
}
```

Here we see a very compact policy with a single rule that leverages match values from the AppGroup
attributes configuration.

None of the policies above have any reference to either data center's IP subnet. Nextensio does not
care about the IP subnets and allows control of access to each data center even if they have overlapping
IP subnets.


#### Route Policy Samples

##### Traffic management for service quality based on user tier

A web services company wants to manage user traffic based on user tier - whether user is a paying
customer or a non-paying customer (in a free tier). The company has two data centers for the same
services, one hosted in a premium cloud provider and the other hosted in a cheaper cloud provider.
Paying customers are to be directed to the data center in the premium cloud provider whereas the
non-paying customers are to be directed to the cheaper cloud provider.The premium cloud provider
provides faster processors with more cores and more memory as well as a richer set of storage options.

Let's assume the web services company provides its services at superduper.com.
Let's also assume the Nextensio instance of this service for paying customers will be prem.superduper.com
and the instance for non-paying customers will be free.superduper.com. Note that these two instances
will be totally transparent to all users; users only see and access superduper.com. Both data centers
run exactly the same application hosted at the same URL and need to make absolutely no changes in their
data centers.

The logic for this intent can be written as

```
IF USER IS PAYING CUSTOMER
    ROUTE TO prem.superduper.com
ELSE
    ROUTE TO free.superduper.com
```

User attribute needed:
1. userTier = "free" or "premium"

The Route policy using template1 could be written as follows. Note that the App attributes are not
required here.

```
package user.routing
default route_tag = "free"

route_tag = rtag {
    input.host == "superduper.com"
    input.user.userTier == "premium"
    rtag := "prem"
}
```

Now let's consider how the above policy would look if we were to define App attributes for our two
instances of superduper.com and assign them some match values for use in the policy. 

Assume we define the attribute userTierSelect per route/instance tag of application superduper.com.

```
Tag 	 Attribute        Match values	  Comments
-------------------------------------------------------------------------------------------------------
free	 userTierSelect	  "free"	  Non-paying customers/users
prem   	 userTierSelect	  "premium	  Paying customers/users


package user.routing
default route_tag = "free"

route_tag = rtag {
    some hostidx
    some route
    # ensure we are matching attributes for the correct host
    input.host == data.hosts[hostidx].host
    input.user.userTier == data.hosts[hostidx].routeattrs[route].userTierSelect
    rtag := data.hosts[hostidx].routeattrs[route].tag
} 
```

The above use case is obviously a very simple one. If there are other applications besides
superduper.com where such traffic management needs to be done, the second policy example may provide
a more compact policy, since the first example will require replicating the rules for each App.
There may also be other traffic management requirements to be applied to other Apps as well. So consider
these and scaling factors befoe deciding which way to go as well as what value to chose for the default 
route tag. 


##### Traffic management to distribute workload during peak hours

A network service company has two data centers providing the same services. One data center is located
in Equinix and the other in AWS. The company wants to prioritize resource usage and distribute workload 
during peak hours by directing all contractors, partners, and consultants as well as everyone from the 
sales team to AWS during that time. All other users will be directed to the Equinix data center. 
During non-peak hours, all users will be directed to the Equinix data center. Peak hours are defined as
9am to 12pm.

The logic for the above intent can we written as follows:

```
IF TIME OF DAY BETWEEN 9 AM TO 12 PM  AND  USER GROUP == "contractor" OR "consultant" OR partner"
    ROUTE to AWS-DATA-CENTER
ELSE IF TIME OF DAY BETWEEN 9 AM TO 12 PM  AND  USER TEAM == "sales"
    ROUTE TO AWS-DATA-CENTER
ELSE
    ROUTE TO EQUINIX-DATA-CENTER
```

User attributes needed for a policy are:
1. userGroup = "employee" or "contractor" or "consultant" or "partner"
2. userTeam = "sales" + many others ("hr", "marketing", "finance", "support", "manufacturing", ...)

Assume the services provided by the company are at spiffyservice.com.
We will designate the AWS instance of the service as aws.spiffyservice.com
and the Equinix instance as equinix.spiffyservice.com.


The routing policy based on template1 can look like this.

```
package user.routing
default route_tag = "equinix"

# match for traffic to be sent to AWS :
# traffic from contractors, consultants or vendors or sales team during peak hours
route_tag = rtag {
    is_peak_time
    input.host == "spiffyservice.com"
    is_nonemployee
    rtag := "aws"
} 
route_tag = rtag {
    is_peak_time
    input.host == "spiffyservice.com"
    # identify sales team traffic during peak time
    input.user.userGroup == "employee"
    input.user.userTeam == "sales"
    rtag := "aws"
}

default is_nonemployee = false
is_nonemployee {
    input.user.userGroup == "contractor"
}
is_nonemployee {
    input.user.userGroup == "consultant"
}
is_nonemployee {
    input.user.userGroup == "partner"
}

default is_peak_time = false
is_peak_time {
    ctime := time.clock([time.now_ns(), ""])
    #Note: hours are in UTC here but can be made "Local" as well.
    ctime[0] >= 21 
    ctime[0] < 24
}
```

Note that in the second rule, we have a check for userGroup == "employee". Without this check, if we
were to check just userTeam == "sales", there could be a conflict between the two rules if any contractor,
consultant or partner are also part of the sales team. This would result in a policy evaluation error.
The userGroup check ensures that only one of the two rules evaluates to true at any time. This is a very
important aspect that has to be considered when defining the rules. Match criteria and values have to be
carefully considered for connected rules to avoid conflicts.

Now let's consider how the above policy would look if we were to define App attributes for our two
instances of spiffyservice.com and assign them some match values for use in the policy. 

Peak time logic:

```
userGroup	userTeam	tag
---------------------------------------
employee	sales		aws
employee	not sales	equinix
contractor	any		aws
consultant	any		aws
partner		any		aws
```

Assume we define the attributes userGroupSelect and userTeamSelect per route/instance tag of
spiffyservice.com. To achieve the above logic, we would have to configure match values as follows:

```
Tag 	 Attribute		Match values	
-------------------------------------------------------------
aws	 userGroupSelect	["contractor","consultant","partner"]
	 userTeamSelect		["sales"]
equinix  userGroupSelect	[""]
	 userTeamSelect		[""]

package user.routing
default route_tag = "equinix"

# match for traffic to be sent to AWS :
# traffic from nonemployees or sales team during peak hours
route_tag = rtag {
    is_peak_time
    some hostidx
    input.host == data.hosts[hostidx].host
    some route
    # identify nonemployee traffic during peak time
    input.user.userGroup == data.hosts[hostidx].routeattrs[route].userGroupSelect[_]
    # return tag from route entry matched
    rtag := data.hosts[hostidx].routeattrs[route].tag
} 
route_tag = rtag {
    is_peak_time
    some hostidx
    input.host == data.hosts[hostidx].host
    some route
    # identify sales team traffic during peak time
    input.user.userGroup != data.hosts[hostidx].routeattrs[route].userGroupSelect[_]
    input.user.userTeam == data.hosts[hostidx].routeattrs[route].userTeamSelect[_]
    # return tag from route entry matched
    rtag := data.hosts[hostidx].routeattrs[route].tag
}

default is_peak_time = false
is_peak_time {
    ctime := time.clock([time.now_ns(), ""])
    #Note: hours are in UTC here but can be made "Local" as well.
    ctime[0] >= 21 
    ctime[0] < 24
}
```

In the rules above, the attribute matches have focused on the logic for identifying the traffic
to be sent to AWS since the default is set to equinix. In reality, there may be other Apps
and they may not all be located in the Equinix data center. When the default data center for
different Apps varies, it would be better to have the default route tag as "" as well as have
one instance of each App without any route tag prefix. For eg., we could have left the Equinix
instance as spiffyservice.com and designated the AWS instance as aws.spiffyservice.com.


##### Service migration using canary deployment

An insurance company wants to roll out a new version of a pricing application. The instance
of the new version will be deployed in AWS at its Virginia site. The current version of the
pricing application runs in a data center located at Equinix in Chicago.
To start testing the new version of the pricing application, the company wants just the sales
team members located in the Boston region and running PCs with Windows10 to be able to access
the application. All other users should continue to access the application instance in Equinix.
The goal is to add more users to use the new version based on the experience of the Boston sales
team Windows10 users.

The logic for this intent can be phrased as

```
IF USER TEAM  ==  SALES  AND  USER LOCATION  ==  BOSTON  AND  USER OSTYPE ==  WINDOWS10
    ROUTE TO AWS-VIRGINIA
ELSE
    ROUTE TO EQUINIX-CHICAGO
```

The user attributes we would need are:
1. userTeam = "sales", and many others
2. userLocation = "boston" and many others
3. userOsType = "windows10", and some others

Assume the pricing application is called salespricing@bestinsurance.com. The current version
will be available in the instance called salespricing@bestinsurance.com whereas the new version
will be available in the instance called new.salespricing@bestinsurance.com.


A policy based on template1 (not using App attributes) can look like this:

```
package user.routing
default route_tag = ""

route_tag = rtag {
    input.host == "salespricing@bestinsurance.com"
    input.user.userTeam == "sales"
    input.user.userLocation == "boston"
    input.user.userOsType == "windows10"
    rtag := "new"
}
```

Now assume we configure the following App attributes for route tag "new" for salespricing@bestinsurance.com:
1. userTeamSelect = ["sales"]
2. userLocationSelect = ["boston"]
3. userOsTypeSelect = ["windows10"]

```
Tag 	 Attribute		Match values	
-------------------------------------------------------------
new	 userLocationSelect	["boston"]
	 userTeamSelect		["sales"]
	 userOsTypeSelect	["windows10"]
```

All the attributes are defined as array type so that more values can be added for selection
in future. For eg., other locations besides Boston can be added. Or maybe MacOs users, and so on.
A policy based on template2 using these attributes would look like this :

```
package user.routing
default route_tag = ""

route_tag = rtag {
    some hostidx, route
    input.host == data.hosts[hostidx].host   
    input.user.userTeam == data.hosts[hostidx].routeattrs[route].userTeamSelect[_]
    input.user.userLocation == data.hosts[hostidx].routeattrs[route].userLocationSelect[_]
    input.user.userOsType == data.hosts[hostidx].routeattrs[route].userOsTypeSelect[_]
    rtag := data.hosts[hostidx].routeattrs[route].tag
}
```

## Next 

Having understood attributes and policies, let us see how they are used in making routing decisions specifically -
[Routing](/docs/architecture/routing.html)
