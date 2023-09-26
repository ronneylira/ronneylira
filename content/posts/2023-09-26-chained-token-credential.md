---
title: Chained Token Credential
description: ""
date: 2023-09-26T10:36:26.297Z
preview: ""
draft: false
tags:
  - dotnet
  - azure
  - devXp
categories:
  - dev experience
  - azure
slug: chained-token-credential
lastmod: 2023-09-26T11:22:53.146Z
---
In this article, we'll explore a handy trick that can make your development journey more enjoyable. We're here to share some knowledge and help you understand a clever coding strategy.

Azure and Dotnet make a fantastic team, offering a bunch of powerful tools that work seamlessly together, making your coding adventures smoother. Today, we're diving into the world of Azure AD's authentication system, which can make switching between development and production modes a breeze.

Much of this code can readily be found online in documentation and code samples. Because these solutions often 'just work,' it's tempting to fall into the trap of not exploring further to see how they can be enhanced.

One of the areas where you can quickly gain traction through online research is setting up Azure services like Key Vault and App Configuration. Typically, you would come across the following configuration:

```csharp
using Azure.Identity;

var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsProduction())
{
    builder.Configuration.AddAzureKeyVault(
        new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/"),
        new DefaultAzureCredential());
}
```
Now, this is how long the service will take for your service to startup while using the ```DefaultAzureCredential```
![Scenario 1: DefaultAzureCredential](/chained-token-credential/Screenshot%202023-09-26%20124932.png)

the [DefaultAzureCredential](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet) is like your trusty friend, trying out different ways to get you logged in, step by step:

1. EnvironmentCredential
2. WorkloadIdentityCredential
3. ManagedIdentityCredential
4. SharedTokenCacheCredential
5. VisualStudioCredential
6. VisualStudioCodeCredential
7. AzureCliCredential
8. AzurePowerShellCredential
9. AzureDeveloperCliCredential
10. InteractiveBrowserCredential

But here's the fun part: when you're in development mode (imagine using Azure CLI), it goes through a bit of a treasure hunt, trying out six different methods before it gets there.

Now, here's where you get to be in charge! Think of it as creating your own playlist of favorite songs. That's what the [ChainedTokenCredential](https://learn.microsoft.com/nl-nl/javascript/api/@azure/identity/chainedtokencredential?view=azure-node-latest)
lets you do. Check this out:

```csharp
using Azure.Identity;

var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsProduction())
{
    builder.Configuration.AddAzureKeyVault(
        new Uri($"https://{builder.Configuration["KeyVaultName"]}.vault.azure.net/"),
        new ChainedTokenCredential(new AzureCliCredential(), new DefaultAzureCredential()));
}
```

In this setup, we give the Azure CLI credential the spotlight first, and then the other methods follow suit. You can customize it to your heart's content, maybe put ManagedIdentityCredential in the lead (because it's the star in production) or mix things up for different situations. The result? Your project starts up **almost a third faster**!
![Scenario 1: ChainedTokenCredential](/chained-token-credential/Screenshot%202023-09-26%20124738.png)

So there you have it – a friendly tip to make your development journey smoother and more efficient. Ready to give it a try? 😊🚀💻

