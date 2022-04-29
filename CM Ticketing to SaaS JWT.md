# From Client-Managed to SaaS: **Web Ticketing to SaaS JWT**

## Introduction
This user guide is designed to help OEM partners with a Qlik Sense Client-managed  deployment that are thinking or willing to move to SaaS.

Typically, authentication and authorization is the first integration point with your solution. When embedding analytics with Qlik Sense, OEM partners have a SSO integration in place and are reusing the existing authorization model in their solution. From a security integration standpoint authentication and authorization are associated but cover different aspects:

Authentication – `Who is the user?​`  
Authorization – `What can this user do or see in Qlik Sense?​`

Take as an example a passanger at the airport going through security control and customs desk to confirm his personal identity. Depending on his air ticket this passenger will have the right to get on a plane based on the authorization given by his ticket.

&nbsp;

![Airport Example Authentication and Authorization​](./images/Airport%20Example%20Authentication%20and%20Authorization%E2%80%8B.png)
Author: _Martijn Biesbroek & Raymond Neves_

&nbsp;

Next, this document will present first the key differences between the most commonly used authentication method for OEM i.e. Web Ticketing. Secondly, a brief description of JWT as idenity provider for authorizing users in your SaaS tenant. Note that JWT mechanism is enabled by default in your SaaS tenant, so it will allow any ISV to integrate seamlessly with their own solution for user authentication without prompting for user credentials (SSO) and re-using existing user authorization.

&nbsp;
___
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

&nbsp;

___
## Things to consider before migrating

Ticketing auth mechanism in client-managed allows to pass on user information as JSON object. Multiple optional claims (user attributes in example below) are supported in Qlik Sense despite the fact that only **Group** will be displayed in the **Users** section of the **Qlik Sense Management Console** (to confirm with Martijn).

    Example:
  
    {
        "UserDirectory": "userDirectory",
        "UserId": "uniqueUserId", [unique per user directory]
        "Attributes":
            [ { "<Attribute1>": "value1a" },
                { "<Attribute1>": "value1b" }, [attributes are not unique]
                { "<Attribute2>": "" }, [value can be empty]
                { "<Attribute3>": "value3" },
                ...
            ], [optional]
        "TargetId": "targetId" [from query string, optional]
    }

   
Source: [Ticket Authentication API ](https://help.qlik.com/en-US/sense-developer/February2022/Subsystems/ProxyServiceAPI/Content/Sense_ProxyServiceAPI/ProxyServiceAPI-ProxyServiceAPI-Authentication-Ticket-Add.htm)

&nbsp;

![Qlik Sense Integrated Authentication and Authorization](./images/Qlik%20Sense%20Integrated%20Authentication%20and%20Authorization%20.png)
Author: _Martijn Biesbroek & Raymond Neves_

&nbsp;

JWT auth is the available mechanism in SaaS that allows to pass user information into Qlik Sense. Ticketing doesn't exist in SaaS as such. With JWT it's possible to send additional claims in the JWT payload. **Today, unlike client-managed, only the optional claim `groups` is read.** This is something important to consider when moving from client-managed to SaaS. Not every optional claim used in client-managed can be moved to SaaS as of today.

Here is a sample JWT payload including the `groups` attribute:

    {
        "jti": "k5bU_cFI4_-vFfpJ3DjDsIZK-ZhJGRbBfusUWZ0ifBI",
        "sub": "SomeSampleSeedValue", //e.g. QLIK\Alvaro or auth0|a08D000001C6YtJIAV
        "subType": "user",
        "name": "Alvaro Palacios",
        "email": "alvaro.palacios@qlik.com",
        "email_verified": true,
        "groups": ["Developers", "Sales", "Presales"]
    }

&nbsp;
___
Written by *Martijn Biesbroek & Alvaro Palacios*  
OEM EMEA Presales Team at Qlik   
[www.qlik.com](https://www.qlik.com)