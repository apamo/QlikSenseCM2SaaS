# From Client-Managed to SaaS: **QSEoW JWT to SaaS JWT**

## Introduction




&nbsp;
___
## Key Differences



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