Qlik Sense SaaS has a different security setup as compared with Qlik Sense Client Managed (CM). Please note CM is the new name for Qlik Sense on Windows. 

In this document we make a split for security for 
- Authentication: Who is the user?
- Authorization: What can he do?

We also assume you have a multi-tenant SaaS platform in which you want to embed Qlik Sense. With multi-tenant we mean that you have a web server on which you host data of multiple customers.

In the end you want to provide an integrated view of Qlik Sense in your software. The user should be able to access only his authorized data in a seamless way. Also the user should be able to view Qlik Sense without having to authenticate again (SSO). You also want to make sure that the right Qlik Sense content is shown in an automated way with a minimum amount of manual work.

## Difference SaaS and CM: authentication overview
In SaaS you can use OIDC or JWT to authenticate. For embedding/OEM use cases normally JWT is used. For OIDC you have the following options:

![image](https://user-images.githubusercontent.com/12411165/166262153-2aa15a45-f272-43fb-bcea-c6f4bb133a82.png)


OIDC and JWT are both also available on Qlik Sense CM. (But CM has more options like header, SAML, ticketing, AD, that are not available on SaaS.)

There is another difference: the use of developer keys. Developer keys are only used on SaaS, they enable a developer to make code which executes API calls on a users behalf. If you create a key the key will have the same access rights as the user who created the key. So if John creates a key, and Simon uses the key, he will have the same rights as John for these API calls. Use the links below for more information
- [qlik.dev, create your first API key](https://qlik.dev/tutorials/generate-your-first-api-key)
- [Qlik help](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Admin/mc-generate-api-keys.htm)

### Difference SaaS and CM: OIDC
There should not be a lot of difference between SaaS and CM use of OIDC. 

OIDC guides:
- [Client managed OIDC guide](https://help.qlik.com/en-US/sense-admin/February2022/Subsystems/DeployAdministerQSE/Content/Sense_DeployAdminister/QSEoW/Administer_QSEoW/Managing_QSEoW/OIDC-configuration-Auth0.htm), 
- [SaaS OIDC guide](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Admin/OIDC-intro.htm)

### Difference SaaS and CM: JWT
JWT does have some slight differences. The idea is the same but the endpoints and the contents of the JWT have changed. The basic idea is still that you
- create a jwt on the server side of your host application (your web server)
- when you open a Qlik Sense page you include the JWT in the header of the request
- Qlik Sense will now supply a cookie to be used in further requests. 

#### High level flow
In client managed (CM) the flow was like this:

![image](https://user-images.githubusercontent.com/12411165/166260604-c7b1c90d-c8d1-40c9-92c3-1ca2fa04056f.png)

In SaaS we don't have virtual proxies anymore. There is just one URL to use: your Qlik SaaS tenant. The flow in SaaS is like this

![image](https://user-images.githubusercontent.com/12411165/166661007-ad2b1e5e-788b-433c-9280-c96dd0526c93.png)

or more detailed:

![image](https://user-images.githubusercontent.com/12411165/166661328-ab1adef5-e7fe-4700-bde6-489524504fb8.png)

Note: that when you receive your session cookie, you can just open the hub an app, or do anything you like with the APIs. 

#### JWT contents
Before we go into the details, what is a JWT? JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object.

A JWT consists of three parts: a header, a payload, and a signature.

![image](https://user-images.githubusercontent.com/12411165/166662255-10bfb81a-02c0-458b-8f5d-7defd3e2ee48.png)

So a JWT is basically like a passport, you created it in a secure environment by a trusted party (the government), once you created it you provide some tools/scanners that can validate whether the passport is real (the `verify signature` part by means of a hash). 

Back to our multi-tenant environment, so once you created the JWT/passport and share it with another system (like Qlik SaaS), it can also see who the user is, and to what groups it belongs. So in these groups you can put your tenant name/id, the groups the user has access to inside the tenant. 

In the image below you can find a javascript example on how to make a jwt.

![image](https://user-images.githubusercontent.com/12411165/166662122-ac3a4b17-a1b8-468f-a4dd-cb00443ee657.png)

#### JWT key difference with CM

The required claims for a Qlik JWT payload are the following:

- jti - (JWT ID) claim provides a unique identifier for the JWT.
- sub - The main identifier (aka subject) of the user.
- subType - The type of identifier the sub represents. In this case, user is the only applicable value.
- name - The friendly name to apply to the user.
- email - The email address of the user.
- email_verified - A claim indicating to Qlik that the JWT source has verified that the email address belongs to the subject.

Two new required attributes for the JWT are (June 2022)
- jti: unique string value prevents reuse of jwt from different origin
Typically part of jwt payload
- nbf: (aka notBefore) is a timestamp specifying when the jwt becomes valid for use
Typically part of jwt signing options


#### Manuals
SaaS JWT implementation guides
- [Qlik Sense Saas: mashup with JWT authentication](https://github.com/jackBrioschi/Basic_Mashup_with_JWT_Authentication_Qlik_Sense_SaaS). The goal of this project is to show how it works a mashup on Qlik Cloud Services with a JWT authentication. In particular, the use case I developed is related to a single cloud tenant environment and uses a combination of REST and Javascript APIs to set-up a mashup integration.
- [Create Signed Tokens for JWT Authorization](https://qlik.dev/tutorials/create-signed-tokens-for-jwt-authorization) Configure Qlik Sense SaaS tenant to use JWT for authorization
- [Implement JWT Authorization](https://qlik.dev/tutorials/implement-jwt-authorization) Configure a Qlik Cloud tenant to use JWT authorization

[Setup JWT for windows](https://integration.qlik.com/?selection=ALkrMhX8JgMMtgRcJ)

## Difference SaaS and CM: authorization overview
We make a division for 
- functional authorizations (what can you do? Read or edit, or publish and app or sheet etc.)
- data authorizations (what level of detail can you see?, which organizational levels like countries, regions, cities, departments etc.)

### Functional authorizations
Today in SaaS, data [security and governance](https://help.qlik.com/en-US/cloud-services/Subsystems/Hub/Content/Sense_Hub/Introduction/data-security-governance.htm) in Qlik Sense Enterprise SaaS is achieved through spaces. Spaces are managed in the Management Console on the Spaces page. So the basic principle is that you create a space and assign a role to it. 

In CM we made use of [security rules](https://youtu.be/sdCVsMzTf64?list=PLqJfqgR62cVAZxS34WGnByjASKrGf0Fpk) to limit access to functionality. That feature has been removed in SaaS. The authorization system in SaaS is currently being migrated to a more advanced system using roles. Those new features will be delivered in the future.

The basic approach for SaaS, if you want embed a multi-tenant SaaS application is that you
- create a tenant for customer
- use spaces to manage security within a customer. (e.g. for each module of your software you create a different space: finance, logistics, HR etc.)

![image](https://user-images.githubusercontent.com/12411165/166256032-4a313610-f60f-41ce-8972-3ed2236e134b.png)

Qlik SaaS ingests the user information and propagates it to multiple components. Using a properly configured Identity Provider, upon authentication a user’s metadata, including their userId and group information can be fed into two principal areas:
- Spaces
- Apps

We’ll cover spaces in more detail next, but the key take-away is that the same identity is being used for both access to resources as well as access to columns or rows when using Qlik’s technique for column and/or row level security, Section Access

Spaces come in 3 types to give you control about what users can do to an app
![image](https://user-images.githubusercontent.com/12411165/166256470-009146b0-549c-47cd-85e0-228fff938c07.png)

#### Personal spaces
Your personal space is your own private work area in the cloud hub. You cannot share apps from your personal space and other users cannot collaborate with you.

#### Shared spaces
Shared spaces are used to develop apps collaboratively and share them with other users in the space. A team might have a shared space for the private development and consumption of their own apps.

Shared spaces can be created by any user with a professional license.

#### Managed spaces
Managed spaces are used for providing governed access to apps with strict access control both for the app and the app data. In a managed space, only the space owner and target app consumers can open the apps. No other users can open the apps in the managed space unless they have been given permission.

Apps in managed spaces are developed in personal or shared spaces and published to the managed space. Members need the Can publish permission in the managed space to publish.

Managed spaces can only be created by tenant or analytics administrators, or users with the ManagedSpaceCreator role.


### Data authorizations (section access)
Section access is basically the same as in CM. Emails can be used in section access

![image](https://user-images.githubusercontent.com/12411165/166264126-d5335fab-15d2-47f1-b850-a386ee2be3d3.png)

#### Entitlements
Entitlements Associate with Data Model to drive data visibility
Associate one data element to drive visibility across entire data model
Row and column level
Use either USERID or USER.EMAIL (not both) for user-level security


![image](https://user-images.githubusercontent.com/12411165/166262478-578683cc-c16c-4b8e-ab8c-2cfb77d71554.png)



### Subject attribute
The subject is critical to Qlik, as it can be used for Section Access and must match verbatim for hybrid deployments
By default, most IdPs will send a random unique ID that is not user friendly
Likely want to change to something like <DOMAIN>\<UserID> so that it is readable and so that the user will match verbatim in other Qlik environments
![image](https://user-images.githubusercontent.com/12411165/166264441-7a2c415b-8226-4b2a-bd43-ab1b0457fe50.png)
  
  
# OIDC and authorization setup tips
  
## SaaS additional authentication options
![image](https://user-images.githubusercontent.com/12411165/166253846-c4ba9d3f-7268-4f02-96b2-5874e53b3118.png)

## OIDC mappings

![image](https://user-images.githubusercontent.com/12411165/166262765-17acba61-c85d-4565-b7ab-47d6da3daea3.png)
  
You can re-use the groups of your host application. Be aware that you first have to login with a user and (a lot of groups) in order for the groups to appear in the drop down list next to a space. 

For groups to be enabled, they must be toggled on under Settings![image](https://user-images.githubusercontent.com/12411165/166254535-71dfbd7c-9ab2-473e-ba6d-e25a2cd07431.png)

- Groups are created when a user logs in
- Can be pre-populated for Space assignment by using a “Dummy” user that contains many groups and is then removed
- Array of groups truncated to 100 on login – current limit

By default, Qlik allows for automatic entitlements of Professional and/or Analyzer users
- If both, Professional entitlements will first be exhausted, followed by Analyzer
- Currently no ability to filter by group
- Can offload filtering by groups to the IdP to restrict access to Qlik

You can use the CLI or other API method to programmatically inject users into Qlik and set their license based on a value, e.g. user details and desired license type in a flat file

Make sure you validate your claims mapping as shown below. ![image](https://user-images.githubusercontent.com/12411165/166255119-197f2761-59da-49dc-ae71-a02b65cbd6de.png)

## Diagnose claims
Navigate to https://<tenant>.<region>.qlikcloud.com/api/v1/diagnose-claims
Useful post-IdP setup to see groups/what else IdP is sending and how they are mapped
  
![image](https://user-images.githubusercontent.com/12411165/166255680-aad1aef5-92d8-48da-8377-c2f2147ed206.png)

## OIDC ID token
  Qlik leverages the claims from the ID Token, so any attributes to be leveraged, e.g. groups, must be included there and not the Access Token.

  ![image](https://user-images.githubusercontent.com/12411165/166255760-d77667a5-53b2-43d5-8a8a-baa889d8d6aa.png)


# Source
Images and text from this document also originate from Levi Turner and Daniel Pilla. Thanks for the great assets guys!

  
  &nbsp;
___
Written by *Martijn Biesbroek & Alvaro Palacios*  
OEM EMEA Presales Team at Qlik   
[www.qlik.com](https://www.qlik.com)
