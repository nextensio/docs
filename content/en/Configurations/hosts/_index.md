
---
title: "hosts"
linkTitle: "hosts"
weight: 20
type: list
menu:
  main:
    weight: 20
---

## Hosts

Host definition             
:-------------------------:
![](/configurations/hosts/host_add.jpg)

* Host: is a valid URL for the host

* Name: A descriptive name


Host routng config             
:-------------------------:
![](/configurations/hosts/host_routes.jpg)

The example above shows a host defined with two "tags" - "aws" and "do", and each tag has
a seperate set of attributes. A tag is just any string, for details on how its used in
routing, refer [routing](/architecture/routing)

For attributes, please refer to the [overview on policy and attributes](/architecture/policyattr) 
and details about [configuring attribute editor](../attributeeditor/) 