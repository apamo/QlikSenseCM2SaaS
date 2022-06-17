# From Client-Managed to SaaS: **Web Ticketing to SaaS JWT**

## Introduction
This user guide is designed to help OEM partners with a Qlik Sense Client-managed  deployment that are thinking or willing to move to SaaS.

Typically, authentication and authorization is the first integration point with your solution. When embedding analytics with Qlik Sense, OEM partners have a SSO integration in place and are reusing the existing authorization model in their solution. From a security integration standpoint authentication and authorization are associated but cover different aspects:

Authentication – `Who is the user?​`  
Authorization – `What can this user do or see in Qlik Sense?​`

Take as an example a passanger at the airport going through security control and customs desk to confirm his personal identity. Depending on his air ticket this passenger will have the right to get on a plane based on the authorization given by his ticket.

&nbsp;

![Airport Example Authentication and Authorization](https://user-images.githubusercontent.com/10588391/168805817-9303a60e-6dd1-46fe-a972-37c06b7337a1.png)
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

![Qlik Sense Integrated Authentication and Authorization](https://user-images.githubusercontent.com/10588391/168805916-3e394f0e-128f-4b25-bae6-852a1da41630.png)
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

Last but not least, an important remark is that, unlike client-managed, virtual proxies don't exist anymore in SaaS. In SaaS you don't need to [create a JWT virtual](https://help.qlik.com/en-US/sense-admin/May2022/Subsystems/DeployAdministerQSE/Content/Sense_DeployAdminister/QSEoW/Administer_QSEoW/Managing_QSEoW/create-virtual-proxy.htm) proxy to perform an HTTP GET call to authenticate an external user. Instead, in the Management Console in SaaS you create a new Identity Provider and choose JWT from the available options. And, there is just one URL to use: your Qlik SaaS tenant to perform an HTTP POST call that will authenticate an external user. The high-level JWT authentication flow in SaaS is as follows:

![image](https://user-images.githubusercontent.com/12411165/166661007-ad2b1e5e-788b-433c-9280-c96dd0526c93.png)
Author: _Giacomo Brioschi & Martijn Biesbroek_

&nbsp;

___
## Considerations Before Migrating

Before you go over the how-to assets in the next section, there are few considerations with SaaS JWT. As you know, the end goal is to avoid the interactive login when users access embedded content from a Qlik Sense application. This requires an integration between your solution and QSE SaaS so that users are automatically logged into QSE SaaS i.e. SSO authentication. Unlike Qlik Sense client-managed, Ticket authentication isn't an option in SaaS so the only option available is JWT.

The JWT has basically three pieces or components: the payload, the signing options, the private key for signing the token. Qlik made important changes to the JWT authorization capabilities with tenants on June 6, 2022. Basically, there are two additional attributes for Qlik's JWT token:

![image](https://user-images.githubusercontent.com/10588391/174314853-2862f04c-268c-41d3-bcad-e520f10a4718.png)
Source: [What's new in Qlik Cloud](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Introduction/saas-change-log.htm)

&nbsp;

### __For any JWT token created before or after the change__

1. Must include `notBefore` for the signing options to comply with the new changes.
2. Must include `jti` (JWT ID) for the Qlik JWT payload to comply with the new changes.

You'll get an `access denied` from Qlik Cloud if the two additional attributes i.e. `jti` and `notBefore` are not present in the JWT token. If you implemented the JWT code before JUne 6th, 2022 you'll most likely don't have these two attributes. Therefore make sure your JWT code is adjusted accordingly to include them.

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
___
## How to Setup JWT in SaaS

In [Qlik.dev](www.qlik.dev) you can learn how to enable JWT in your SaaS tenant, configure a web app to create a JWT token, and authenticate a user into SaaS to view embedded visualizations from a Qlik Sense application. Example code is provided in both Qlik's [GitHub](https://github.com/qlik-oss/qlik-cloud-jwt) and [Glitch](https://glitch.com/@qlik) account that can be used for your own reference and implementation.

__- Tutorial (How-to):__ [Implement JWT Authorization](https://qlik.dev/tutorials/implement-jwt-authorization)  
__- Example referenced in the tutorial:__ [Authorize Users with JWT for Qlik Embedded](https://glitch.com/edit/#!/qlik-cloud-jwt)  
__- Full JavaScript example:__ [How to use JWT authentication in mashups](https://community.qlik.com/t5/Knowledge/Qlik-Sense-SaaS-How-to-use-JWT-authentication-in-mashups/ta-p/1852388)

Example of JWT creation and signing code:

    const fs = require('fs');
    const uid = require('uid-safe');
    const jwt = require('jsonwebtoken');

    const payload = {
      jti: uid.sync(32), // 32 bytes random string
      sub: '0hEhiPyhMBdtOCv2UZKoLo4G24p-7R6eeGdZUQHF0-c',
      subType: 'user',
      name: 'Alvaro Palacios',
      email: 'alvaro.palacios@qlik.com',
      email_verified: true,
      groups: ['Administrators', 'Presales', 'Qlik'],
    };

    const privateKey = fs.readFileSync('./certs/privatekey.pem');

    // kid and issuer have to match with the IDP config and the
    // audience has to be qlik.api/jwt-login-session
    const signingOptions = {
      keyid: 'my-custom-jwt',
      algorithm: 'RS256',
      issuer: 'https://oemiberia.eu.qlikcloud.com',
      expiresIn: '6h',
      notBefore: '0s',
      audience: 'qlik.api/login/jwt-session',
    };

    const myToken = jwt.sign(payload, privateKey, signingOptions);

    console.log(myToken);

&nbsp;

Other tutorials for JWT authentication in SaaS:
- [Create Signed Tokens for JWT Authorization](https://qlik.dev/tutorials/create-signed-tokens-for-jwt-authorization)
- [Generate your first API Key](https://qlik.dev/tutorials/generate-your-first-api-key)
  
&nbsp;
___
Written by *Martijn Biesbroek & Alvaro Palacios*  
OEM EMEA Presales Team at Qlik   
[www.qlik.com](https://www.qlik.com)
