> Note: "Token Vault" for App Service is in Private Preview, and can be accessed by contacting our team, and completing the onboarding process under NDA. Private Preview features like "Token Vault" are primarily meant to gather feedback, and should not be used in production. Support may be discontinued in the future.

# What are management APIs?

The management APIs are primarily meant to enable uniform resource management along with other Azure resources, either through REST-based requests or through deployment templates. They are based on Azure Resource Monitor (ARM) and can be used for programmatic resource mangement actions like read, create, update, delete, list etc. The ARM interface also enables joint template deployment of "Token Vault" and other Azure resoruces for deploying full solutions. These APIs are typically avaiable under `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]`.


# Calling the APIs with Rest Client extension

## Pre-requisites
1. [Visual Studio Code](https://code.visualstudio.com/) installed on your machine
1. [HTTP REST Client extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) installed in Visual Studio Code
1. [Git command line](https://git-scm.com/)

## Getting started with `.http` files
1. Using command line to navigate to an empty folder and clone the repo by running the command `git clone https://github.com/Azure/azure-tokens.git`
1. Run `code azure-tokens` to open the cloned folder in Visual Studio Code.
1. Open `/docs/management-api-reference.http` in Visual Studio Code to get started.
1. Update the following variables to reflect you own Azure environment.
    1. **`@subscriptionId`** should be set to the your Azure subscription ID.
    1. **`@resourceGroup`** should be set to the a resource you have contributor access to.
1. Under the **List Vaults** section, click on the `Send request` link above `GET {{vaultsPath}}`. The *REST Client* extension will walk you through logging into Azure Resource Manager and resolve the value for `{{$aadToken}}`:
    1. Click on **Sign in** when a VS Code prompts you to sign in. The authorization code will get copied to your clipboard.
    1. Proceed through browser sign in by pasting in the authorization code and proceeding with usual AAD login.
    1. Upon completion of sign in on browser, return to VS code and click **Done**.
    1. The extension might additionally request that you specify your **tenant Id** but this is typically defaulted.
1. The **Response** window will appear for your HTTP request in VS code containing response code, headers and a body. If working correctly, it will return status code `200 OK` and response body containing a JSON list of the resource group's token vaults.

# Security

All calls to the token vault ARM APIs must bcontain a JWT security token `Authorization` header, where the JWT token in turn contains `"aud": "https://management.azure.com/"`. The principal of this token must also have appropriate access through the resource's Access Control, aka Identity Access Management (IAM) settings.

One way to obtain a security token for this purpose is by using the code below, which in turn depends on the `AzureServiceTokenProvider` library.

```cs
var azureServiceTokenProvider = new AzureServiceTokenProvider("https://management.azure.com/");
string securityToken = await azureServiceTokenProvider.GetAccessTokenAsync(TokenVaultResource);
```

# "Token Vault" management operations

## List vaults

Vaults can be listed by requesting a `GET` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults`. The result should be `200 OK` with a payload similar to below. 

```json
{
  "value": [{
    "id": "https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/testvault1",
    "name": "testvault1",
    "type": "Microsoft.TokenVault/vaults",
    "location": "westcentralus",
    "properties": {
      "redirectUrl": "https://testvault1.westcentralus.tokenvault.azure.net:443/redirect",
      "vaultUrl": "https://testvault1.westcentralus.tokenvault.azure.net",
      "authorizedPostRedirectUrls": ["http://example.com/tokens/postlogin"]
    },
  },
  ...
  ]
}
```

Try this operation out in VS Code under **List vaults** section of the[.http file](/docs/management-api-reference.http).

## Create or update vault

A vault can be created by requesting a `PUT` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]` with the payload below. The result should be `201 Created`.

```json
{
    "location": "westcentralus",
    "properties": {
        "authorizedPostRedirectUrls": [
            "http://example.com/postlogin"
        ]        
    }
}
```

Try this operation out in VS Code under **Create or update vault** section of the [.http file](/docs/management-api-reference.http).

## Read vault

A vault can be read by requesting a `GET` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]`. The resulting payload will be similar to below. 

```json
{
    "id": "https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]",
    "name": "[token-vault-name]",
    "type": "Microsoft.TokenVault/vaults",
    "location": "westcentralus",
    "properties": {
      "redirectUrl": "https://[token-vault-name].westcentralus.tokenvault.azure.net:443/redirect",
      "vaultUrl": "https://[token-vault-name].westcentralus.tokenvault.azure.net",
      "authorizedPostRedirectUrls": ["http://example.com/tokens/postlogin"]
    },
  }
```

Try this operation out in VS Code under **Read vault** section of the [.http file](/docs/management-api-reference.http).

## Delete vault

A vault can be read by requesting a `DELETE` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]`with no payload. The result should be `200 OK`.

Try this operation out in VS Code under **Delete vault** section of the [.http file](/docs/management-api-reference.http).

# Service management operations

## List services

Services can be listed by requesting a `GET` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services`. The result should be `200 OK` with a payload similar to below. 

```json
{
  "value": [{
    "id": "/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/testservice1",
    "name": "testservice1",
    "type": "Microsoft.TokenVault/vaults/services",
    "properties": {
      "tokenParameters": {},
      "authentication": {
        "managedIdentityProvider": {
          "name": "dropbox"
        }
      },
      "displayName": "testService1"
    }
  },
  ...
  ]
}
```

Try this operation out in VS Code under **List services** section of the[.http file](/docs/management-api-reference.http).

## Create or update service

A service can be created by requesting a `PUT` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]` with the payload below. The result should be `201 Created`.

```json
{
  "properties": {
    "authentication": {
      "managedIdentityProvider": {
        "name": "dropbox"
      },
      "parameters": {
        "clientid": "[dropbox-client-id]",
        "clientsecret": "[dropbox-client-secret]"
      }
    },
    "displayName": "Test Dropbox service"
  }
}
```

Try this operation out in VS Code under **Create or update service** section of the [.http file](/docs/management-api-reference.http).

## Read service

A service can be read by requesting a `GET` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]`. The resulting payload will be similar to below. 

```json
{
  "id": "/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]",
  "name": "[service-name]",
  "type": "Microsoft.TokenVault/vaults/services",
  "properties": {
    "tokenParameters": {},
    "authentication": {
      "managedIdentityProvider": {
        "name": "dropbox"
      }
    },
    "displayName": "Test Dropbox service"
  }
}
```

Try this operation out in VS Code under **Read service** section of the [.http file](/docs/management-api-reference.http).

<!-- ## Delete service

A service can be read by requesting a `DELETE` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]`with no payload. The result should be `200 OK`.

Try this operation out in VS Code under **Delete service** section of the [.http file](/docs/management-api-reference.http). -->

# Token management operations

## List tokens

Tokens can be listed by requesting a `GET` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]/tokens`. The result should be `200 OK` with a payload similar to below. 

```json
{
  "value": [{
    "id": "/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]/tokens/testtoken1",
    "name": "testtoken1",
    "type": "Microsoft.TokenVault/vaults/services/tokens",
    "location": "westcentralus",
    "properties": {
      "parameterValues": {},
      "tokenUri": "https://[token-vault-name].westcentralus.tokenvault.azure.net/services/[service-name]/tokens/testtoken1",
      "loginUri": "https://[token-vault-name].westcentralus.tokenvault.azure.net/services/[service-name]/tokens/testtoken1/login",
      "value": null,
      "status": {
        "state": "Error",
        "error": {
          "code": "Unauthenticated",
          "message": "This connection is not authenticated."
        }
      },
      "displayName": "Test token 1",
    }
  },
  ...
  ]
}
```

Try this operation out in VS Code under **List tokens** section of the[.http file](/docs/management-api-reference.http).

## Create or update token

A token can be created by requesting a `PUT` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]/tokens/[token-name]` with the payload below. The result should be `201 Created`.

```json
{
  "name": "testtoken",
  "properties": {
    "displayName": "Test Dropbox token"
  }
}
```

Try this operation out in VS Code under **Create or update service** section of the [.http file](/docs/management-api-reference.http).

## Read token

A token can be read by requesting a `GET` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]/tokens/[token-name]`. The resulting payload will be similar to below. 

```json
{
  "id": "/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]/tokens/testtoken",
  "name": "testtoken",
  "type": "Microsoft.TokenVault/vaults/services/tokens",
  "properties": {
     "parameterValues": {},
    "tokenUri": "https://[token-vault-name].westcentralus.tokenvault.azure.net/services/[service-name]/tokens/testtoken",
    "loginUri": "https://[token-vault-name].westcentralus.tokenvault.azure.net/services/[service-name]/tokens/testtoken/login",
    "value": null,
    "status": {
      "state": "Error",
      "error": {
        "code": "Unauthenticated",
        "message": "This connection is not authenticated."
      }
    },
    "displayName": "testToken"  
  }
}
```

Try this operation out in VS Code under **Read token** section of the [.http file](/docs/management-api-reference.http).

<!-- ## Delete service

A token can be read by requesting a `DELETE` operation against `https://management.azure.com/subscriptions/[subscription-id]/resourceGroups/[resource-group]/providers/Microsoft.TokenVault/vaults/[token-vault-name]/services/[service-name]/tokens/[token-name]`with no payload. The result should be `200 OK`.

Try this operation out in VS Code under **Delete token** section of the [.http file](/docs/management-api-reference.http). -->