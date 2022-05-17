# From Client-Managed to SaaS: **Section Access**

## Introduction
This user guide describes the outline for allowing OEM partners to successfully migrate to Qlik Sense SaaS (Qlik Cloud) a Section Access (SA) table that works in Qlik Sense Enterprise server (the version of the product you can install on Windows). When "SaaS" or "Qlik Cloud" is mentioned in this document, we are still referring to the cloud version of Qlik Sense hsoted in our own cloud infrastructure. When "Client-managed" or "Windows" is mentioned in this document, it means the server version of the Qlik Sense hosted by you in your own servers in either a private cloud, public cloud or on-premise.

SA mechanism is used in Qlik Sense (both Windows and SaaS) for dynamic data reduction by the Qlik Engine to provide a different view of the data within the same dashboard layout. In other words, this allows to control the security of an application at the row and column level. This mechanism is part of our **security model** in SaaS as depicted below:

![Security Model in Qlik Sense SaaS](https://user-images.githubusercontent.com/10588391/168802513-81c52f73-a380-4bb0-9b7f-0ed2d0cc1917.png)

&nbsp;

In the OEM space, Section Access can be leveraged for the following reasons:

1. __Segregate customers' data__ within one monolithic app that contains __all__ customer data.
2. Add row-level security to define __who (users) gets to see what (content)__ when provisioning one app per customer.

Next two sections in this document are meant to review how the SA mechanism works in both client-managed and SaaS before we dig deeper into the key differences and how to implement/migrate in SaaS when coming from a client-managed deployment.

&nbsp;

***

## Section Access in Client-managed

If your apps in client-managed use SA today then probably have a security table loaded at the beginning of your app's load script that connects to the data model through one or several reduction fields:

![Section access security table in load script](https://user-images.githubusercontent.com/10588391/168802565-c670907c-9e1c-4a86-9003-1192ed644083.png)

&nbsp;

In client-managed the SA table must contain a minimum of two system fields:
- ACCESS
- USERID

Other optional system fields are:
- NTNAME
- GROUP
- SERIAL
- OMIT

A complete description of these fields and how to use in your SA secutiry table is available in our [Online Help](https://help.qlik.com/en-US/sense/February2022/Subsystems/Hub/Content/Sense_Hub/Scripting/Security/manage-security-with-section-access.htm#anchor-1).

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

&nbsp;

If you were to import your app in client-managed to SaaS and try to reuse this same SA table, most likely, it won't work well because the USERID field in the SA table is compared to the value of the `subject claim` from Qlik IdP or custom IdP that supports OIDC as shown below:

![Example of IdP in a SaaS deployment](https://user-images.githubusercontent.com/10588391/168802744-11655280-1f7e-4a4f-8a66-b3870fe79984.png)

&nbsp;

Next, we're going to review how Section Access looks like in SaaS, what the mandatory system fields are, etc. and later in this document, how to make the necessary adjustments to port or migrate to SaaS the SA security table you currently use in your client-managed deployment.

&nbsp;
***

## Section Access in SaaS
SaaS also supports SA for dynamic data reduction. The mechanism works identical i.e. Qlik engine reducing the app's dataset on the fly when user logins, however, there are some differences compared to client-managed when building the security table in the load script. This difference is mainly in the systems fields available and data values to be used.

In SaaS the SA security table must contain, as a minimum, two system fields:
- ACCESS
- USERID or __USER.EMAIL__

The other system fields are optional:
- USERID
- NTNAME
- GROUP
- OMIT  

### **Important Considerations:**

- SERIAL system field is not available in SaaS
- USERID is not a mandatory system field 
- Use either USERID or USER.EMAIL (not both) for user-level security​
- Use GROUP for group-level security when using a custom IdP or JWT
- USERID is always compared to the value in the **IdP subject**. The **IdP subject** field can be used for distinguishing one user from another if the names are identical and the email field is not visible.
    - When using Qlik Account, the **IdP subject** can be viewed in the **Management Console** under the **Users** section.
    - When using a custom IdP, the **IdP subject** can be mapped to match your internal Windows identity e.g. DOMAIN/USERNAME
    - When using JWT authentication, the IdP subject will be set in the `sub` claim of the user payload. Example of JWT payload is shown below. Note that it uses the **email address** in both the `sub` and `email` claim, however, the subject could be mapped to the user's **internal Windows identity**.  

            {  
            "jti": "k5bU_cFI4_-vFfpJ3DjDsIZK-ZhJGRbBfusUWZ0ifBI"
            "sub": "alvaro.palacios@qlik.com",   
            "subType": "user",    
            "name": "Alvaro Palacios",   
            "email": "alvaro.palacios@qlik.com",    
            "email_verified": true,    
            "groups": ["Presales"]  
            }

- USER.EMAIL contains the user email address which will be obtained from the configured identity provider. If SA table in SaaS needs to apply restrictions at the user level, using USER.EMAIL rather than USERID is a good options since it avoids having to deal with the value set in the `subject claim` which may vary depending on the configure identity provider.

To learn more details about how to work with SA in SaaS please visit our [Online Help](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Scripting/Security/manage-security-with-section-access.htm)

&nbsp;
***

## Moving Section Access to SaaS

This user guide is targeting Qlik's OEM partners who currently use Qlik Sense Client-managed and are now considering SaaS or have already started moving their Qlik Sense deployment to SaaS.

Most likely, there's a SSO integration between your solution and Qlik Sense, so you're leaning on your existing authentication mechanism to avoid replication of users and groups in Qlik Sense. This can be achieved using several methods (see image below) in client-managed, being Web Ticketing the most commonly used since [it's relatively easy to setup](https://community.qlik.com/t5/Knowledge/Qlik-Sense-for-Windows-All-you-need-to-know-to-start-using/ta-p/1758478):

![Authentication Methods in Qlik Sense Client-managed](https://user-images.githubusercontent.com/10588391/168802912-b8fabc82-0901-4350-860b-a35ff75fbef9.png)

&nbsp;

The migration journey from client-managed to SaaS looks different depending on the use case. For example, if the configured IdP in client-managed is the same in SaaS, then the SA table can be exactly the same. However, if we have different identity provides on both ends, then the SA table will require some adjustaments to work in SaaS. Next section provides more details for each use case:

1. Same custom IdP in both client-managed and SaaS
2. Different custom IdP between client-managed and SaaS
3. Custom IdP in client-managed and JWT authentication in SaaS
4. Custom IdP in client-managed and Qlik IdP in SaaS

&nbsp;

### **1. Same custom IdP in both client-managed and SaaS**

This is the easiest implementation because the same SA table can be reused as is and ported directly to SaaS. This SA security table uses USER.ID system field to give access to data. Regardless of the mappings donde in your IdP i.e. internal Windows identity, email address, or any other custom values, this will seamlessly work in SaaS since the same IdP configured in client-managed will be used in SaaS. Just remember that your configured IdP in client-managed must be compatible with the OIDC protocol to work with SaaS.

&nbsp;

![Example of same IdP in client-managed and SaaS deployment](https://user-images.githubusercontent.com/10588391/168803293-4b355da8-72bf-4041-8287-ed926d04b038.png)

&nbsp;

Authorization script:

    Section Access;
    LOAD * INLINE [
        ACCESS,  USERID,      COUNTRY 
        ADMIN,   ABD\Admin   
        USER,    ABC\Joe,     United States          
        USER,    ABC\Ursula,  Germany
        USER,    ABC\Stefan,  Sweden
    ];	

&nbsp;

### **2. Different custom IdP between client-managed and SaaS**

In this case, the original SA security table used in client-managed isn't 100% compatible with SaaS so it needs to be adjusted.

![Example of different IdP in client-managed and SaaS deployment](https://user-images.githubusercontent.com/10588391/168803378-6c2ecdec-8dc4-4ad8-acbe-d98b9aedcaed.png)

&nbsp;

- Most likely we'll have a conflict in the field USER.ID. The original values won't match the values stored in your IdP connected to SaaS, and so this column needs to be mapped against the new values in your IdP. Alternatively, the original values could be mapped in the IdP in SaaS but this requires working with the IdP configuration e.g. Okta, Auth0, ADFS, etc. rather than making changes in the authorization script.
- For an alternate way to verify user identity using email address, see the field USER.EMAIL. Using USER.EMAIL instead of USERID is more practical and recommended in SaaS.
- Generally speaking, use either USERID or USER.EMAIL (not both) for user-level security
- However, if you need to manage user access in a multi-cloud environment i.e. use client-managed and SaaS simultaneously for a while, you can create a SA table that works on both environments. See example below.

&nbsp;

Authorization script:

    Section Access;
    LOAD * INLINE [
        ACCESS,  USERID,      USER.EMAIL,                   COUNTRY 
        ADMIN,   ABD\Admin    *,                            *
        ADMIN,   *,           admin.sense@example.com       *
        USER,    ABC\Joe,     *,                            United States          
        USER,    *,           joe.smith@example.com,        United States
        USER,    ABC\Ursula,  *,                            Germany
        USER,    *,           ursula.schultz@example.com,   Germany   
        USER,    ABC\Stefan,  *,                            Sweden
        USER,    *,           stefan.svensson@example.com,  Sweden
    ];	

### **3. Custom IdP in client-managed and JWT authentication in SaaS**

&nbsp;

When using JWT auth in SaaS, the subject claim `sub` and the email address claim `email` are custom values defined in the JWT payload. Therefore, you can use either the value in `sub` when using USERID or `email` when using USER.EMAIL inside your SA table. You cannot use both at the same time.

&nbsp;

![Accessing Qlik Sense SaaS with a signed JWT](https://help.qlik.com/en-US/sense-admin/February2022/Subsystems/DeployAdministerQSE/Content/Resources/Images/dr_QlikSenseAccessJWT.png)

&nbsp;

Authorization script:

    Section Access;
    LOAD * INLINE [
        ACCESS,  USER.EMAIL,                   GROUP,       COMPANY 
        ADMIN,   admin.sense@qlik.com,         QLIK,        *       
        USER,    joe.smith@abc.com,            ABC,         ABC
        USER,    ursula.schultz@acme.com,      ACME,        ACME   
        USER,    stefan.svensson@abc.com,      ABC,         ABC
    ];

Note that Qlik Sense SaaS will get the information from the `groups` claim of the configured identity provider (JWT) and compare it to the value in this field. For group-level security, you can link data restrictions to values in the field GROUP, then set USER.EMAIL to asterisk `"*"`. This simplifies and reduces the amount of rows to be scanned in the SA table every time the app is opened by a user.

&nbsp;

### **4. Custom IdP in client-managed and Qlik IdP in SaaS**

In this case, the subject claim cannot be mapped to a custom value and it's an auto-generated string e.g. `auth0|a08D000001C6YtJIAV` that can be viewed in **Users** section of the **Management Console**.

&nbsp;

![Users in SaaS Management Console configured with Qlik IdP](https://user-images.githubusercontent.com/10588391/168804006-831e5ae4-3efc-4f1d-b586-ca86f17f7ab2.png)

&nbsp;

You can use either USERID or USER.EMAIL (not both) in the SA table for row-level security at the user level. Make sure the values in USERID column match the same values found in **IdP subject** in the **Management Console**.  
_User groups are not supported when using Qlik Identity Provider (IdP)_

    Section Access;
    LOAD * inline [
        ACCESS, USERID, COUNTRY, OMIT
        ADMIN, auth0|a08D000001C6YtJIAV, *
        USER, auth0|a08D00000190d9UIAQ, SPAIN, MARGIN
        ADMIN, INTERNAL\SA_SCHEDULER, *
    ];

&nbsp;




## Guidelines and Tips for using Section Access

Visit our [Online Help](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Scripting/Security/manage-security-with-section-access.htm#anchor-9) to review some important facts and helpful hints to know about Section Access regarless of your deployment type, i.e. this applies to both client-managed and SaaS.

&nbsp;
___
Written by Martijn Biesbroek & Alvaro Palacios   
OEM EMEA Presales Team at Qlik   
[www.qlik.com](https://www.qlik.com)
