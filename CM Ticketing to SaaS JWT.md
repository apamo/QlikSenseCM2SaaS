# From Client-Managed to SaaS: **Web Ticketing to SaaS JWT**

## Introduction
This user guide is designed to help Qlik's OEM partners with a customer-managed Qlik Sense Enterprise deployment to smoothly move to SaaS.
This document will present the key differences between the most commonly used authentication method for OEM, i.e. Web Ticketing, and the JWT authentication mechanism available in SaaS that would allow any ISV to integrate seamlessly with their solution for user authentication and re-using existing authorization.


## Key differences

There are plenty of online assets that are available to understand, learn and implement **Ticketing** and the other authencation mechanisms available in Qlik Sense Client-managed. The following article from Qlik Support's Knowledge Base (KB) is one of our favorites:
[Qlik Sense for Windows: All you need to know to start using iFrames/Mashups](https://community.qlik.com/t5/Knowledge/Qlik-Sense-for-Windows-All-you-need-to-know-to-start-using/ta-p/1758478)

For the reader of this article who wants to review the Web Ticketing basic concept, the following two videos will be helpful:
1. [SaaS Integration - Integrated web and security flow](https://www.youtube.com/watch?v=M49nv6on5Eg) (1 min)
2. [Qlik Sense ticketing basic concept](https://www.youtube.com/watch?v=cL17AG2ALr0) (12 min)

An introduction to JSON Web Tokens can be found in this [link](https://jwt.io/introduction).

The key differences between Web Ticketing and JWT authentication type are subtle but important to mention:

On one side (Ticketing):
- Ticketing is slightly easier to implement than JWT.
- Ticketing is Qlik's propietary solution for authenticating users programmatically using a one-time ticket in JSON format.
- The external system (OEM partner's solution) implementing Ticketing doesn't create the ticket but sends a request to the Proxy Service running in the Qlik Sense Windows server.
- After ticket validation, a session cookie is placed on the user's browser which allows access to Qlik Sense Client-managed content.

On the other side (JWT):
- JWT is relatively more difficult to implement than Ticketing.
- JWT is the open standard in the Internet for securily transmitting information between parties as JSON object.
- With JWT auth, the external system (OEM partner's solution) creates a signed token "on-prem" that is sent to SaaS to verify the identity of the user trying to login. 
- After token validation (using the public key), a session cookie is placed on the user's browser which allows access to Qlik Sense SaaS content.


## Things to consider before migrating

With Ticketing we can send additional user attributes as JSON object such us groups, roles, etc.

    Example:

    {  
        "UserDirectory": "userDirectory",  
        "UserId": "uniqueUserId",  
        "Attributes": [    
            {      "<Attribute1>": "value1a"    }, 
            {      "<Attribute2>": "value1b"    }  
        ]
    }




