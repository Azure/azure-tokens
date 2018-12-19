> Note: "Token Vault" for App Service is in Private Preview, and can be accessed by contacting our team, and completing the onboarding process under NDA. Private Preview features like "Token Vault" are primarily meant to gather feedback, and should not be used in production. Support may be discontinued in the future.

# General recipe for creating services

"Token Vault" requires you to create a developer account with the service that you want your users to authenticate with. This way the corresponding service, say Dropbox for example, can have a direct relationship with you as a developer and can notify you of issues or unusual activity.

In general, the following are the required steps to create a service resource:

## Register your app with the service

Register an app with the service you want to call on behalf of your users. This is typically done through the service's developer site:

1. Go to the service's **Developer site**. For example go to the [Dropbox developer site](https://www.dropbox.com/developers/apps).
1. create an account with the service using your contact information.
1. Create an app registration, and set the redirect URI to `https://[your-vault-name].westcentralus.tokenvault.azure.net/redirect`.
1. You may have to authorize the domain `[your-vault-name].westcentralus.tokenvault.azure.net` depending on the service.
1. Get the **App key** and **App secret** for use in upcoming steps.

## Create the service resource

You can create a service using ARM templates, or programmatically from the runtime. For ARM template creation and deployment refer to [this guide](https://github.com/joerob-msft/app-service-msi-tokenvault-dotnet), and specifically [this template](https://github.com/joerob-msft/app-service-msi-tokenvault-dotnet/blob/master/azuredeploy.json). To programmatically create the service resource refer to the **Create service** section under [Management API reference](/docs/management-api-reference.md).

You need to set the following parameters on the service:

1. The managed service name can be looked up in the table below under **Managed service name** corresponding to your service. See `[managed-service-name]` below
1. The relevant **parameters** for **App key** and **App secret**. See `[service-client-id]` and `[service-client-secret]` below.

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
| Dropbox | `dropbox` | https://www.dropbox.com/developers/apps | App key, App secret, Redirect URIs  |  [Create a Dropbox service resource](\service-definition-reference\dropbox.md)
| Twitter | `twitter` | https://developer.twitter.com/en/apps | API key, API secret key, Callback URL |
| Facebook | `facebook` | https://developers.facebook.com/apps | App ID, App Secret |
| Generic OAuth 2.0| `oauth2generic`| Refer to service developer page. | Typically: App key, App secret, Redirect URI |
| [More coming soon] | | | |  

## Dropbox

1. Go to the [Dropbox developer portal](https://www.dropbox.com/developers).
2. **Sign in** using the link on top-right of the web site. **[Sign up](https://www.dropbox.com/register)** if you do not have an account already.
3. [Create a new app](https://www.dropbox.com/developers/apps/create), choose **Dropbox API**, **Full Dropbox** access, and create a unique name for your app.
4. Record the **App key** and **App secret** values for future use.
5. Add to **Redirect URIs** the value `https://[token-vault-name].westcentralus.tokenvault.azure.net/redirect` where `[token-vault-name]` is the name of your token vault, that you will create in the next step. 
1. Follow instrictions to **Create the service** above.