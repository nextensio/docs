---
title: "Access Control"
linkTitle: "Access Control"
weight: 10
description: >
---

## Access Control

Nextensio provides a flexible and powerful capability for access control. As we've seen
so far, nextensio provides the capability of defining and maintaining attributes per App
and per AppGroup ID. These attributes can be matched with user attributes using policies to
perform access control either at the App level or at the AppGroup ID level.

We have seen in the Routing section that traffic can be routed based on load balancing or
policy based explicit routing. When doing explicit routing via a Routing policy, it should
not happen that we select an instance of the App only to find later that the user is not
permitted to access the App's connector with that instance. Therefore, in situations involving policy
based explicit routing, the Routing policy can also include rules to perform access control.
Nextensio allows the creation of a fake or non-existent instance of the App when a user's access
to an App is to be denied. The tag for this fake instance is "deny". If the Routing policy returns
the tag "deny", instead of prefixing this tag to the App to derive the instance, the user's
traffic stream is dropped/blocked.

In cases where there is no routing, a customer can still chose to use the Routing policy
purely as an access control policy at the App level to match attributes to decide when to
return the "deny" tag (as mentioned above). To permit access to the App, the policy would return
the null or empty string ("") tag.
In this case, the customer can chose to use an Access policy to control access to the AppGroup ID
(or connector). This Access policy matches user attributes with specific values to determine whether
to permit or deny access to the Appgroup ID (and thereby to a data center). In the 'Easy' mode, the
values are specified in the rules. In the 'Expert' mode, the values are set in AppGroup attributes.
Note that this access control applies to a group of Apps represented by the AppGroup ID (by configuring
them as services under the AppGroup ID). An AppGroup ID, in such cases, should therefore comprise of Apps
which have the same access control requirements. When a data center has multiple services with different
access control requirements, they should be bundled together into different AppGroups (and  mapped
to different connectors).

Access control via the Access Policy at the Appgroup ID level is done as given below:

1. When a user logs into a nextensio gateway, nextensio gets all the "attributes" of that user
2. When that user tries to access a service appx.awesomecustomer.com, we first do a route lookup
as described [here](/architecture/routing.html). After route lookup, we figure out the "AppGroup ID"
(connector) to be used to carry this traffic
3. Nextensio then executes the access policy configured in the [policy section](/configurations/policies.html)
to match these user attributes with values specified directly in the policy or by looking up the AppGroup ID's
attributes to return a true (allowed) or false (denied) value.
