> Note: "Token Vault" for App Service is in Private Preview, and can be accessed by contacting our team, and completing the onboarding process under NDA. Private Preview features like "Token Vault" are primarily meant to gather feedback, and should not be used in production. Support may be discontinued in the future.

# General recipe for creating services

In order to take advantage of "Token Vault" against an external service (e.g. Dropbox), you first need to create a developer account them. The corresponding service will have a direct relationship with you as the app developer and can notify you of issues or unusual activity. The can also enforce security and usability limits on a per-application basis.

The following are the general set of steps required to create a service resource:

## Register your app with the service

Register an app with the service (eg Dropbox) you want to call on behalf of your users. This is typically done through the service's developer site:

1. Go to the service's **Developer site**. For example visit the [Dropbox developer site](https://www.dropbox.com/developers/apps).
1. create an account with the service using your contact information.
1. Create an app registration, and set the redirect URI to `https://[your-vault-name].westcentralus.tokenvault.azure.net/redirect`.
1. Record the **App key** and **App secret** to be used in upcoming steps.

## Create the service resource

You can create a service using ARM templates, or programmatically from the runtime. For ARM template creation and deployment refer to [this guide](https://github.com/joerob-msft/app-service-msi-tokenvault-dotnet), and specifically [this template](https://github.com/joerob-msft/app-service-msi-tokenvault-dotnet/blob/master/azuredeploy.json). To programmatically create the service resource refer to the **Create service** section under [Management API reference](/docs/management-api-reference.md).

You need to set the following parameters on the service:

1. The managed service name can be looked up in the table below under **Managed service name** corresponding to your service. See `[managed-service-name]` below
1. The relevant **parameters** for **App key** and **App secret**. See `[service-client-id]` and `[service-client-secret]` below.
1. Depending on the service, you may have to set additional parameters (e.g. `scope`). You can lookup these values in the table below. 

The service creation payload generally looks like below:

```json
{
  "properties": {
    "authentication": {
      "managedIdentityProvider": {
        "name": "[managed-service-name]"
      },
      "parameters": {
        "clientid": "[service-client-id]",
        "clientsecret": "[service-client-secret]",
        "otherserviceparameter" : "[other-service-parameter]"
      }
    },
    "displayName": "[service-display-name]"
  }
}
```

# Our managed service definitions

| Service name   |  Managed service name | Developer site  |   Parameters   |  Special instructions | 
|-----|-------|--------|-------|--------|
| Dropbox | `dropbox` | https://www.dropbox.com/developers/apps | App key (`clientid`), App secret (`clientsecret`), Redirect URIs(set on app)  |  
| Twitter | `twitter` | https://developer.twitter.com/en/apps | API key(`clientid`), API secret key(`clientsecret`), Callback URL(set on app) |
| Facebook | `facebook` | https://developers.facebook.com/apps | App ID(`clientid`), App Secret(`clientsecret`) |
| Generic OAuth 2.0| `oauth2generic`| Refer to service developer page. | Typically: App key(`clientid`), App secret(`clientsecret`), Redirect URI, and many more ... | [Create an OAuth 2.0 generic service resource](\service-definition-reference\oauth2-generic.md) |
| [More coming soon] | | | |  

## Dropbox

1. Go to the [Dropbox developer portal](https://www.dropbox.com/developers).
2. **Sign in** using the link on top-right of the web site. **[Sign up](https://www.dropbox.com/register)** if you do not have an account already.
3. [Create a new app](https://www.dropbox.com/developers/apps/create), choose **Dropbox API**, **Full Dropbox** access, and create a unique name for your app.
4. Record the **App key** and **App secret** values for future use.
5. Add to **Redirect URIs** the value `https://[token-vault-name].westcentralus.tokenvault.azure.net/redirect` where `[token-vault-name]` is the name of your token vault, that you will create in the next step. 
1. Follow instrictions to **Create the service** above.