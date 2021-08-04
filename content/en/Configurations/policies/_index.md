
---
title: "Policies"
linkTitle: "Policies"
weight: 20
type: list
menu:
  main:
    weight: 20
---

### Policy overview

A nextensio policy is a set of rules to match attributes and evaluate a result. The result may be
a binary true/false (allow/deny), or a string of characters, depending on the type of policy. The
policy is written in a language called Rego and processed by a policy engine called Open Policy
Agent (OPA). OPA and Rego are industry standards.

Each policy is written as a set of one or more rules, where each rule contains expressions that
are evaluated. All expressions within a rule are logically ANDed together. Separate rules are ORed
together. If a rule evaluates to true, it can set a variable (also called document) to a value it
desires to indicate the evaluation result.

Nextensio has three types of policies:
* a policy for access control per connector (or AppGroup ID) - Access Policy
* a policy for routing to multiple differentiated instances of a service and/or for access control per
service - Route Policy
* a policy for tracing user traffic flows - Trace Policy

Each policy takes a user ID and the attributes for that user ID as input. These attributes are referred
to using the "input." prefix to each user attribute name. For eg., if there is a single-value user
attribute called userRole, it would be referred to as input.userRole within the policy. To refer to a
multi-value user attribute called userTeam, one would use input.userTeam\[\_\] where \[\_\] refers to
any value in userTeam.

Note: a multi-value attribute cannot be referred to using the syntax for a single-value attribute or
vice-versa.

It is also possible to iterate through the values of a multi-value attribute by defining an index and
using the index to refer to the attribute values, eg.,
```
{
    some index
    input.userTeam[index] ...
}
```

Besides user attributes, each type of policy takes more inputs:
* the Access policy takes the target AppGroup ID as a mandatory input and a file containing all AppGroup ID
attribute records as optional reference data
* the Route policy takes the target host ID (aka service) as a mandatory input and a file containing all
host attribute records as optional reference data
* the Trace policy takes a file containing trace request records as optional reference data. Each
trace request record specifies one or more user attribute values to match.

The Appgroup ID fed as input to the Access policy is referred to using input.bid (short for bundle ID).
The host ID fed as input to the Route policy is referred to using input.host.

Attributes in a reference data file are referred to using the "data." prefix. The first thing to
keep in mind is that a reference data file is a collection or array of records.
Refer to each policy sub-section below for how to reference the attributes available via a reference
data file.

There are two strategies for writing any policy:
* write the policy using hardcoded values so as to not use the reference data file
* write the policy in a generalized form to take values to be matched from a reference data file

When the number of hosts (services) is small, the first strategy can be employed for the Route policy.
When the number of AppGroup IDs is small, the first strategy can be adopted for the Access policy.
When the number of trace requests is small, the first strategy can be adopted for the Trace policy.

However, as the number of hosts or AppGroup IDs or trace requests goes up substantially, maintaining
a policy with separate rules having hardcoded values per host or AppGroup ID or trace request can get
cumbersome. It becomes desirable to collapse the policy into a small set of generalized rules that
instead rely on a reference data file. Writing policy rules to use attributes from a reference data
file has to be done with thought and care, as the generalized policy needs to consider the diverse
requirements of different hosts or connectors. Refer to other sections in this chapter for some of
the factors to be considered for each type of policy based on its associated reference data file.

### Summary of policy types

```
Type		Input				  Reference data*
----------------------------------------------------------------------
Access Policy	user attributes + AppGroup ID	  AppGroup ID file
Route Policy	user attributes + host ID  	  Host attributes file
Trace Policy	user attributes	       		  Trace requests file
```
Note: * indicates optional


### Policy changes

A policy can be changed in multiple ways:
* to add a new rule (for a new host or AppGroup ID or trace request)
* to remove a rule (for a host or AppGroup ID or trace request)
* to change the match criteria in any rule
* to enhance the match criteria in any rule
* to prune the match criteria in any rule
* to convert a policy from one strategy to another

All policy changes have to be done with extreme care and the logic validated before deploying the
policy changes.
Policy changes may be driven by attribute changes for users, hosts, or AppGroups. In such a case
the policy changes should always be sequenced to avoid failures due to attribute references that
are not valid (attribute does not exist or is of the wrong type).
* When new attributes are being introduced or attribute types are being changed, the attribute
changes need to be rolled out first before making policy changes
* When attributes need to be deleted, policy changes should be made first to remove references
before removing the attributes.
* Attribute type changes should always be done by introducing attributes with the new type first,
then changing the policy or policies impacted, and finally deleting the attributes with the old
type.

### Access Policy

```
    User attributes                        AppGroup
          +     ----> Access Policy <---- attributes
    AppGroup ID                              file
      (always)                            (optional)
```

An Access policy needs to start with these three statements:
```
package app.access
allow = is_allowed
default is_allowed = false
```

The package statement (first one) is a declaration of this policy to OPA.
The second statement represents the result returned after the policy is evaluated and is initialized
to a default value using a variable used in the policy rules.
The third statement defines and initializes the variable used in the policy rules. Initializing it to
"false" defines it as a boolean variable. As a result, "allow" also becomes a binary/boolean result.

The result returned by the policy is accessed at app.access.allow.

A customer can decide to use a different name for the is_allowed variable, but the result always needs to
be returned at app.access.allow.

As covered earlier in 'Policy overview', the user attributes and AppGroup ID received as input are
referred to using input.\<attribute-name\> and input.bid.
So how does one access the Appgroup attribute values from the reference data file ?
The Appgroup attributes file is an array of records for multiple AppGroup IDs. To refer to the
i'th record which corresponds to some AppGroup ID, one needs to use data.bundles[i].
The attributes for that AppGroup ID would then be referred to via data.bundles[i].<attribute name>.
The AppGroup ID in that i'th record would be data.bundles[i].bid.

So if there is a single-value AppGroup attribute called roleAllowed, it would be referenced via
data.bundles[i].roleAllowed.
If there is a multi-value AppGroup attribute called teamAllowed, it would be referenced via
data.bundles[i].teamAllowed\[\_\] to refer to any of the possible values.

Rules can then be written as follows where \<rule expressions\> match input attributes to constant
values or values for attributes obtained from the reference data file. For eg.,
```
is_allowed {  # Rule1
    <rule expressions>
}
is_allowed {  # Rule2
    <rule expressions>
}
...
```
where a \<rule expression\> can be
```
    some i
    input.userRole == data.bundles[i].RoleAllowed[_]
to match the userRole value to any value in roleAllowed, or
    input.userTeam[_] == data.bundles[i].teamAllowed[_]
to match every value in userTeam to any value in teamAllowed.
```
##### Key things to remember
```
allow : where result of policy should be returned
input.bid : Target Appgroup ID for user
input.<attribute-name> : User attribute
data.bundles[i].bid : AppGroup ID for a record in the reference data
data.bundles[i].<attribute-name> : An attribute in an AppGroup ID record in the reference data
```

### Route Policy

```
    User attributes                          Host
         +    ---->   Route Policy <----  attributes
      host ID                                file
      (always)                            (optional)
```

A Route policy needs to start with these two statements and should not be changed:
```
package user.routing
default route_tag = ""
```

The package statement (first one) is a declaration of this policy to OPA.
The second statement represents the result returned after the policy is evaluated and is initialized
to a default value of "" (null string). As covered earlier, the policy rule(s) can also evaluate to
set route_tag = "deny" in order to block any user's access to a specific host.

The result returned by the policy is accessed at user.routing.route_tag

As covered earlier in 'Policy overview', the user attributes and host ID received as input are
referred to using input.\<attribute-name\> and input.host.
So how does one access the host attribute values from the reference data file ?
The host attributes file is an array of records for multiple host IDs. However, each host ID can have
an array of records for host route attributes. So we basically have an array of records within an array
of records.
To refer to the i'th record which corresponds to some host ID, one needs to use data.hosts[i].
The host ID in that i'th record can then be accessed via data.hosts[i].host.

The route attributes for that host ID are referred to via data.hosts[i].routeattrs[j].<attribute name>
for the j'th record that corresponds to some route tag for that host ID.

So if there is a single-value host route attribute called roleSelect, it would be referenced via
data.hosts[i].routeattrs[j].roleSelect.
If there is a multi-value host route attribute called teamSelect, it would be referenced via
data.hosts[i].routeattrs[j].teamSelect\[\_\] to refer to any of the possible values.

Rules can then be written as follows where \<rule expressions\> match input attributes to constant
values or values for attributes obtained from the reference data file. For eg.,
```
route_tag = tag1 {  # Rule1
    <rule expressions>
    tag1 := <some route tag value>
}
route_tag = tag2 {  # Rule2
    <rule expressions>
    tag2 := <some other route tag value>
}
...
```
route_tag will be set to tag1 or tag2 or ... depending on which rule evaluates to true. The tag value can
also be taken from the matching host route attributes record in the reference data via
data.hosts[i].routeattrs[j].tag.
Multiple rules should not evaluate to true as that leads to a conflict for OPA. A workaround to prevent
that is to use the "else" facility:
```
route_tag = tag1 {  # Rule1
    <rule expressions>
    tag1 := <some route tag value>
} else = tag2 {  # Rule2
    <rule expressions>
    tag2 := <some other route tag value>
} else = ....
```
A \<rule expression\> can be
```
    some i, j
    input.userRole == data.hosts[i].routeattrs[j].RoleSelect[_]
to match the userRole value to any value in roleSelect, or
    input.userTeam[_] == data.hosts[i].routeattrs[j].teamSelect[_]
to match every value in userTeam to any value in teamSelect.
```
##### Key things to remember
```
route_tag : where result of policy should be returned
input.host : Target host ID for user
input.<attribute-name> : User attribute
data.hosts[i].host : Host ID from a record in the reference data
data.hosts[i].routeattrs[j].<attribute-name> : An attribute in a host ID route attributes record in the reference data
```

### Trace Policy

```
    User attributes ----> Trace Policy <---- Trace requests file
       (always)                                  (optional)
```

A Trace policy needs to start with these two statements and should not be changed:
```
package user.tracing
default request = "no"
```

The package statement (first one) is a declaration of this policy to OPA.
The second statement represents the result returned after the policy is evaluated and is initialized
to a default value of "no". To deny a trace request, either "no" or "none" can be returned.
The trace policy is executed for every new user flow.

The result returned by the policy is accessed at user.tracing.request. If tracing is to be enabled
for a flow, it can be any string that is interpreted as an ID for the trace request. This string is
inserted in the trace spans as a "nxt-trace-requestid: \<request\>" tag so that trace spans can
be identified back to a specific trace request.

As covered earlier in 'Policy overview', the user attributes received as input are referred to using
input.\<attribute-name\>.
So how does one access the trace request attribute values from the reference data file ?
The trace request attributes file is an array of records for multiple trace request IDs. To refer to the
i'th record which corresponds to some trace request ID, one needs to use data.tracerequests[i].
The attributes for that trace request ID would then be referred to via data.tracerequests[i].\<attribute name\>.
Use data.tracerequests[i].traceid to access the trace request ID.

If there is a single-value trace request attribute called roleTraced, it would be referenced via
data.tracerequests[i].roleTraced.
If there is a multi-value trace request attribute called teamTraced, it would be referenced via
data.tracerequests[i].teamTraced\[\_\] to refer to any of the possible values.

Rules can then be written as follows where \<rule expressions\> match input attributes to constant
values or values for attributes obtained from the reference data file. For eg.,
```
request = req1 {  # Rule1
    <rule expressions>
    req1 := <some trace request id>
}
request = req2 {  # Rule2
    <rule expressions>
    req2 := <some other trace request id>
}
...
```
request will be set to req1 or req2 or ... depending on which rule evaluates to true. Multiple rules
should not evaluate to true as that leads to a conflict for OPA. A workaround to prevent that is to
use the "else" facility as shown earlier.

A \<rule expression\> can be
```
    some i
    input.userRole == data.tracerequests[i].roleTraced[_]
to match the userRole value to any value in roleTraced, or
    input.userTeam[_] == data.tracerequests[i].teamTraced[_]
to match every value in userTeam to any value in teamTraced.
```
##### Key things to remember
```
request : where result of policy should be returned
input.<attribute-name> : User attribute
data.tracerequests[i].traceid : Trace request ID from a record in the reference data
data.tracerequests[i].<attribute-name> : An attribute in a trace request record in the reference data
```

### Policy Configuration
:-------------------------:
![](/configurations/policies/policy.jpg)

Nextensio policies are written in [Rego Language](https://www.openpolicyagent.org/docs/latest/policy-language/)

For an overview of the Route policy, refer to [routing](/architecture/routing.html).
For an overview of the Access policy, refer to [access control](/architecture/accescontrol.html)

Policy configuration provides a text editor to edit any of the three policies. Care must be taken to
ensure that changes that impact the logic of a policy are validated first (how is TBD).

TODO ASHWIN: Give some more details here as you feel appropriate. Also, now that I think of it, access 
policies for different appgroups might be different right ? Each appgroup is a completely differen beast
of its own with its own attributes and stuff. So we will have one big merged set of attributes for all
app groups, which might be ok, but do we want to have one big humongous policy describing access 
stuff for different app groups ? How is that even done - how do we say "if appgroup == X use this
part of the policy, else use that part of the policy" etc.. Do we see this evolving to have 
AcessPolicy per appGroup and similarly RoutePolicy per host ?


