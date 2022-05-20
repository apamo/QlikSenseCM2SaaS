# From Client-Managed to SaaS: **Authorization Model**

## Introduction
As of May 2022, Authorization model in SaaS offers limited granular security permissions to control access to resources and capabilities in SaaS.  
Future improvements are underway to provide a fine grained security model.
Current authorization model has the following characteristics:
- User and Group attributes provided by custom IdP (OIDC compatible) or JWT auth in SaaS.
- Groups only available when using your custom IdP or JWT auth in SaaS.
- Spaces for collaboration and application level security driven by User / Groups associated with roles (i.e. view, edit, admin, publish)​
- Section Access for data authorization supports row-level security for users or groups (provide by custom IdP or JWT token).

IMAGE Security Model  
Author: *Levi Turner & Daniel Pilla*

&nbsp;
___
## Key Differences
- In SaaS we don't have role-based access control (RBAC) through the implementation of security rules as in client-managed

![Security rule in client-managed for Developer stream access](https://help.qlik.com/en-US/sense-admin/February2022/Subsystems/DeployAdministerQSE/Content/Resources/Images/ui_Security_rule_TestStream1_pt2.png)

&nbsp;

- Place to store apps and other assets are now called `Spaces` in SaaS instead of Streams. 

IMAGE Spaces  
Author: *Levi Turner & Daniel Pilla*
&nbsp;

- There are three types of Spaces for different purposes: personal, shared and managed.

IMAGE Space Types  
Author: *Levi Turner & Daniel Pilla*
&nbsp;

- Data authorization (Section Access) use either USERID or USER.EMAIL (not both) for user-level security​. See this [article](https://github.com/apamo/QlikSenseCM2SaaS/blob/main/Section%20Access.md) for more detailed information about migrating your SA security table.

IMAGE Section Access
Author: *Levi Turner & Daniel Pilla*

&nbsp;
___
## Considerations Before Migrating

- Professional end-users can create apps and other assets from scratch - No sensible controls available. That means the `"+ Add new"` button available in the Cloud Hub cannot be hidden/disabled for users with professional entitlement.
- As we don't security rules we cannot create one global rule to control access to customer Spaces dynamically as it's typically done in OEM scenarios.

IMAGE Security rule for OEM

- Single SaaS tenant is not designed to host all the OEM partner's customers due to limited multitenancy support:
    - When creating Data Alerts you can list **all users available inside the tenant**.
    - When using Collaborative Notes you can see/tag **all users available inside the tenant**.
    - When using automations the `List Users` block will return **all users available inside the tenant**.
- OEM partners are encouraged and should use a multiple tenants architecture where every customer has its own SaaS tenant. This will allow tight access control to resources and capabilities/features in SaaS. The only exception to this is when the use case is very simple e.g. read only (Analyzer) with all add-on capabilities/features disabled i.e. alerts, notes, automations, reporting, etc.

&nbsp;
___
## Space Access Management for OEM
**This is work in progress** as R&D works on the new fine grained security model.

&nbsp;  
**This is still a roadmap item - no commitment from Qlik - dates and final functionality subject to change at our sole discretion*

