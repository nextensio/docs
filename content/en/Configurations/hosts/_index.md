---
title: "Apps"
linkTitle: "Apps"
weight: 20
type: list
menu:
  main:
    weight: 20
---

Aug 5, 2021

## App information

Apps are the entities that users try to access. They represent the services deployed inside data
centers. Apps are grouped together, generally based on similar security requirements, and
configured as services for one or more AppGroup ID (aka connector).

Every app is expected to have a valid URL in the tenant's domain, eg., appx.awesomecustomer.com
and appy@awesomecustomer.com. A descriptive name can also be entered for the app.
The rest of the configuration per app involves attributes. In the 'Expert' mode, app attributes
are used for routing based on a Route policy by providing the app attributes file as reference data
for matching with user attributes. Nextensio does routing by providing a way to differentiate multiple
instances of the same app/service. The differentiation is done by prefixing a "route tag" to each
app ID. So if the application appx.awesomecustomer.com is to be deployed in AWS and GCP and differentiated
for routing control, the route tag "aws" can be used for AWS and "gcp" for GCP to yield
aws.appx.awesomecustomer.com and gcp.appx.awesomecustomer.com. The app attributes configuration would
then create two routes called "aws" and "gcp" and configure attributes per route. So app attributes are
actually configured per app route so that match criteria can be set per route.

As mentioned earlier, it is also possible to perform access control per app, and this is achieved
by having a special route called "deny". This route represents a blackhole and is never configured
as a service in any AppGroup ID. If the Route policy evaluates to "deny", the user flow is dropped.

In the 'Easy' mode, one has to define/create the routes (including the "deny" route) and then define
rules per route.  App attributes are not required/used in this mode.

* When there is no routing or access control for any app, the Route policy should evaluate to ""
(a null string) by default. No 'Easy' mode rule is required in this case.
* When there is only access control, the Route policy should evaluate to "deny" or "". In the 'Easy'
mode, rules need to be defined for the "deny" route.
* When there is just routing, the Route policy should evaluate to some route tag (depending on the
app) or to "" (as a catch-all in case there is no match). There should ideally be at least one
instance of the application without any route prefix configured on at least one AppGroup ID. In the
'Easy' mode, rules are required per route defined.
* When there is routing and access control for one or multiple apps in any combination, the
Route policy should evaluate to a route tag, or to "deny" or to "". If the Route policy can evaluate
to "", there must be at least one instance of the application without any route prefix configured on at
least one AppGroup ID. In the 'Easy' mode, rules are required per route including the "deny" route.

App entries (just ID and name) are currently used to identify the URL domains of the tenant and
hence are mandatory. App attributes are required only if they are to be used as reference data to a
Route policy. 
Currently, a tenant has to identify the superset of attributes that may be required for all possible
apps requiring routing or access control. However, only a subset of the attributes may be relevant
for any specific app. We will go through some scenarios in detail below.

Through this configuration editor, the following operations can be done:
* add a new app with or without routes
* delete an app with or without routes
* add new app route(s) with attribute match values to an existing app - ('Expert' mode only)
* delete app route(s)
* change any attribute value(s) for any app route(s) - ('Expert' mode only)

All the operations above should be safe, ie., they should not cause a conflict in a Route policy.
Conflict here means reference to an attribute that does not exist or whose type is different from
that expected in the policy. Hence these operations can be done at any time.

It goes without saying that any additions or changes must be carefully validated for correctness.
Incorrect values can lead to unexpected/wrong behavior. In addition, even with correct values, the
policy logic must be validated. Additions or changes in values can impact the logic assumed in the
Route policy, based on how the policy rules are written.

An important thing to note is that the policy engine does not tolerate the route tag evaluation yielding
multiple results. This is a conflict for the policy engine and it fails with an error. Taking our example
above, if the policy rule(s) end up matching the specified attributes for both "aws" and "gcp" at the
same time, the policy engine cannot decide the result. If there is a possibility of multiple matches,
the policy has to be written such that the rules provide a precedence or priority. This can be done
via the else facility - "if rule1, tag = "aws", else if rule2, tag = "gcp"".
This applies to either mode, the 'Easy' mode or the 'Expert' mode.

Conflicts of this type can happen whenever the attributes used for the different route tags (prefixes
"aws" and "gcp" in our case) do not have at least one common single-value user attribute with unique
match values specified per route.

A golden rule for app route attributes is to therefore use at least one common single-value user attribute
for matching all routes for an app (including any "deny" route) using unique attribute values for each
route. This ensures no user will match multiple routes. This common single-value user attribute does
not have to be the same for all apps, just for the routes within an app. Once this is done to ensure there
cannot be multiple route matches, it does not matter which other user attributes are used with what
sort of values (unique or overlapping) for further matching to fine-tune the selection per route.
Again, this applies whether defining rules for policy generation in the 'Easy' mode or writing a policy
directly in the 'Expert' mode.

However, when attribute values are specified for matching different attributes per app, the policy
needs to handle default values. If the number of apps requiring routing is small, the policy can be
written such that the rules are separate per app. This can help avoid the checks for default values
as each rule can be customized for the attributes to be matched for each app's routes. This caution
applies when writing policies in the 'Expert' mode.

Let's go through some examples to cover the above as well as some other considerations. Those choosing
to use the 'Easy' mode can skip these examples.

```
App = appx.awesomecustomr.com
Attribute	Route="aws"	  Route="gcp"		Route="deny"
---------------------------------------------------------------------------
typeAllowed	"employee"	  "contractor"		"vendor", "partner"
teamAllowed	-		  -			-
roleAllowed	"manager","IC"	  "IC"			"IC"
location	-		  -			-

There should never be a conflict in the above case for any of the routes since typeAllowed ensures
that. A user cannot have more than one relationship type with the tenant (either one of employee,
contractor, vendor, or partner), and each route specifies a different value to match.
Checking for user role (or any other attributes) is therefore for fine-tuning the selection.

Now let's consider the case where route attributes for our app have been specified as shown below:

App = appx.awesomecustomr.com
Attribute	Route="aws"	  Route="gcp"		Route="deny"
--------------------------------------------------------------------------
typeAllowed	"employee"	  "employee"		"vendor", "partner"
teamAllowed	-		  "sales"		-
roleAllowed	-		  -			-
location	"california"	  -			-

App = appy@awesomecustomr.com
Attribute	Route="aws"	  Route="gcp"		Route="deny"
--------------------------------------------------------------------------
typeAllowed	"employee"	  "employee"		"vendor", "partner"
teamAllowed	-		  "engineering"		-
roleAllowed	-		  -			-
location	"us-east"	  -			-
```

In the above case, let's say a user who is an "employee" in the "sales" team and based in "california"
is trying to get to appx.awesomecustomer.com.
A policy written with a rule that checks typeAllowed, teamAllowed and location will always fail since
match values are not specified for all routes for any selected attribute. Eg., teamAllowed does not
have a value specified for routes "aws" and "deny", and location does not have any match value for
routes "gcp" and "deny". Such a rule is shown below.

```
default route_tag = "gcp"
route_tag = prefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.team == data.hosts[app].routeattrs[route].teamAllowed
      input.user.location == data.hosts[app].routeattrs[route].location
      prefix := data.hosts[app].routeattrs[route].prefix
}
```

If the policy is written as follows with three rules where each rule corresponds to each route, it will
fail as well if there is a user who is a sales team employee based in california accessing
appx.awesomecustomer.com. Both rules will match, yielding "aws" and "gcp" - a conflict. Also note that
without the null value checks for teamAllowed and location in the third rule, a user who is an employee
will match routes "aws" and "gcp", giving a conflict.

```
default route_tag = "gcp"
route_tag = awsPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.team == data.hosts[app].routeattrs[route].teamAllowed
      awsPrefix := data.hosts[app].routeattrs[route].prefix
}
route_tag = gcpPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.location == data.hosts[app].routeattrs[route].location
      gcpPrefix := data.hosts[app].routeattrs[route].prefix
}
route_tag = denyPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      data.hosts[app].routeattrs[route].teamAllowed == ""
      data.hosts[app].routeattrs[route].location == ""
      denyPrefix := data.hosts[app].routeattrs[route].prefix
}
```

There is a way around the above issue by using the "else" facility to specify precedence, but it does
change the logic and should be used only if the logic is acceptable. A user who is a sales team employee
based in california accessing appx.awesomecustomer.com will always match the "gcp" route.

```
default route_tag = "gcp"
route_tag = awsPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.team == data.hosts[app].routeattrs[route].teamAllowed
      awsPrefix := data.hosts[app].routeattrs[route].prefix
} else = gcpPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.location == data.hosts[app].routeattrs[route].location
      gcpPrefix := data.hosts[app].routeattrs[route].prefix
} else = denyPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      denyPrefix := data.hosts[app].routeattrs[route].prefix
}
```

The above examples show that when an attribute is selected for matching a route, match values should be
specified for all routes present. Also, these match values need to be unique for at least one attribute
(to ensure there is no conflict in matching).

In all the examples above, the rules were written to be app independent. So what happens if the
attributes to match vary from app to app ? It is similar to the case where different attributes are
selected per route for any given app.

If the number of apps is small, a simpler solution may be to have separate rules per app/service.
For eg., assume we use typeAllowed and teamAllowed for appx.awesomecustomer.com, and typeAllowed and
location for appy@awesomecustomer.com (we'll drop the "deny" route for now).

```
default route_tag = "gcp"
route_tag = appxPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      data.hosts[app].routeattrs[route].serviceName == "appx.awesomecustomer.com"
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.team == data.hosts[app].routeattrs[route].teamAllowed
      appxPrefix := data.hosts[app].routeattrs[route].prefix
}
route_tag = appyPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      data.hosts[app].routeattrs[route].serviceName == "appy.awesomecustomer.com"
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.location == data.hosts[app].routeattrs[route].location
      appyPrefix := data.hosts[app].routeattrs[route].prefix
}
```

An alternate way to achieve the above without hardcoding app names would be to write the policy rules
as follows :

```
default route_tag = "gcp"
route_tag = appxPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.team == data.hosts[app].routeattrs[route].teamAllowed
      appxPrefix := data.hosts[app].routeattrs[route].prefix
} else = appyPrefix {
      some app
      some route
      input.user.service == data.hosts[app].routeattrs[route].serviceName
      input.user.relationshipType == data.hosts[app].routeattrs[route].typeAllowed
      input.user.location == data.hosts[app].routeattrs[route].location
      appyPrefix := data.hosts[app].routeattrs[route].prefix
}
```
However, as the number of apps requiring routing and/or access control gets large, the above method
can get cumbersome and will become unmanageable beyond a certain point. The only way around is to write
a compact, generalized policy using reference data. But we've seen above that writing a generalized policy
requires matching the same set of attributes for every app. But different apps may require different
criteria. If we are selective with the attributes chosen per app, then a generalized policy fails
unless default values for unused attributes are not considered. However, considering default values for
unused attributes is non-trivial. The way around this is to convert unused attributes into "dont-cares"
and that can be done by setting the attribute match criteria to have all possible values that a matching
user attribute can have. Let's take an example to illustrate this:
```
  userRole attribute	roleSelect attribute
  -------------------------------------------
  employee
  consultant
  contractor		["employee","consultant","contractor","vendor","partner"]
  vendor
  partner
```
So whereas a user would have one of the attributes shown in the left column, an app route
that does not care about the value of this attribute can set the match criteria to all the values
shown in the right column for its roleSelect attribute. roleSelect would be defined as array
type. This will allow matching any user unlike a default value which will not match any user.

Where a user attribute has a small number of possible values, all these possible values should
be defined as the match criteria for the corresponding app route attribute that is a "don't-care".
Of course one needs to keep track of the addition of any new value to the user attribute and reflect
that addition in the app route attribute value as well.

If a user attribute has a large number of possible values, the above task may get difficult.
In such a case, the customer can consider grouping apps based on attribute groups where the
attribute groups are separated based on attributes with a large set of possible values. The
common elements in each group would be the attributes with a small set of possible values.
The policy rules can then be separated out for each attribute group but still written in a
generalized way to keep the policy compact.

Unless a change is very urgent (say the change is required to address a security issue related to
app access-control), it is recommended that changes be batched together and done once or twice a day
for better controls and validations. Of course this is just a suggestion and it is up to the tenant
to decide what works best for them.

For addition of new attributes, deleting an attribute, or changing the name or type of an attribute,
refer to the [Attribute Editor section](../configurations/attributeeditor.html)


App definition             
:-------------------------:
![](/docs/configurations/hosts/host_add.jpg)

* App: is a valid URL for the application

* Name: A descriptive name for the application


App routing config
:-------------------------:
![](/docs/configurations/hosts/host_routes.jpg)

The example above shows an app defined with two routes - "aws" and "do", and each route has
a separate set of attributes. A route tag or prefix is just any string. For details on how its used in
routing, refer above or [routing](/docs/architecture/routing.html)

For attributes, please refer to the [overview on policy and attributes](/docs/architecture/policyattr.html) 
and details about [configuring attribute editor](../configurations/attributeeditor.html) 
