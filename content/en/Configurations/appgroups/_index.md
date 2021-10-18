
---
title: "App Groups"
linkTitle: "App Groups"
weight: 20
type: list
menu:
  main:
    weight: 20
---

Aug 5, 2021

## App Groups

A typical large enterprise data center can have hundreds of applications. We are referring
to the applications after discounting the SAAS apps like office365 or salesforce etc.. Even
after discounting all that, a data center can have like five hundred applications. If we consider
a large retailier like a Target or Home Depot, a few example applications can be

* The backend for the point of sales (POS) billing systems in the stores
* Product manuals and training videos
* Live streaming classes / workshops 
* Inventory management accessible to company employees (in some departments)
* Inventory management accessible to the vendors / contractors

These all can be in one cloud or multiple, and even within one cloud in one data center or multiple.
For discussion sake lets assume that these apps are all in the same cloud, same data center. As we
can see above, the apps are of differing nature - they can be categorized in multiple ways. Some apps
can be categorized on "who can access it" (employees v/s vendors, or even employees department wise).
Some apps can be categorized on "what kind of resources do they need" - the point of sales app can
be of a low bandwidth low compute requirement where as the live streaming workshop is a high 
bandwidth high compute requirement.

App Groups (also referred to as "bundle" in some contexts) are Nextensio's way of providing flexibility
to categorize apps in a data center into multiple buckets and have different access control for those
buckets. Can there be just one big app group ? Sure there can be, that's customer's choice as to
whether there needs to be one big app group or a hundred smaller app groups. Since an AppGroup maps
to a connector, besides the access control requirements, the bandwidth requirement of apps through a
connector also needs to be considered. One can of course have multiple replicas for an AppGroup ID to
load balance the traffic through multiple connectors. But one may want to consider separating apps
based on bandwidth and/or latency requirements, besides security requirements.

Let us see what are the configuration parameters of an app group.

### AppGroup information

AppGroups are the connectors that provide access into a data center. They represent a group of
services or applications inside a data center. The grouping of apps for an AppGroup ID is generally
based at least on similar security requirements so that an access control policy can be applied at the
connector level. Further sub-grouping can always be done based on things like bandwidth and/or latency
requirements.

Every AppGroup is expected to have an id in email id format in the tenant's domain, eg., appx@awesomecustomer.com
for the tenant awesomecustomer.com. Note that this id does not have to be a valid email id.
As additional information about the AppGroup, a descriptive name also needs to be provided. There may be
additional informational fields provided in future for urgent contact information of the admin, etc.

The rest of the configuration per AppGroup ID involves attributes. AppGroup ID attributes are used
for access control based on an Access policy by providing the AppGroup attributes file as reference
data for matching with user attributes. This requires writing and maintaining the Access policy directly
and is only possible in the 'Expert' mode. AppGroup attributes are not available in the 'Easy' mode.
Those chosing to use the 'Easy' mode can therefore jump ahead to the 'AppGroup Configuration' section.

Currently, a tenant has to identify the superset of attributes that may be required for all possible
AppGroups requiring access control. However, only a subset of the attributes may be relevant
for any specific AppGroup. Therefore, only a few attribute values may be configured with match values
per AppGroup ID. The Access policy rules must consider this and take into account that some attribute
values may have default values.

When the number of AppGroup IDs is small, the policy rules can be customized per AppGroup ID to match
just the attributes that are relevant for the access control criteria for that Appgroup ID.  This takes
care of dealing with default values for any attributes not selected for matching.
But when the number of AppGroup IDs is large, this can get cumbersome and unmanageable beyond a certain
point. In such cases, one would need to go for a generalized policy using reference data to compact it.

But writing a generalized policy requires matching the same set of attributes for every Appgroup ID.
When there are a large number of AppGroup IDs, it is even more likely that different AppGroup IDs may
require different criteria, ie., different combinations of attribute matching. If we are selective with
the attributes chosen per Appgroup ID, then a generalized policy fails unless default values for unused
attributes are not considered. However, considering default values for unused attributes is non-trivial.
The way around this is to convert unused attributes into "dont-cares" and that can be done by setting
the match criteria for such attributes to have all possible values that a matching user attribute can
have. Let's take an example to illustrate this:
```
  userRole attribute	roleAllowed attribute
  -------------------------------------------
  employee
  consultant
  contractor		["employee","consultant","contractor","vendor","partner"]
  vendor
  partner
```
So whereas a user would have one of the attributes shown in the left column, an AppGroup ID
that does not care about the value of this attribute can set the match criteria to all the values
shown in the right column for its roleAllowed attribute. roleAllowed would be defined as array
type. This will allow matching any user unlike a default value which will not match any user.

Where a user attribute has a small number of possible values, all these possible values should
be defined as the match criteria for the corresponding AppGroup ID attribute that is a "don't-care".
Of course one needs to keep track of the addition of any new value to the user attribute and reflect
that addition in the AppGroup ID attribute value as well.

If a user attribute has a large number of possible values, the above task may get difficult.
In such a case, the customer can consider grouping Appgroup IDs based on attribute groups where the
attribute groups are separated based on attributes with a large set of possible values. The
common elements in each group would be the attributes with a small set of possible values.
The policy rules can then be separated out for each attribute group but still written in a
generalized way to keep the policy compact.

Through this configuration editor, the following operations can be done:
* add a new AppGroup ID and attribute values
* delete an AppGroup ID
* change any attribute values for any AppGroup ID

All the operations above should mostly be safe, ie., they should not cause a conflict with the Access policy.
Conflict here means reference to an attribute that does not exist or whose type is different from
that expected in the policy. Hence these operations can be done at any time.

It goes without saying that any additions or changes must be carefully validated for correctness.
Incorrect values can lead to unexpected/wrong behavior. In addition, even with correct values, the
policy logic must be validated. Additions or changes in values can impact the logic assumed in the
Access policy, based on how the policy rules are written.
Unless a change is very urgent (say the change is required to address a security issue related to
access-control), it is recommended that changes be batched together and done once or twice a day
for better controls and validations. Of course this is just a suggestion and it is up to the tenant
to decide what works best for them.

For addition of new attributes, deleting an attribute, or changing the name or type of an attribute,
refer to the [Attribute Editor section](../configurations/attributeeditor.html)


### AppGroup Configuration

AppGroup Configuration             
:-------------------------:
![](/configurations/appgroups/appgroup_add.jpg)

* AppGroup ID: This is just an email id that will uniquely identify and authenticate an appgroup, 
note that this does NOT have to be a valid email, it just needs to be in an email id format as
of today.

* AppGroup Name: Name is just any descriptive string

* AppGroup ID Compute Pods: Each AppGroup ID as we discussed has different compute requirements
based on how much bandwidth intensive the app is. Based on the application's scale/performance
requirements, we can tune the number of compute units allocated in the Nextensio Gateways
for handling traffic for this AppGroup ID.

* Apps: All modern applications are HTTP based and identified with URLs. For example if 
customer awesomecustomer has an AppGroup which has two applications, the Point of Sales
and usermanuals, identified by pos.awesomecustomer.com and manuals.awesomecustomer.com.
So we will enter pos.awesomecustomer.com,manuals.awesomecustomer.com. Basically, the URLs 
identify the applications in this AppGroup ID. The Apps can be selected through a drop-down list.

For attributes, please refer to the [overview on policy and attributes](/architecture/policyattr.html) 
and [access control](/architecture/accesscontrol.html) details about [configuring attribute editor](../configurations/attributeeditor.html) 