# Introduction
This user guide describes the outline for allowing OEM partners to successfully migrate to Qlik Cloud a Section Access (SA) table that works in Qlik Sense Enterprise server, i.e. the version of the product you can install on Windows. When "SaaS" or "Qlik Cloud" is mentioned in this document, we are still referring to Qlik Sense but the cloud version hosted in our own cloud infrastructure. When "Client-managed" or "Windows" is mentioned in this document, it means the server version hosted by you.

SA mechanism is used in Qlik Sense (both Windows and SaaS) for dynamic data reduction by the Qlik Engine to provide a different view of the data within the same dashboard layout. In other words, this allows to control the security of an application at the row and column level.

In the OEM space, Section Access can be used for two reasons:

1. To separate/segregate customer data within one monolithic app containing __ALL__ customer data.
2. When having one app per customer, adding row-level security to define who gets to see what.


## Section Access in Client-managed
If your apps in Client-managed use SA today then probably have a security table loaded at the beginning of your load script that looks like this:

![section access table in windows]()


## Section Access in SaaS
SaaS also supports SA in the same form as in Client-managed i.e. a security table in the load script with a number of data restrictions, but it has some particularities though. For example, ...
To learn more details about how to work with SA in SaaS please visit our [Online Help](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Scripting/Security/manage-security-with-section-access.htm) 

## Implementation in SaaS



## Tips and Recommendations
