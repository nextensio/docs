---
title: "Overview"
linkTitle: "Overview"
weight: 1
description: >
  
---

Getting applications to talk to their backends or the backends talk to each other, is a complex 
task involving deep traditional networking skills, layer3 routing, BGP / IPSec etc..

* Its hard to get it right
* Its inflexible and slows down the pace at which changes can be done
* Its difficult to diagnose if something goes wrong

A class of solutions available and upcoming that tries to solve the problem by hiding the 
complexity behind GUIs/APIs etc.. appears simpler but uses the same underlying old 
technologies and just delays the time before the same old problems are exposed. 

Nextensio solution has an architecture based on modern web/cloud principles and moves away
from traditional routing/bgp/ipsec.

## Existing architecture

Say an enterprise has an App X runing on their users' devices, and has backends in both
Amazon AWS and Google GCP. In the picture here, the "Cloud Gateway" is either owned by the
enterprise or is a 'service' like SD-WAN / SASE offered by one of the vendors. The
typical options of connecting to AWS / GCP from the cloud gateway are either layer3 routing
with BGP configurations over some dedicated circuits OR using ipsec tunnels into a tunnel 
head end in AWS and GCP and then running BGP over those tunnels. And of course the user is 
provided the IP address of the backend in either AWS or GCP via DNS.

![](/overview/traditional.jpg)

IT admins who maintain such topologies are well aware of how non trivial it is to set up
and maintain/troubleshoot these classic networing technologies. They often get into 
situations where additional capacity needs to be added, more tunnels need to be provisioned,
and now suddenly the question comes up as to whether tunnels can load balance or they can
only be active/standby at a tunnel layer, and if tunnel layer can load balance, how to tweak
BGP to pick all the underlying tunnels etc.. etc.. The SD-WAN / SASE vendors do provide 
GUIs to configure BGP and tunnels etc.., but under the covers its just configuring a BGP
routing engine and often when things go wrong, customers will witness back-to-square-one 
troubleshooting with BGP CLIs

The above is purely talking about getting from the user to the backend in the clouds. If the
backends themselves want to talk to each other, then the network architect has to figure out
what IP addressing scheme to use, how to advertise routes from one DC to the other DC, etc..
This is actually a problem that has disappointing solutions - the solutions proposed by
many vendors are to "virtualize" the network and make both DCs appear to be part of the same
"virtual network" - that is disappointing at so many levels, because forcing completely 
different cloud vendors with completely different networking models to look like "one network"
is just going backwards 

## Nextensio Architecture

Nextensio operates on the basic principle that what the user application is trying to access
is a "service" - a REST API call to http://foobar.com/api/v1/listusers is an example. All
modern applications are accesses to some http service or the other. In the current/traditional 
models, the service is converted to an IP address, then the IP address is routed, policies 
are applied based on the IP etc., and when the IP hits the data center, its converted back 
into a service by web/http proxies like nginx or kubernetes ingress etc.. 

The question nextensio asks is "do we need to do the translations" ? The answer is we do not
need to do the translations - the traditional/current models do it because there is nothing 
better out there! 

![](/overview/nextensio.jpg)

Now suddenly the cloud gateway is not dealing with routing IP addresses any more, its dealing
with connecting services. So we have a database of services and users trying to reach backends
offering those services, and backends trying to reach other backends offering some services.
And that opens up the possibility for doing a very rich set of routing and policies, and 
very rich telemetry and AI-Ops based on more abstract, more meaningful data than just ip addresses.


## Next 

It is easy to come away from this overview with an idea that maybe the cloud gateway doing
the service based actions is nothing but a regular kubernetes service mesh - thats what kubernetes
does anyways, service routing! While that is a reasonable first impression, that is far from
an accurate picture. What we do is far more than kubernetes or any http mesh technology out
there can do. For example, we can have programmable/dynamic policies using Rego language. 
To find out more, read our [Architecture](/architecture/onboarding.html) section


