> Note: Azure Token Store is in Private Preview, and can be accessed by contacting our team, and completing the onboarding process under NDA. Private Preview features like Token Store are primarily meant to gather feedback, and should not be used in production. Support may be discontinued in the future.

# General recipe for creating services

In order to take advantage of Token Store against an external service (e.g. Dropbox), you first need to create a developer account them. The corresponding service will have a direct relationship with you as the app developer and can notify you of issues or unusual activity. The can also enforce security and usability limits on a per-application basis.

The following are the general set of steps required to create a service resource:

## Register your app with the service

Register an app with the service (eg Dropbox) you want to call on behalf of your users. This is typically done through the service's developer site:

1. Go to the service's **Developer site**. For example visit the [Dropbox developer site](https://www.dropbox.com/developers/apps).
1. create an account with the service using your contact information.
1. Create an app registration, and set the redirect URI to `https://[your-store-name].tokenstore.azure.net/redirect`.
1. Record the **App key** and **App secret** to be used in upcoming steps.

## Create the service resource

You can create a service using ARM templates, or programmatically from the runtime. For ARM template creation and deployment refer to [this guide](https://github.com/joerob-msft/app-service-msi-tokenstore-dotnet), and specifically [this template](https://github.com/joerob-msft/app-service-msi-tokenstore-dotnet/blob/master/azuredeploy.json). To programmatically create the service resource refer to the **Create service** section under  [Runtime API reference](/docs/runtime-api-reference.md) or [Management API reference](/docs/management-api-reference.md).

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
| Dropbox | `dropbox` | https://www.dropbox.com/developers/apps | App key (`clientid`), App secret (`clientsecret`), Redirect URIs (set on app)  |  
| Microsoft AAD Graph | `aad` | [https://portal.azure.com](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps) | Application Id (`clientid`) , Application Key(`clientsecret`), Resource URI (`ResourceUri`), Reply URI (set on app)|  See the AAD Graph section below |
| LinkedIn | `linkedin` | https://www.linkedin.com/developers | Client ID (`clientid`), Client Secret (`clientsecret`), Authorized Redirect URLs (set on app) | |
| Instagram | `instagram` | https://www.instagram.com/developer | Client ID (`clientid`), Client Secret (`clientsecret`), Scopes (`scopes`),Valid redirect URIs (set on app) | |
| Twitter | `twitter` | https://developer.twitter.com/en/apps | API key (`clientid`), API secret key (`clientsecret`), Callback URL (set on app) |
| Facebook | `facebook` | https://developers.facebook.com/apps | App ID (`clientid`), App Secret (`clientsecret`) |
| Generic OAuth 2.0| `oauth2generic`|  | App key (`clientid`), App secret (`clientsecret`), Redirect URI (set on app), and **many more additional parameters** ... | See the OAuth 2.0 generic section below |
| Google | `google` | https://console.developers.google.com/apis/dashboard | Client ID (`clientid`), Client Secret (`clientsecret`)
| Others | | [coming soon]

## Dropbox

The following instructions describe the service creation steps that may be different from the general recipe presented above.

1. Go to the [Dropbox developer portal](https://www.dropbox.com/developers).
1. **Sign in** using the link on top-right of the web site. **[Sign up](https://www.dropbox.com/register)** if you do not have an account already.
1. [Create a new app](https://www.dropbox.com/developers/apps/create), choose **Dropbox API**, **Full Dropbox** access, and create a unique name for your app.
1. Record the **App key** and **App secret** values for future use.
1. Add to **Redirect URIs** the value `https://[token-store-name].tokenstore.azure.net/redirect` where `[token-store-name]` is the name of your token store, that you will create in the next step.
1. Follow instrictions to **Create the service** above.

Try this operation out in VS Code under **Create Dropbox service** section of the [.http file](/docs/service-definition-reference.http). See the [Runtime API reference](/docs/runtime-api-reference.cs) topic for instructions on how to use the `.http` file with VS Code's Rest Client extension.

## AAD Graph

The following instructions describe the service creation steps that may be different from the general recipe presented above.

1. Go to [Azure Portal application registration](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps).
1. Create a new application, Use a unique name for your app, select **Application type** *Web app / API*, and set sign-on URL to `https://[token-store-name].tokenstore.azure.net/redirect` where `[token-store-name]` is the name of your token store, that you will create in the next step.
1. After creation record the **Application ID** values for future use.
1. Select **Settings**, then **Keys**, enter a *Key description*, select **Expires** value *Never*, then *Save*.
1. Record the Key Password Value (or **Application secret**) for future use. Note: This is the only time you will be able to retrieve the secret from the Azure portal UI.
1. Go to **Properties** and set the application property **Multi-tenanted** to *Yes*.
1. Go to **Required permissions**, click *Add*, then select the API you want to access from Microsoft Graph.
    1. To access *SharePoint* you can select **Office 365 SharePoint Online**, then check the appropriate permissions needed.
    1. To access *Microsoft Graph* itself you can select **Microsoft Graph**, then check the appropriate permissions needed.
    1. To access *Outlook* itself you can select **Office 365 Exchange Online**, then check the appropriate permissions needed.  
1. Under **Required permissions** click on **Grant permissions** as a tenant admin for permissions that need admin input.
1. Follow instrictions to **Create the service** above. The payload will look something like below:

```js
{
    "tokenParameters": {
        "resourceUri": {
            "parameterType": "string",
            "defaultValue": "https://graph.microsoft.com"
        }
    },
    "authentication": {
        "managedIdentityProvider": {
            "name": "aad"
        },
        "parameters": {
            "clientid": "[your-aad-client-id]",
            "clientsecret": "[your-aad-client-secret]",
            "resourceUri": "https://graph.microsoft.com"
        }
    },
    "displayName": "aad{{serviceName}}"
}
```

Try this operation out in VS Code under **Create AAD Graph service** section of the [.http file](/docs/service-definition-reference.http). See the [Runtime API reference](/docs/runtime-api-reference.cs) topic for instructions on how to use the `.http` file with VS Code's Rest Client extension.

## OAuth 2.0 generic

The OAuth 2.0 generic service can by used to create a customized service reosurce, and is meant for developers who deeply understand the OAuth 2.0 redirect process. Any external OAuth 2.0 service can be used, so the application registration depends on the specific external service. The following instructions describe the service creation steps that may be different from the general recipe presented above.

1. Follow instrictions to **Create the service** above. The payload for DropBox for example will look something like below:
    1. Set `clientid` and `clientsecret` to the **Client ID** and **Client Secret** values you obtained from the external service.
    1. Set the `AuthorizationUrlTemplate` and `AuthorizationUrlQueryStringTemplate` to represent the URI Token Store will redirect to on the external service.
    1. Set the `TokenUrlTemplate` and `TokenBodyTemplate` to describe the URI and data "Token Store uses to get the refresh token from the external service.
    1. Set the `RefreshUrlTemplate` and `RefreshBodyTemplate` to represent the URI and data Token Store uses to get a refreshed token from the external service.

```js
{
    "tokenParameters": {
    },
    "authentication": {
        "managedIdentityProvider": {
            "name": "oauth2generic"
        },
        "parameters": {
            "clientid": "[your-dropbox-client-id]",
            "clientsecret": "[your-dropbox-client-secret]",
            "AuthorizationUrlTemplate": "https://www.dropbox.com/oauth2/authorize",
            "AuthorizationUrlQueryStringTemplate": "?client_id={ClientId}&response_type=code&redirect_uri={RedirectUrl}&force_reapprove=true&state={State}",
            "TokenUrlTemplate": "https://api.dropbox.com/oauth2/token",
            "TokenBodyTemplate": "code={Code}&grant_type=authorization_code&redirect_uri={RedirectUrl}&client_id={ClientId}&client_secret={ClientSecret}",
            "RefreshUrlTemplate": "https://api.dropbox.com/oauth2/token",
            "RefreshBodyTemplate": "refresh_token={RefreshToken}&grant_type=refresh_token&client_id={ClientId}&client_secret={ClientSecret}"
        }
    },
    "displayName": "Oauth2Generic{{serviceName}}"
}
```

Try this operation out in VS Code under **Create OAuth 2.0 generic service** section of the [.http file](/docs/service-definition-reference.http). See the [Runtime API reference](/docs/runtime-api-reference.cs) topic for instructions on how to use the `.http` file with VS Code's Rest Client extension.

## Facebook

The following instructions describe the service creation steps that may be different from the general recipe presented above.

1. Go to the [Facebook developer portal](https://developers.facebook.com/apps).
1. **Sign in** using the link on top-right of the web site.
1. Create a new application, by choosing a unique name for your app.
1. Record the **App ID** and **App secret** values for future use.
1. Add the **Facebook Login** product on the first page, and under **Settings**, set **Valid OAuth Redirect URIs** to `https://[token-store-name].tokenstore.azure.net/redirect` where `[token-store-name]` is the name of your token store, that you will create in the next step.
1. Set rest of the required fields like *Privacy policy URL*, then *Save changes*.
1. Follow instrictions to **Create the service** above.

Try this operation out in VS Code under **Create Facebook service** section of the [.http file](/docs/service-definition-reference.http). See the [Runtime API reference](/docs/runtime-api-reference.cs) topic for instructions on how to use the `.http` file with VS Code's Rest Client extension.


## Instagram

The following instructions describe the service creation steps that may be different from the general recipe presented above.

1. Go to the [Instagram developer portal](https://www.instagram.com/developer).
1. **Sign in** using the link on top-right of the web site.
1. Create a new client, by choosing a unique name for your app.
1. Record the **Client ID** and **Client Secret** values for future use.
1. Under **Security** tab, set **Valid redirect URIs** to `https://[token-store-name].tokenstore.azure.net/redirect` where `[token-store-name]` is the name of your token store, that you will create in the next step.
1. Set rest of the required fields like *Company name*, then *Save changes*.
1. Follow instrictions to **Create the service** above, but make sure the parameter `scopes` is set to `basic`.

Try this operation out in VS Code under **Create Instagram service** section of the [.http file](/docs/service-definition-reference.http). See the [Runtime API reference](/docs/runtime-api-reference.cs) topic for instructions on how to use the `.http` file with VS Code's Rest Client extension.