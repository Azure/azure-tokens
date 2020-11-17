
> Note: This iteration of Azure Token Store Private Preview has been deprecated. We are working to bring this functionaly back in a more integrated way. Follow this Repo for updates in the future.

# Azure Token Store

Azure Token Store is a general purpose OAuth Token Management service for developers. It helps simplify the process of authenticating and authorizing users across SaaS services, thus reducing the level of investment needed in ramping up, implementing and maintaining security features.

# Let's get started!

1. **Deploy your first Token Store** for App Service by following [instructions on GitHub](https://github.com/joerob-msft/app-service-tokenvault-advanced).
    1. Deploy and use the Site on Azure.
    1. Clone, develop and debug locally, then deploy to Azure.
1. **Send us feedback!** Either through our discussion alias (tokens@microsoft.com) or through [issues on GitHub](https://github.com/Azure/azure-tokens/issues).
1. We are adding more reference material and walkthroughs here on this GitHub page.
1. For a more **advanced multi-user implementation** example follow these [instructions on GitHub](https://github.com/joerob-msft/app-service-tokenvault-advanced).

# References

- **[Management API reference](/docs/management-api-reference.md)**
- **[Runtime API reference](/docs/runtime-api-reference.md)**
- **[Service definition reference](/docs/service-definition-reference.md)**

# Programming interfaces

The service consists of two distinct REST-based Application Programming Interfaces (APIs), respectively for **management** and **runtime** operations. Both APIs are based on concepts of tokens, services, stores and access policy as resources. Currently there are no client libraries available for the said REST-based APIs, but this is something we are thinking about for the future.

## Resources

| Resource name | Parent resource | API availability | Supported actions |
|---------------|---|--|---|
| Store | Resource group | Management | CreateOrUpdate, Read, Delete, List |
| - Access policy | Store | Management | CreateOrUpdate, Read, Delete, List |
| - Service | Store | Management, Runtime | CreateOrUpdate, Read, Delete, List |
| - - Token | Service | Management, Runtime | CreateOrUpdate, Read, Delete, List, Login, Save, GetAccessToken |

**Token** resource abstracts the data and functionality around OAuth 2.0 tokens. The user can perform login action on a specific token resource, which then abstracts the refresh and access tokens. The web application can then call the GetAccessToken action, and use the access token at runtime to call the external service on behalf of the user.

**Service** resource represents a specific external service and its acoompanying authentication settings. We provide a list of managed service definitions that can be used to create a service resouce with minimal complexity. Usually client ID and client secret are the only parameters that need to be assigned upon service resource creation.

**Store** resource is a container of tokens and services, and represents the scope at which billing takes effect, as well as the scope at which Azure resource operations can be performed in ARM.

**Access policy** resources determine how security is applied to runtime operations. Access policies are inherited, i.e. those set at the Store level apply to the actions (CreateOrUpdate, Read, Delete, List) on Service and Token resource. The principal of user who creates the store is added with full access policy by default.

## Runtime APIs

The runtime APIs are available through a dedicated web address for each token store. They support most of the resource management actions, however they also support the all-important token operations like get-access-token. They are meant for direct programmatic use and are more optimized for performance. The service host is `https://[token-store-name].tokenstore.azure.net`.

See **[Runtime API reference](/docs/runtime-api-reference.md)** for more details.

## Management APIs (on Azure Resource Manager)

The management APIs are primarily meant to enable uniform resource management along with other Azure resources, either through REST-based requests or through deployment templates. They are based on Azure Resource Management (ARM) and can be used for programmatic resource mangement actions like read, create, update, delete, list etc. The ARM interface also enables joint template deployment of Token Store and other Azure resoruces, in form of full solutions. The web address for these APIs are under `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.Token`.

See **[Management API reference](/docs/management-api-reference.md)** for more details.

*Stores*, *access policies*, *services* are strictly managed by the developer. *Tokens* are also sometimes managed by the developer, however they can be assigned to, and accessed by users through the web application runtime, depending on the scenario.

# Managed service definitions

We will have support for a long list of OAuth-based SaaS services in the future, and will also support customization. However for private preview, our support is deliberately limited to a list of close to 10 services. Most samples povide instructions on using Dropbox, however here we provide more details about a few of the other common services.

See **[Service definition reference](/docs/service-definition-reference.md)** for more details.

# Security

## Phishing attack vulnerability

The redirect pattern used in OAuth 2.0 authorization, is vulnerable to a certain class of phishing attacks. You should not worry about this, if you are a developer who is just getting stared with using Token Store. However when you integrate Token Store with your production web application, implementing a post-login redirect pattern is needed to mitigate the said class of vulnerabilities. See **[Phishing attack vulnerability](/docs/phishing-attack-vulnerability.md)** for more details.

# Appendix

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Legal Notices

Microsoft and any contributors grant you a license to the Microsoft documentation and other content
in this repository under the [Creative Commons Attribution 4.0 International Public License](https://creativecommons.org/licenses/by/4.0/legalcode),
see the [LICENSE](LICENSE) file, and grant you a license to any code in the repository under the [MIT License](https://opensource.org/licenses/MIT), see the
[LICENSE-CODE](LICENSE-CODE) file.

Microsoft, Windows, Microsoft Azure and/or other Microsoft products and services referenced in the documentation
may be either trademarks or registered trademarks of Microsoft in the United States and/or other countries.
The licenses for this project do not grant you rights to use any Microsoft names, logos, or trademarks.
Microsoft's general trademark guidelines can be found at http://go.microsoft.com/fwlink/?LinkID=254653.

Privacy information can be found at https://privacy.microsoft.com/en-us/

Microsoft and any contributors reserve all others rights, whether under their respective copyrights, patents,
or trademarks, whether by implication, estoppel or otherwise.
