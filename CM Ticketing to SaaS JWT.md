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
## Key Differences

There are plenty of assets available online to understand, learn and implement **Ticketing** and the other authencation mechanisms available in Qlik Sense Client-managed. The following article from Qlik Support's Knowledge Base (KB) is very helpful:
[Qlik Sense for Windows: All you need to know to start using iFrames/Mashups](https://community.qlik.com/t5/Knowledge/Qlik-Sense-for-Windows-All-you-need-to-know-to-start-using/ta-p/1758478)

The OEM EMEA team at Qlik created as well its own Client-managed content for OEM partners. If you'd like to review the Web Ticketing basic concept, then check out the following two videos:
1. [SaaS Integration - Integrated web and security flow](https://www.youtube.com/watch?v=M49nv6on5Eg) (1 min)
2. [Qlik Sense ticketing basic concept](https://www.youtube.com/watch?v=cL17AG2ALr0) (12 min)

&nbsp;

![Qlik Sense Integrated Authentication and Authorization](./images/Qlik%20Sense%20Integrated%20Authentication%20and%20Authorization%20.png)
Author: _Martijn Biesbroek & Raymond Neves_

&nbsp;

If you wish to learn more about JWT, an introduction to JSON Web Tokens can be found in this [link](https://jwt.io/introduction).

The differences between Ticket and JWT authentication are subtle but key to understand before implementing JWT in SaaS:

### __Ticket auth:__
- Slightly easier to implement than JWT.
- Qlik's propietary solution for authenticating users programmatically using a ticket in JSON format.
- The external system (OEM partner's solution) implementing Ticketing doesn't create the ticket itself but sends a request to the Proxy Service running in the Qlik Sense Windows server to obtain it.
- After ticket validation by the Proxy Service, a session cookie is placed in the user's browser which allows access to embedded content from a Qlik Sense application.
- Body in ticket request can include additional user information for authorization i.e. control access on Qlik Sense resources. This information can be leveraged in Security Rules and Section Access as session attributes.
- The "Group" attribute is the only user attribute that can be used in the Section Access security table for data reduction i.e. row-level security.
- Unlike AD or LDAP "Group" attribute, additional attributes in a ticket won't be displayed in the **Users** section of the **Qlik Sense Management Console** since these are session attributes.

Example of body request:
  
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

### __JWT auth:__
- Relatively more difficult to implement than Ticket authentication.
- Open standard in the Internet for securily transmitting information between parties as JSON object.
- The external system (OEM partner's solution) creates a signed token "on-premise" which is sent to SaaS to verify the identity of the user trying to login interactively. 
- After token validation by Qlik Cloud (using the public key), a session cookie is placed in the user's browser which allows access to Qlik Sense SaaS content.
- Similarly, tokens allows to send additional claims in the JWT.
- Today, the only optional user claim (session attribute) that is read is `"groups"`. This claim can be used be used for authorization in SaaS. You can leverage `"groups"` in SaaS for Space access management.

&nbsp;

___
## Things to Consider Before Migrating

Before you read the how-to implement tutorial in the next section, there are few things to consider with SaaS JWT. The main goal is to create a SSO auth integration between your solution and QSE SaaS so that users are automatically logged into QSE SaaS when accessing embedded content in a [Qlik Mashup](). Unlike Qlik Sense client-managed, Ticket authentication isn't an option in SaaS so you need to use JWT.

The JWT has basically three pieces or components: the payload, the signing options, the private key for signing the token. Depending on the creation date of your SaaS tenant, there are two additional properties/attributes for the Qlik JWT payload and the signing options:

### __Qlik Cloud tenants made until June 6, 2022__

- No need to include `notBefore` for the signing options although recommended to mitigate any potential disruption.
- No need to include `jti` (JWT ID) for the Qlik JWT payload although recommended to mitigate any potential disruption.

Source: [What's New in Qlik Cloud](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Introduction/saas-change-log.htm) on May 3rd 2022.

&nbsp;

### __Qlik Cloud tenants made after June 6, 2022__

You'll get an `access denied` from Qlik Cloud without the two additional attributes i.e. `jti` and `notBefore`. Those OEM partners running one single tenant architecture in Qlik Cloud to host all their customers will have the option in the future to switch to a multi-tenant architecture in SaaS where each customer has its own Qlik Cloud tenant. The latter means the creation of new tenants so make sure that you're JWT code is adjusted accordingly to include the two additional attributes related to JWT authorization.

Here is a sample JWT payload including the new mandatory attribute `jti` and optional attribute `groups`:

    {
        "jti": "k5bU_cFI4_-vFfpJ3DjDsIZK-ZhJGRbBfusUWZ0ifBI",
        "sub": "SomeSampleSeedValue", //e.g. QLIK\Alvaro or auth0|a08D000001C6YtJIAV
        "subType": "user",
        "name": "Alvaro Palacios",
        "email": "alvaro.palacios@qlik.com",
        "email_verified": true,
        "groups": ["Developers", "Sales", "Presales"]
    }

Here is a sample of the signing options in JWT including the new mandatory atttribute `notBefore`:
    
    {
        "keyid": "eccf892a-3457-6789-4455-a11g3f869a47",
        "algorithm": "RS256",
        "issuer": "https://oemiberia.eu.qlikcloud.com",
        "expiresIn": "6h",
        "notBefore": "0s",
        "audience": "qlik.api/login/jwt-session"
    }

&nbsp;

## How to Setup JWT in SaaS

In [Qlik.dev](www.qlik.dev) you can learn how to enable JWT in your SaaS tenant, configure a web app to create a JWT token, and authenticate a user into SaaS to view embedded visualizations from a Qlik Sense application. Example code is provided in both Qlik's [GitHub](https://github.com/qlik-oss) and [Glitch](https://glitch.com/@qlik) account that can be used for your own reference and implementation.

__- How-to tutorial:__ [Implement JWT Authorization](https://qlik.dev/tutorials/implement-jwt-authorization)  
__- Example code:__ [Athorize Users with JWTS for Qlik Embedded](https://glitch.com/edit/#!/qlik-cloud-jwt)  
__- Example code 2:__ [How to use JWT authentication in mashups](https://community.qlik.com/t5/Knowledge/Qlik-Sense-SaaS-How-to-use-JWT-authentication-in-mashups/ta-p/1852388)

IMAGE

&nbsp;

Other tutorials for interactive authentication in SaaS:
- [Create Signed Tokens for JWT Authorization](https://qlik.dev/tutorials/create-signed-tokens-for-jwt-authorization)
- [Generate your first API Key](https://qlik.dev/tutorials/generate-your-first-api-key)
  
&nbsp;
___
Written by *Martijn Biesbroek & Alvaro Palacios*  
OEM EMEA Presales Team at Qlik   
[www.qlik.com](https://www.qlik.com)