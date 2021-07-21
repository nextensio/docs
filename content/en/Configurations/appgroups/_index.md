
---
title: "app groups"
linkTitle: "app groups"
weight: 20
type: list
menu:
  main:
    weight: 20
---

## App Groups

A typical large enterprise data center can have hundreds of applications, we are referring
to the applications after discounting the SAAS apps like office365 or salesforce etc.. Even
after discounting all that, a data center can have like five hundred applications. If we consider
a large retailier like a target or home depot, a few example applications can be 

* The backend for the point of sales (POS) billing systems in the stores
* Product manuals and training videos
* Live streaming classes / workshops 
* Inventory management accessible to company employees (in some departments)
* Inventory management accessible to the vendors / contractors

These all can be in one cloud or multiple, and even within one cloud in one data center or multiple.
For discussion sake lets assume that these apps are all in the same cloud same data center. As we
can see above, the apps are of differing nature - it can be categorized in multiple ways. Some apps
can be categorized on "who can access it" (employees Vs vendors, or even employees department wise).
Some apps can be categorized on "what kind of resources does it need" - the point of sales app can
be of a low bandwidth low compute requirement where as the live streaming workshop is a high 
bandwidth high compute requirement.

App Groups (also referred to as "bundle" in some contexts) are Nextensio's way of providing flexibility
to categorize apps in a data center into multiple buckets and have different policies for those 
buckets. Can there be just one big app group ? Sure there can be, thats customer's choice as to
whether there needs to be one big app group or a hundred smaller app groups. Let us see what are the
configuration parameters of an app group.

### Config

AppGroup Configuration             
:-------------------------:
![](/configurations/appgroups/appgroup_add.jpg)

* AppGroup ID: This is just an email id that will uniquely identify and authenticate an appgroup

* AppGroup Name: Name is just any descriptive string

* AppGroup Compute Pods: Each appgroup as we discussed has different compute requirements
based on how much bandwidth intensive the app is. Based on the applications scale/performance
requirements, we can tune the number of compute units allocated in the Nextensio Gateways
for handling traffic for this appgroup

* Services: All modern applications are HTTP based and identified with URLs. For example if 
customer awesomecustomer has an appgroup which has two applications, the Point of Sales 
and usermanuals, identified by pos.awesomecustomer.com and manuals.awesomecustomer.com.
So we will enter pos.awesomecustomer.com,manuals.awesomecustomer.com. Basically, the URLs 
identify the applications in this appgroup

For attributes, please refer to the [overview on policy and attributes](/architecture/policyattr.html) 
and [access control](/architecture/accesscontrol.html) details about [configuring attribute editor](../attributeeditor.html) 