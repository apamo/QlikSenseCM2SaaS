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
SaaS also supports SA for dynamic data reduction. The mechanism works identical i.e. Qlik engine reducing the app's dataset on the fly when user logins, however, there are some differences compared to client-managed when building the security table in the load script. This difference is mainly in the systems fields available and data values to be used.

In SaaS the SA table must contain, as a minimum, two system fields:
- ACCESS
- USERID or __USER.EMAIL__

Other optional system fields are:
- USERID
- NTNAME
- GROUP
- OMIT

Note that SERIAL system field is not available in SaaS, whereas USERID is not mandatory, as opposed to client-managed, when using USER.EMAIL.

Other considerations when building a SA table in SaaS are:

- USERID is always compared to the value in the IdP subject.
    - When using Qlik Account, the IdP subject can be viewed in the Management Consoles under the Users section.
    - When using a custom IdP, the IdP subject can be mapped to match your internal Windows identity e.g. DOMAIN/USERNAME
    - When using JWT authentication, the IdP subject will be set in the `sub` claim of the user payload. Example of jwt payload below uses the **email address** in the `sub` claim but this could also contain your **internal Windows identity**.  

            {  
            "sub": "alvaro.palacios@qlik.com",   
            "subType": "user",    
            "name": "Alvaro Palacios",   
            "email": "alvaro.palacios@qlik.com",    
            "email_verified": true,    
            "groups": ["Presales"]  
            }

- USER.EMAIL contains the user email address which will be obtained from the configured identity provider. If SA table in SaaS needs to apply restrictions at the user level, using USER.EMAIL rather than USERID is a good options since it avoids having to deal with the value set in the `subject claim` which may vary depending on the configure identity provider.

To learn more details about how to work with SA in SaaS please visit our [Online Help](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Scripting/Security/manage-security-with-section-access.htm) 

## Moving Section Access to SaaS

This user guide is targeting Qlik's OEM partners who currently use Qlik Sense Client-managed and are now considering SaaS or have already started moving their deployment to SaaS.

Most likely, there's a SSO integration between your solution and Qlik Sense, and you're leaning on your existing authentication mechanism to avoid replication of users and groups in Qlik Sense. This can be achieved in several ways in client-managed, being Web Ticketing the most commonly used since [it's relatively easy to setup](https://community.qlik.com/t5/Knowledge/Qlik-Sense-for-Windows-All-you-need-to-know-to-start-using/ta-p/1758478):

![Authentication mechanisms in Qlik Sense Client-managed](./images/Authentication%20Methods.png)

The migration journey to SaaS will look different depending on the configured IdP in client-managed and the authentication method to be setup in SaaS:

### 1. Same custom IdP in both client-managed and SaaS 
This is the easiest implementation because the same SA table can be reused and ported directly to SaaS. This SA security table uses USER.ID system field to give access to data. Regardless of the mapping in the IdP i.e. internal Windows identity, email address, or custom value. This is going to be a seamless transition since the same IdP provider will be configured in SaaS. Just remember that your custom IdP must support OIDC protocol to work with SaaS.

### 2. Different custom IdP between client-managed and SaaS
In this case, the SA table will need some adjustments.
- If you want to keep using USER.ID you'll need to replace the data values to match those in the IdP connected to SaaS. Alternatively, the USER.ID can be mapped against the same values from the original SA table.
- If you want to use USER.EMAIL instead of USERID, then add the email address for each user and remove the USER.ID column.

### 3. Custom IdP in client-managed and JWT in SaaS





### Qlik IdP

In this case, the subject claim is an auto-generated x chars string. You can either use USERID or USER.EMAIL together with ACCESS type.

![user in SaaS tenant configured with Qlik IdP](./images/Users%20with%20Qlik%20IdP.png)


### Custom IdP

### JWT authentication

## Tips and Recommendations
