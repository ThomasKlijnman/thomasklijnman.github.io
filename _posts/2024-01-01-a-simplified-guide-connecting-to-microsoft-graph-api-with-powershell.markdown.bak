---
id: 80
title: 'A simplified guide: Connecting to the Graph API with PowerShell'
date: '2024-01-01T16:15:03+01:00'
author: Thomas
excerpt: 'In one of my recent side quests, I delved into the realm of the Microsoft Online API services, exploring ways to efficiently specific information from those services using the Microsoft Graph API.'
layout: post
guid: 'https://codeartisanjourney.com/?p=80'
permalink: /graphapi/a-simplified-guide-connecting-to-microsoft-graph-api-with-powershell/
categories:
    - GraphAPI
tags:
    - API
    - Microsoft365
    - Powershell
---

In one of my recent side quests, I delved into the realm of the Microsoft API services, exploring ways to efficiently retrieve specific information from those services using the Microsoft Graph API. Traditionally, that process for me involved navigating through one or more individual Powershell modules with predefined cmdlets.

However, I discovered a more direct route; leveraging HTTPs calls for querying data.

By opting for HTTPs calls over Powershell modules you can simplify the method for making and posting requests and even throw out your module dependency.

In this blog post I aimed to optimize the approach you need to make for connecting to the Microsoft Graph API and simplify the data retrieval process, one of the possible connection methods to this approach lies involves obtaining an expiring access token through an Entra ID App registration with a client secret.

In an upcoming blog post, I will even delve deeper into the ways of restricting authentication to Microsoft Graph API services using this access token method.

## Simplifying the connection

Connecting to one of the Microsoft online services via the underlying API can sometimes be a sequence of multiple tasks, especially when dealing with multiple Powershell modules. However, by utilizing HTTPs calls, the process becomes more straightforward and consolidated.

The simplicity of this method allows for a seamless interaction with the array of API services offered by Microsoft without the need for multiple Powershell modules.

## Access Token Magic

<details class="wp-block-details has-vivid-red-color has-text-color has-link-color has-small-font-size wp-elements-d122976472ec6003c19f8ad69b44c955 is-layout-flow wp-block-details-is-layout-flow"><summary>Disclaimer</summary>This section assumes that you are already familiar with and have set up an Entra ID App registration, along with the addition of API permissions and a client secret.

</details>  
Let’s dive straight into the scripting part. I’ve created a few simple functions in PowerShell that send and retrieve an access token.

See the below scripts that I’ve used for this example:

```
# Build the authorization header
$tokenUrl = "https://login.microsoftonline.com/<em><strong>< Tenant ID</strong> ></em>/oauth2/token"
$body = @{
    'resource' = 'https://graph.microsoft.com'
    'client_id' = <strong><em>< App registration ID ></em></strong>
    'client_secret' = <strong><em>< App registration client secret ></em></strong>
    'grant_type' = 'client_credentials'
}

# Obtain the access token
$tokenResponse = Invoke-RestMethod -Method Post -Uri $tokenUrl -ContentType "application/x-www-form-urlencoded" -Body $body
$accessToken = $tokenResponse.access_token
```

In simple terms, the above code requests an access token from login.microsoftonline.com, which later can be used in the below function where I use that earlier generated access token to make an HTTPs call to the Graph API.

```
# Function to invoke the Graph API
function Invoke-GraphApi {
    param (
        [string]$ApiEndpoint,
        [string]$Method = 'GET',
        [string]$Body = $null,
        [string]$Version = 'v1.0'
        [string]$ContentType = 'application/json'
    )

    # Set the base URL for the Graph API
    $baseUrl = "https://graph.microsoft.com/$Version"

    # Authorize with the dynamically obtained access token
    $authorization = @{
        'Authorization' = "Bearer $accessToken"
        'Content-Type' = 'application/json'
    }

    # Build the full URL
    $url = "$baseUrl$ApiEndpoint"

    # Execute the HTTP call based on the specified method
    $response = Invoke-RestMethod -Uri $url -Headers $authorization -Method $Method -Body $Body -ContentType $ContentType

    # Return the response
    $response
}

```

<details class="wp-block-details has-small-font-size is-layout-flow wp-block-details-is-layout-flow"><summary>Technical note</summary>Due to limitations in the underlying method used in the .NET Framework by the Invoke-RestMethod, the body parameter can only be used when the HTTP method is not GET. That’s why this script works in Powershell 7. But can be modified to be used in Powershell 5.1.

</details>  
Using both the authorization and function together gives the possibility to make a variety of requests, ranging from simple GET requests to more complex POST and PATCH requests, this along with incorporating parameters in the request body, it enables a simple approach to handling diverse types of requests with ease using the same method.

Well.. How easy can this be?

<figure class="wp-block-image size-large wp-lightbox-container" data-wp-context="{"uploadedSrc":"http:\/\/codeartisanjourney.com\/wp-content\/uploads\/2023\/12\/AccessTokenExample.png","figureClassNames":"wp-block-image size-large","figureStyles":null,"imgClassNames":"wp-image-123","imgStyles":null,"targetWidth":1037,"targetHeight":419,"scaleAttr":false,"ariaLabel":"Enlarge image","alt":""}" data-wp-interactive="core/image">![](https://thomasklijnman.github.io/_site/assets/img/12/AccessTokenExample.png?resize=640%2C259)<button aria-haspopup="dialog" aria-label="Enlarge image" class="lightbox-trigger" data-wp-init="callbacks.initTriggerButton" data-wp-on--click="actions.showLightbox" data-wp-style--right="context.imageButtonRight" data-wp-style--top="context.imageButtonTop" type="button"> <svg fill="none" height="12" viewbox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg"> <path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff"></path> </svg> </button></figure>In the below example I then use that same access token to retrieve all groups in my lab tenant with the below code.

<figure class="wp-block-image size-large wp-lightbox-container" data-wp-context="{"uploadedSrc":"http:\/\/codeartisanjourney.com\/wp-content\/uploads\/2023\/12\/FunctionRunExample.png","figureClassNames":"wp-block-image size-large","figureStyles":null,"imgClassNames":"wp-image-125","imgStyles":null,"targetWidth":1153,"targetHeight":841,"scaleAttr":false,"ariaLabel":"Enlarge image","alt":""}" data-wp-interactive="core/image">![](https://thomasklijnman.github.io/_site/assets/img/12/FunctionRunExample.png?resize=640%2C467)<button aria-haspopup="dialog" aria-label="Enlarge image" class="lightbox-trigger" data-wp-init="callbacks.initTriggerButton" data-wp-on--click="actions.showLightbox" data-wp-style--right="context.imageButtonRight" data-wp-style--top="context.imageButtonTop" type="button"> <svg fill="none" height="12" viewbox="0 0 12 12" width="12" xmlns="http://www.w3.org/2000/svg"> <path d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" fill="#fff"></path> </svg> </button></figure>## Moving Beyond PowerShell Modules

While Powershell modules are good tools, breaking away from their dependency has allowed me to explore more of the Microsoft Graph API possibilities. This shift from Powershell modules has also expanded my personal perspective and allows me to get more potential out of the Microsoft API services without relying solely on predefined modules and cmdlets.

More in depth Graph API posts will be posted soon.