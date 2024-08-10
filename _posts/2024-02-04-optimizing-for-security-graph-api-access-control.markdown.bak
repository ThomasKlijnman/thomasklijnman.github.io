---
id: 139
title: 'Optimizing for security: Graph API Access Control'
date: '2024-02-04T19:36:19+01:00'
author: Thomas
layout: post
guid: 'https://codeartisanjourney.com/?p=139'
permalink: /graphapi/optimizing-for-security-graph-api-access-control/
categories:
  - GraphAPI
tags:
  - API
  - ConditionalAccess
  - Powershell
---

### Why should restrictions be imposed on GraphAPI access?

Granting permissions to Entra ID App registrations for access via the Graph API to Azure tenants can provide significant benefits, particularly in service automation scenarios.

However, managing unrestricted access to these registrations can be inconvenient and a known risk. It’s crucial to establish conditional restrictions to ensure controlled access to your Azure Tenant via the Graph API.

Enter… the Conditional Access for workload identities!

First, what are workload identities:  
As Microsoft describes it, *“A workload identity is an identity you assign to a software workload (such as an application, service, script, or container) to authenticate and access other services and resources.”*

Microsoft now also offers ‘Entra Workload ID Premium’ to use workload identities within Conditional Access. These workload identities serve as the gateway, granting applications or service principals access to resources like the Graph API.

So, in this post, we connect to the Graph API as a workload identity via an IP restriction enforced by a Conditional Access policy.

<details class="wp-block-details has-vivid-red-color has-text-color has-link-color wp-elements-44103e58f4665527c37b842beb47c688 is-layout-flow wp-block-details-is-layout-flow" style="font-size:18px"><summary>Disclaimer</summary>

The instructions below assume that you have a Microsoft Entra Workload-ID license, which enables you to create Conditional Access policies based on workload identities, such as a service principal.

</details>

### Creating the named location

The journey begins with creating a named location, the first step in defining the IPs we want to exclude from our conditional access policy.

To start this, navigate to the Conditional Access management blade. ([The quick way](https://enca.cmd.ms/), thanks to Merill Fernando from [merill.net](http://merill.net)) and select named locations, as shown in the example below.

![Conditional Access Blade](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2023/12/CA-Blade-Highlight-Example.png?resize=640%2C509&ssl=1)

In the Named locations blade, I then create a new location as shown below.

![New Named Location](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2023/12/CA-Blade-NewNamedLocation.png?resize=386%2C399&ssl=1)

### Assigning the named location to a conditional access policy

To make the named location a requirement in the conditional access policy, we have to assign it to a new or existing conditional access policy.  
  
For the purpose of this post, we are creating a new policy by navigating to the Conditional Access blade and selecting the workload identity we wish to control via Conditional Access.

![New Workload ID](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2023/12/CA-Blade-NewWorkloadID.png?resize=606%2C869&ssl=1)

After selecting the workload identity to which we want to apply the restrictions, we give the policy a name and select the earlier created named location as an exclusion, so that only that location is allowed to bypass the policy.  
  
In the access controls, we select ‘Block access’ so that every other location other than the selected named location is blocked from access to the workload identity.

![Named Location in Workload ID](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2023/12/CA-Blade-NewWorkloadID-NamedLocation.png?resize=640%2C549&ssl=1)

### How does this work in practice?

As published in [my earlier blog post](https://codeartisanjourney.com/graphapi/a-simplified-guide-connecting-to-microsoft-graph-api-with-powershell/) to easily generate a Graph API access token, we are re-using some code.

When we request an access token via the trusted named location, we get the following result:

![Example Without Restriction](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2024/01/CA-Run-ExampleWithOutRestriction-1024x491.png?resize=640%2C307)

And then when we request an access token outside of our trusted named location, we get the next result back:

![Example With Restriction](https://raw.githubusercontent.com/ThomasKlijnman/thomasklijnman.github.io/main/_images/2024/01/CA-Run-ExampleWithRestriction-1024x406.png?resize=640%2C254)

### Conclusion

When creating workload identities to use in, for example, service automation products, we also need to pay attention to the access controls for these identities. This aspect can be overlooked, leading to unnecessary risk.

Unfortunately, this feature is not standard and requires additional licenses to configure.

In an upcoming post, I’ll delve into how to best maintain minimal permissions when creating a workload identity for use in automation products (scripts, etc.), as most workload identities typically do not need any more permissions than necessary.
