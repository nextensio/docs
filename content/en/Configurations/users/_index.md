
---
title: "Users"
linkTitle: "Users"
weight: 20
type: list
menu:
  main:
    weight: 20
---

Aug 5, 2021

## Users

Users are the entities that deploy the nextensio Agent software and connect to the nextensio
gateways to access applications. Users of a company can be full-time employees, part-time employees,
contractors, consultants, vendors, partners, and so on. Users belong to a tenant (company/enterprise)
who would be a customer of nextensio.

Every user is expected to have an email id in the tenant's domain, eg., johndoe@awesomecustomer.com
for the tenant awesomecustomer.com.
As additional information about the user, the user's name also needs to be provided. There may be
additional informational fields provided in future for urgent contact information, etc.

The rest of the configuration per user involves attributes. Attributes represent meta data defined
by the customer for organization specific information to be tagged to each user. The set of attributes
defined will apply uniformly to all users, and every user is expected to have a value for each attribute.
Attributes can be defined for employment type of user or relationship of user to tenant, user role,
user title, user grade level/seniority/privilege level within tenant organization, user
team/department/business-unit (these can be separate), user's default/typical workplace location, and
so on. It is entirely up to the tenant to define and name the attributes and select the type of each
attribute.

Nextensio also provides a fixed set of user attributes per user device. These attributes provide info
about user device name, device o/s type, and device o/s verion. Names of these attributes start with an
underscore ('_') character and can be used in policies. These attributes are:
* _hostname
* _osType
* _osName
* _osMajor
* _osMinor
* _osPatch
* _model

User attributes are mandatory and they form the basic input to any policy, whether for routing,
access control, or tracing. Through this configuration editor, the following operations can be done:
* add a new user
* delete a user
* change any attribute values for any user

All operations above are safe, ie., they cannot cause a conflict with any policy, and hence can be
done at any time. However, the changes to any attribute values or the values entered for a new user
must be carefully validated for correctness. Incorrect values can lead to unexpected/wrong behavior.
Unless a change is very urgent (say the change is required to address a security issue related to
access-control), it is recommended that changes be batched together and done once or twice a day
for better controls and validations. Of course this is just a suggestion and it is up to the tenant
to decide what works best for them.

For addition of new attributes, deleting an attribute, or changing the name or type of an attribute,
refer to the [Attribute Editor section](../configurations/attributeeditor.html) 


Add user             
:-------------------------:
![](/configurations/users/user_add.jpg) 

* User ID: A unique email id of the user

* User Name: A descriptive name

NOTE: In the next update, Nextensio will start supporting federation with
customer's Identity Provider / LDAP and at that point the list of users can be
exported from customer's Identity Provider instead of manually configuring here

For attributes, please refer to the [overview on policy and attributes](/architecture/policyattr.html) 
and [access control](/architecture/accesscontrol.html) details about [configuring attribute editor](../configurations/attributeeditor.html) 