
---
title: "Attribute editor"
linkTitle: "Attribute editor"
weight: 20
type: list
menu:
  main:
    weight: 20
---

## Attributes

Note: "AppGroup ID" and connector will be used interchangeably as they represent the same entity.

Attributes are a way to associate meta data with some entity. In nextensio, those entities are users,
hosts, and connectors. These attributes are defined, maintained and used by the customer and do not
impact nextensio in any way. For eg., nextensio does not need to change any software when a new attribute
is defined or used. The definition and use of attributes is totally within the customer's domain. Of
course attributes need to be defined with thought and care, esp when it comes to their type and purpose.
The cost of changing or deleting attributes can become cumbersome depending on the scale of records using
those attributes or how they are used in the policies.

The attribute editor is an innovative component of the nextensio controller UX that lets a customer
define the attributes required for users, hosts, and connectors. Through the editor, the customer defines
* name of the attribute
* type of the attribute
* where it is applicable

Note that the values are NOT defined here.

As soon as the attribute is defined (added), it is also added to all the existing records for the attributes
file where it is applicable (user, host or AppGroup) with default values. 

User attributes are mandatory as they are the basic mandatory input to any policy. Most user attributes
will have a single value, ie., they will not be array types. However, there can be some attributes that
can have multiple values. For eg., whereas an employee who is an individual contributor will normally be
part of a single team or department, an employee at the management level will likely be part of multiple
teams or departments based on their span of control. This is important, because the type chosen should
be the least-common-denominator. So if an attribute can be an array type for even one user, it needs to
be an array type for all users. This is true even for host or AppGroup attributes. Defining an attribute
type as an array does not mean it must have multiple values - the attribute can still have a single or even
a null value (for eg., "" is the null value for a string). 

As covered in earlier sections under Architecture, host and connector attributes are optional, depending
on how the policies are written, ie., whether they need reference data. Host and connector attributes
can also be created and maintained even if the policies do not use them (in case a customer wants to be
prepared for changing their policy strategy later or simply as a record for reference).


Attribute Editor
:-------------------------:
![](/configurations/attributeEditor/attredit.jpg)

The purpose of attributes and where they are used etc. is also documented [here](/architecture/policyattr.html)

### Handling attribute changes

Once operational, a customer may have a need to
* add a new attribute
* change the type of an attribute
* change the name of an attribute
* delete an attribute no longer needed

All such changes should be done with care and following a process sequence to avoid disruptions.
For eg., an attribute being used in a policy cannot be changed or deleted, because that will
cause the policy to fail. Failure of a policy can result in wrong behavior when it comes to
routing or access control. We will go over the general policy for handling the above types of
attribute changes.

Note that attribute values are not changed via the editor, nor is changing attribute values disruptive
unlike changing attribute names or types.

Many or most attribute changes will originate at the user level and require corresponding
attribute changes in the host attributes and/or AppGroup attributes. In this case, the corresponding
attribute defined for the host and/or AppGroup may not be the same type. In many cases, whereas
a user attribute may be a single value attribute, it may be desirable to define an array type
attribute for the host or AppGroup attributes to allow matching against multiple values. This can be
leveraged in the policies if using the attributes file as reference data because Rego provides an easy
way to match any value in an array of values. If the policy were to use the other strategy of using
hard coded values for matching, checking for a match with multiple values makes the policy more complex
as rules have to be replicated.

There may of course be situations where an attribute is added to just the host and/or AppGroup
attributes.

A general rule of thumb when using policies is that
* for adding an attribute, get the data changes done and deployed first before changing the policy
* for deleting an attribute, change and deploy the policy first before changing the attribute files.
* for changing an attribute, introduce new attributes, change policies to use the new attributes, then
remove the old attributes

#### Adding a new attribute

This is a safe operation as adding a new attribute should not cause a policy to fail, since no
policy should be referring to an attribute before it is defined. Changing policies to use a new
attribute should be the absolute last task in this process.

The first task, of course, is to define the new attribute via the editor. As soon as the new attribute
is added, it will be propagated to all records in the attribute files where it is applicable (user,
host, AppGroup). The new attribute will have a default value assigned, based on the type, as shown
below:

* integer - 0
* array of integers - [ 0 ]
* string - ""
* array of strings - [ "" ]
* boolean - false
* array of booleans - [ false ]
* date - 0
* array of dates [ 0 ]

[ ] is the nomenclature to refer to an array. It dictates how data is written to the database and
read from it so that the array type is maintained even when there is a single value. This is a must
for the policy processing engine which expects an attribute value to be of the same type for all
records in any reference data file associated with a policy.

The second task is to update the attribute values in all records of the respective DB and ensure
the values are correct for every single record. Verifying that attribute values are correct is
a manual job and each customer decides the best way to accomplish it. If more than one attribute
is to be added, it is advisable to club those attribute additions so that this process can be
optimized.

Once the new attribute(s) is/are rolled out, ie., all DB records have been updated and verified
for correctness, the third task of using the attribute in any policy can be undertaken. The policies
that need to use the new attribute(s) would be modified now.

The second and third tasks are done outside this editor but covered here for completeness.

#### Changing attribute type

This should be accomplished as follows :
* define and add a new attribute with the required type as explained above
* modify any policies involved to use the new attribute(s) and deploy them after testing,
* remove the attribute(s) with the deprecated type.

#### Changing attribute name

This should be handled the same way as changing an attribute type.

#### Deleting a deprecated attribute

It is desirable to clean up attributes no longer required, say because they were replaced by
new attributes. This task is, of course, not necessarily urgent. One must ensure that any
attribute being deleted is no longer being referenced in any policy in use.
So the first task is to ensure/confirm that all references to the attribute have been removed
from all policies.
Then delete the attribute definition via this editor. That delete will remove the attribute from
all attribute files where the attribute is present.



