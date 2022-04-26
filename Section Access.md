# Introduction
This user guide describes the outline for allowing OEM partners to successfully migrate to Qlik Cloud a Section Access (SA) table that works in Qlik Sense Enterprise server, i.e. the version of the product you can install on Windows. When "SaaS" or "Qlik Cloud" is mentioned in this document, we are still referring to Qlik Sense but the cloud version hosted in our own cloud infrastructure. When "Client-managed" or "Windows" is mentioned in this document, it means the server version hosted by you.

SA mechanism is used in Qlik Sense (both Windows and SaaS) for dynamic data reduction by the Qlik Engine to provide a different view of the data within the same dashboard layout. In other words, this allows to control the security of an application at the row and column level.

In the OEM space, Section Access can be used for two reasons:

1. To separate/segregate customer data within one monolithic app containing __ALL__ customer data.
2. When having one app per customer, adding row-level security to define who gets to see what.


## Section Access in Client-managed
If your apps in client-managed use SA today then probably have a security table loaded at the beginning of your app's load script that connects to the data model through one or several reduction fields:

![section access table in load script](https://i.ytimg.com/vi/b-UNTSymHj4/maxresdefault.jpg)

In client-managed the SA table must contain a minimum of two system fields:
- ACCESS
- USERID

Other optional system fields are:
- NTNAME
- GROUP
- SERIAL
- OMIT

A compelte description of these fields and how to use in your SA secutiry table is available in our [Online Help](https://help.qlik.com/en-US/sense/February2022/Subsystems/Hub/Content/Sense_Hub/Scripting/Security/manage-security-with-section-access.htm#anchor-1).

Then you SA table must contain a number of data reduction fields to restrict access to row-level data. Example:

    Section Access;  
    Load * INLINE [  
        ACCESS,  USERID,                 REDUCTION
        USER,    AD_DOMAIN\ADMIN,        *
        USER,    AD_DOMAIN\A,            1
        USER,    AD_DOMAIN\B,            2
        USER,    AD_DOMAIN\C,            3
        ADMIN,   INTERNAL\SA_SCHEDULER,        
    ];  
    Section Application;  
    T1:  
    Load *,  
    NUM AS REDUCTION;  
    LOAD  
    Chr(RecNo()+ord('A')-1) AS ALPHA,   
    RecNo() AS NUM  
    AUTOGENERATE 3;

If you were to import your app in client-managed to SaaS and try to reuse this same SA table, most likely, it won't work well because the USERID field in the SA table is compared to the value of the `subject claim` from Qlik IdP or custom (OIDC compatible) IdP.

Next we're going to see how the SA table works in SaaS, mandatory system fields, etc. and later in this document, how to make the necessary adjustments to reuse SA table from client-managed deployments in SaaS.

## Section Access in SaaS
SaaS also supports SA in the same form as in Client-managed i.e. a security table in the load script with a number of data restrictions, but it has some particularities though. For example, ...
To learn more details about how to work with SA in SaaS please visit our [Online Help](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Scripting/Security/manage-security-with-section-access.htm) 

## Implementation in SaaS

There are three possible ways to authenticate users in SaaS. Your configured method will impact on the migration journey from client-managed to SaaS.

### Qlik IdP

In this case, the subject claim is an auto-generated x chars string. You can either use USERID or USER.EMAIL together with ACCESS type.



### Custom IdP

### JWT authentication

## Tips and Recommendations
