
---
title: "policies"
linkTitle: "policies"
weight: 20
type: list
menu:
  main:
    weight: 20
---

# Policy

Policy Configuration             
:-------------------------:
![](/configurations/policies/policy.jpg)

Nextensio policies are written in [Rego Language](https://www.openpolicyagent.org/docs/latest/policy-language/)

There are two policies with names "RoutePolicy" and "AcessPolicy". The former controls
[routing](/architecture/routing.html) and the latter controlls [access control](/architecture/accescontrol.html)

TODO ASHWIN: Give some more details here as you feel appropriate. Also, now that I think of it, access 
policies for different appgroups might be different right ? Each appgroup is a completely differen beast
of its own with its own attributes and stuff. So we will have one big merged set of attributes for all
app groups, which might be ok, but do we want to have one big humongous policy describing access 
stuff for different app groups ? How is that even done - how do we say "if appgroup == X use this
part of the policy, else use that part of the policy" etc.. Do we see this evolving to have 
AcessPolicy per appGroup and similarly RoutePolicy per host ?


