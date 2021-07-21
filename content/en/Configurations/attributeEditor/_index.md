
---
title: "attribute editor"
linkTitle: "attribute editor"
weight: 20
type: list
menu:
  main:
    weight: 20
---

## Attributes

Attribute Editor
:-------------------------:
![](/configurations/attributeEditor/attredit.jpg)

It will be useful to have an understanding of what attributes are and where its used
etc.. as documented [here](/architecture/policyattr.html)

The attribute editor is purely defining / setting the ground rules for how many attributes
are there, what are their names, who it applies to (User, AppGroup or Host) and what kind 
of values it can contain. Note that the values itself are different for each host or each
user or each appgroup, and the values are set in those individual configuration pages.

The moment an attribute is added, that new attribute is applied to all users / appgroups / hosts
(depending on who that attribute applies to) with a default value for that type. The default
values are as below

* integer - 0
* array of integers - [ 0 ]
* string - ""
* array of strings - [ "" ]
* boolean - false
* array of booleans - [ false ]
* date - 0
* array of dates [ 0 ]


