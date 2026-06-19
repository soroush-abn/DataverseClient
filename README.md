# Data8 .NET Client SDK for On-Premise Dynamics 365/CRM (Upgraded to .NET 10)

> **This is a community fork** of the original [Data8.PowerPlatform.Dataverse.Client](https://github.com/Data8/DataverseClient) upgraded to .NET 10 with updated dependencies and improved cross-platform support.

## What's Changed in This Fork

This fork includes the following improvements:

- **✅ Upgraded to .NET 10.0** - Removed .NET Framework 4.6.2 and .NET Core 3.1 support
- **✅ Updated dependencies** - Using latest System.ServiceModel.Federation (10.0.652802) and Microsoft.IdentityModel packages
- **✅ Removed NSspi dependency** - Native NTLM authentication now works on Linux with .NET 8+ without requiring `gss-ntlmssp`
- **✅ Simplified package references** - No conditional compilation or platform-specific packages needed

### Why This Fork?

The original package targets older frameworks and has not been updated for .NET 10. This fork brings the codebase forward while maintaining full compatibility with on-premise Dynamics 365/CRM instances using WS-Trust authentication.

---

## Overview

The [Microsoft.PowerPlatform.Dataverse.Client](https://github.com/microsoft/PowerPlatform-DataverseServiceClient)
package provides an SDK for connecting to Dataverse & Dynamics 365 instances from .NET, but relies on OAuth
authentication. This poses a problem when you need to connect to an on-premise instance that either does not support
OAuth, or where the OAuth tokens regularly expire and cannot be automatically refreshed.

This package builds on top of the Microsoft one and offers an alternative `IOrganizationService` implementation using WS-Trust.
This allows you to connect using the URL of the organization service, username and password without any additional
configuration.

Because this `OnPremiseClient` implements the same `IOrganizationService` as the standard `ServiceClient` implementation,
your code can work with either as shown in the sample code below.

## Sample
```csharp
using Data8.PowerPlatform.Dataverse.Client;
using Microsoft.PowerPlatform.Dataverse.Client;
using Microsoft.Xrm.Sdk;

var onPremIfd = new OnPremiseClient("https://org.crm.contoso.com/XRMServices/2011/Organization.svc", "AD\\username", "password!");
var onPremAD = new OnPremiseClient("https://crm.contoso.com/org/XRMServices/2011/Organization.svc", "AD\\username", "password!");
var online = new ServiceClient("AuthType=ClientSecret;Url=https://contoso.crm.dynamics.com;ClientId=637C79F7-AE71-4E9A-BD5B-1EC5EC9F397A;ClientSecret=p1UiydoIWwUH5AdMbiVBOrEYn8t4RXud");

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

## Compatibility

This package is designed to be used with on-premise Dynamics 365 instances. Supported authentication types are:

* Integrated Windows Authentication (AD)
* Claims-Based Authentication (ADFS)
* Internet Facing Deployment (IFD)

**Requirements:**
- **.NET 10.0 or later**

## Notes on Integrated Windows Authentication

If claims-based authentication is not configured on your Dynamics 365 instance, you will be using Integrated Windows
Authentication. This library can authenticate to these instances on both Windows and Linux.

**With .NET 8+**, NTLM authentication works natively on Linux **without any additional packages** like `gss-ntlmssp`. The runtime handles NTLM negotiation directly.

You can choose to supply a username (in the format `DOMAIN\Username` or `username@domain`) and password, or leave
both blank to authenticate as the currently logged on user.

## Early Bound Support

This library supports both late-bound and early-bound code. To use early-bound classes, create them using
[CrmSvcUtil](https://learn.microsoft.com/en-us/power-apps/developer/data-platform/org-service/generate-early-bound-classes)
or the [Early Bound Generator](https://www.xrmtoolbox.com/plugins/DLaB.Xrm.EarlyBoundGenerator/) tool in XrmToolBox
as normal. You'll then need to enable them using the `EnableProxyTypes` method in the same way as the CrmServiceClient uses.
After they are enabled you can pass your early bound instances and get strongly typed responses from all the normal
`IOrganizationService` methods, or use the generated OrganizationServiceContext to interact with the service:

csharp
var client = new OnPremiseClient(url, username, password);
client.EnableProxyTypes();

// Using IOrganizationService
var contact = new Contact { FirstName = "Early Bound", LastName = "Context" };
contact.Id = client.Create(contact);

// Using OrganizationServiceContext
var context = new CrmServiceContext(client);
var newContacts = context.ContactSet
.Where(c => c.CreatedOn > DateTime.Today)
.Select(c => new { c.FirstName, c.LastName })
.ToList();

## Async Support

As well as the standard `IOrganizationService` interface, this library also supports both the `IOrganizationServiceAsync`
and `IOrganizationServiceAsync2` interfaces to allow you to use async programming styles when accessing Dynamics 365 data.

The `IOrganizationServiceAsync2` interface supports cancellation. Similar to the Microsoft library, cancellation can only
be performed up to the point the request is sent to the server. Requests cannot be cancelled once they are sent.

## Credits

Many thanks to:
- **[Data8](https://www.data-8.co.uk/)** for developing the original library and releasing it for public use
- **[MarkMpn](https://github.com/MarkMpn)** for the original implementation

## Support and Contributing

This fork is maintained on a best-efforts basis. For issues specific to the .NET 10 upgrade, please open an issue in this repository. For general usage questions, refer to the [original repository](https://github.com/Data8/DataverseClient).

Contributions are welcome via Pull Requests.

## License

MIT License - see [LICENSE](LICENSE) file for details.