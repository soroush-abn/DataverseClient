# Data8 .NET Client SDK for On‑Premise Dynamics 365/CRM
### Upgraded to .NET 10

> Community Fork  
> This repository is a community-maintained fork of  
> https://github.com/Data8/DataverseClient  
> upgraded to .NET 10 with updated dependencies and improved cross‑platform support.

---

## What's Changed in This Fork

This fork modernizes the original project with the following improvements:

- Upgraded to .NET 10.0
  - Removed support for .NET Framework 4.6.2
  - Removed support for .NET Core 3.1
- Updated dependencies
  - System.ServiceModel.Federation (10.0.652802)
  - Latest Microsoft.IdentityModel.* packages
- Removed NSspi dependency
  - Native NTLM authentication works on Linux with .NET 8+
  - No need for gss-ntlmssp
- Simplified package references
  - No conditional compilation
  - No platform-specific package requirements

---

## Why This Fork?

The original package targets older frameworks and has not been updated for .NET 10.

This fork updates the codebase while maintaining full compatibility with on‑premise Dynamics 365 / CRM instances using WS‑Trust authentication.

---

## Overview

The official Microsoft Dataverse SDK:

https://github.com/microsoft/PowerPlatform-DataverseServiceClient

provides an SDK for connecting to Dataverse and Dynamics 365 instances from .NET, but relies on OAuth authentication.

This can be problematic when connecting to on‑premise environments that:

- Do not support OAuth
- Require username/password authentication
- Have OAuth tokens that expire and cannot be refreshed automatically

This package builds on top of the Microsoft SDK and provides an alternative IOrganizationService implementation using WS‑Trust authentication.

This allows connecting using:

- Organization Service URL
- Username
- Password

No OAuth configuration required.

Because OnPremiseClient implements the same IOrganizationService interface as the standard ServiceClient, the same code can work with either implementation.

---

## Sample
```csharp
using Data8.PowerPlatform.Dataverse.Client;
using Microsoft.PowerPlatform.Dataverse.Client;
using Microsoft.Xrm.Sdk;

var onPremIfd = new OnPremiseClient(
    "https://org.crm.contoso.com/XRMServices/2011/Organization.svc",
    "AD\\username",
    "password!"
);

var onPremAD = new OnPremiseClient(
    "https://crm.contoso.com/org/XRMServices/2011/Organization.svc",
    "AD\\username",
    "password!"
);

var online = new ServiceClient(
    "AuthType=ClientSecret;Url=https://contoso.crm.dynamics.com;ClientId=YOUR_CLIENT_ID;ClientSecret=YOUR_SECRET"
);

CreateRecord(onPremIfd);
CreateRecord(onPremAD);
CreateRecord(online);

void CreateRecord(IOrganizationService svc)
{
    var entity = new Entity("account")
    {
        ["name"] = "Data8"
    };

    entity.Id = svc.Create(entity);
}
```
---

## Compatibility

This package is designed for on‑premise Dynamics 365 instances.

Supported authentication types:

- Integrated Windows Authentication (Active Directory)
- Claims-Based Authentication (ADFS)
- Internet Facing Deployment (IFD)

Requirements:

- .NET 10.0 or later

---

## Notes on Integrated Windows Authentication

If claims-based authentication is not configured, Dynamics 365 will use Integrated Windows Authentication (IWA).

This library can authenticate to these instances on both:

- Windows
- Linux

With .NET 8+, NTLM authentication works natively on Linux and does not require additional packages such as:

gss-ntlmssp

The .NET runtime handles NTLM negotiation internally.

You may either:

- Provide credentials (DOMAIN\Username or username@domain)
- Leave them empty to authenticate as the currently logged-on user

---

## Early Bound Support

This library supports both late-bound and early-bound programming.

Generate early-bound classes using:

CrmSvcUtil  
https://learn.microsoft.com/en-us/power-apps/developer/data-platform/org-service/generate-early-bound-classes

or

Early Bound Generator (XrmToolBox)  
https://www.xrmtoolbox.com/plugins/DLaB.Xrm.EarlyBoundGenerator/

Enable proxy types as follows:
```csharp
var client = new OnPremiseClient(url, username, password);
client.EnableProxyTypes();

Using IOrganizationService:

var contact = new Contact
{
    FirstName = "Early Bound",
    LastName = "Context"
};

contact.Id = client.Create(contact);

Using OrganizationServiceContext:

var context = new CrmServiceContext(client);

var newContacts = context.ContactSet
    .Where(c => c.CreatedOn > DateTime.Today)
    .Select(c => new { c.FirstName, c.LastName })
    .ToList();
```
---

## Async Support

In addition to IOrganizationService, this library also supports:

- IOrganizationServiceAsync
- IOrganizationServiceAsync2

This allows using async/await patterns when accessing Dynamics 365.

IOrganizationServiceAsync2 also supports cancellation.

Similar to the Microsoft client, cancellation can only occur before the request is sent to the server.  
Requests cannot be cancelled once they have been sent.

---

## Credits

Many thanks to:

- https://www.data-8.co.uk/ for developing the original library
- https://github.com/MarkMpn for the original implementation

---

## Support and Contributing

This fork is maintained on a best-effort basis.

For issues related to the .NET 10 upgrade, please open an issue in this repository.

For general usage questions, refer to the original repository:  
https://github.com/Data8/DataverseClient

Pull requests are welcome.

---

## License

MIT License — see the LICENSE file for details.
