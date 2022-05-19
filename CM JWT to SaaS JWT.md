# From Client-Managed to SaaS: **QSEoW JWT to SaaS JWT**

## Introduction

An introduction to JSON Web Tokens can be found in this [link](https://jwt.io/introduction).

Basically, JSON Web Token (JWT) is an open standard for secure transmission of information between two parties as a JavaScript Object Notation (JSON) object. JWT is used for authentication and authorization. Because JWT enables single sign-on (SSO), it minimizes the number of times a user has to log on to cloud applications and websites. 

The following example shows the steps involved when gaining access to Qlik Sense by using a signed JWT.

![Accessing Qlik Sense SaaS with a signed JWT](https://help.qlik.com/en-US/sense-admin/February2022/Subsystems/DeployAdministerQSE/Content/Resources/Images/dr_QlikSenseAccessJWT.png)

This [page](https://help.qlik.com/en-US/sense-admin/May2022/Subsystems/DeployAdministerQSE/Content/Sense_DeployAdminister/QSEoW/Administer_QSEoW/Managing_QSEoW/JWT-authentication.htm) from Qlik's Online Help shows the basic concept explained for using JWT in Qlik Sense Client-managed:
- JWT authentication
- JWT structure 
- Standard fields in a JWT claim
- Limitations
- [Example code](https://github.com/qlik-oss/enigma.js/tree/master/examples/authentication/sense-using-jwt)

This [article](https://community.qlik.com/t5/Knowledge/Qlik-Sense-How-to-set-up-JWT-authentication/ta-p/1716226) from Qlik Support's Knowledge Base explains how to simply set up JWT authentication using Qlik Sense Client-managed default certificates and test it.

Therefore, planning this migration should be relatively easy for OEM partners, or at least, easier than if you're coming from Sense [Ticket authentication](https://github.com/qlik-oss/enigma.js/tree/master/examples/authentication/sense-using-ticket) for client-managed version. Here, you're already familiar with JWT auth method as it's the current method in place, the token creation process, signing code, etc. Yet, there are some nuances between JWT in SaaS vs client-managed you need to learn about.

&nbsp;
___
## Key Differences
JWT authentication in SaaS does have some slight differences compared to the client-managed version using virtual proxies. The idea and concept is the identical but the endpoints and the contents of the JWT have changed in SaaS. The basic idea is still that you:

1. Create a JWT token on the server side of your host application (your web server)
2. When you access Qlik Sense embedded content you include the JWT in the header of the request
3. In return, Qlik Sense will now supply a session cookie to be used in further requests to open Qlik Sense content

In client-managed the JWT authentication flow looks like this:

![image](https://user-images.githubusercontent.com/12411165/166260604-c7b1c90d-c8d1-40c9-92c3-1ca2fa04056f.png)  
Author: _Martijn Biesbroek & Raymond Neves_

&nbsp;

In SaaS we don't have the concept of [Virtual Proxy](https://help.qlik.com/en-US/sense-admin/May2022/Subsystems/DeployAdministerQSE/Content/Sense_DeployAdminister/QSEoW/Administer_QSEoW/Managing_QSEoW/create-virtual-proxy.htm) anymore. There is just one URL to use: your Qlik SaaS tenant. The JWT authentication flow in SaaS looks like this:

![image](https://user-images.githubusercontent.com/12411165/166661007-ad2b1e5e-788b-433c-9280-c96dd0526c93.png)
Author: _Giacomo Brioschi & Martijn Biesbroek_

&nbsp;

Since virtual proxies don't exist anymore, you don't need to [create virtual proxy](https://help.qlik.com/en-US/sense-admin/May2022/Subsystems/DeployAdministerQSE/Content/Sense_DeployAdminister/QSEoW/Administer_QSEoW/Managing_QSEoW/create-virtual-proxy.htm) for JWT in the QMC to then perform an HTTP GET call to authenticate an external user. Instead, in the Management Console in SaaS you'll create a new Identity Provider and choose JWT from the available options. Then, your host application will perform an HTTP POST call to your Qlik SaaS tenant URL to authenticate an external user.


&nbsp;
___
## Considerations Before Migrating

Before you go over the how-to assets in the next section, there are few considerations with SaaS JWT. As you know, the goal is to avoid the interactive login when users access embedded content from a Qlik Sense application. This requires an integration between your solution and QSE SaaS so that users are automatically logged into QSE SaaS i.e. SSO authentication. Unlike Qlik Sense client-managed, Ticket authentication isn't an option in SaaS so the only option available is JWT.

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