# Introduction
This user guide describes the outline for allowing OEM partners to successfully migrate to Qlik Cloud a Section Access (SA) table that works in Qlik Sense Enterprise server, i.e. the version of the product you can install on Windows. When "SaaS" or "Qlik Cloud" is mentioned in this document, we are still referring to Qlik Sense but the cloud version hosted in our own cloud infrastructure. When "Client-managed" or "Windows" is mentioned in this document, it means the server version hosted by you.

SA mechanism is used in Qlik Sense (both Windows and SaaS) for dynamic data reduction by the Qlik Engine to provide a different view of the data within the same dashboard layout. In other words, this allows to control the security of an application at the row and column level.

In the OEM space, Section Access can be used for two reasons:

1. 


## Section Access in Client-managed
If your apps in Client-managed use Section Access for customer data segregation or to control access on data by different groups of users, for data reduction then you have table that looks like this:

![section access table in windows]()


## Section Access in SaaS
SaaS uses 

## Recommendation

##
