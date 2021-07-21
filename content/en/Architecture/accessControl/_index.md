---
title: "access control"
linkTitle: "access control"
weight: 10
description: >
---

## Acess Control

Access control is very straight forward. The algorithm for that is as below

1. When a user logs into a nextensio gateway, nextensio gets all the "attributes" of that user
2. When that user tries to access a service appx.awesomecustomer.com, we first do a route lookup
as described [here](/architecture/routing.html) and after route lookup we figure out the "App Group"
to be used to carry this traffic
3. Nextensio lookups the app group's attributes, and with the user attribute and app group
attribute in hand, nextensio will use the routing policy configured in the 
[policy section](/configurations/policies.html) to match these attributes and return a true (allowed)
or false (denied) value

