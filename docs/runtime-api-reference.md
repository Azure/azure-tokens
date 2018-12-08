> Note: "Token Vault" for App Service is in Private Preview, and can be accessed by contacting our team, and completing the onboarding process under NDA. Private Preview features like "Token Vault" are primarily meant to gather feedback, and should not be used in production. Support may be discontinued in the future.

# What are runtime APIs

The runtime APIs are available through a dedicated web address for each token vault. They provide token operations like get-valid-token, in addition to supporting most of the resource management actions. Runtime APIs are meant for direct programmatic access and are better optimized for performance. The service host is typically available at `https://[token-vault-name].westcentralus.tokenvault.azure.net`.