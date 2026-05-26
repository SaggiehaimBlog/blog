+++
author = "Saggie Haim"
title = 'How to Connect to Azure With Powershell'
date = 2020-04-02
image = "/images/howtoconnecttoazure.jpeg"
draft = false
description = "PowerShell is the Automator's best friend. I automate everything with PowerShell, and you can also. When it comes to Azure, I can't remember the last time I created a resource in the Portal. I either use PowerShell or IaC to provision resources or modify them."
toc = false
tags = [ 
    "PowerShell",
    "Azure",
    "Automation",
]
categories = [
    "PowerShell",
    "Azure",
]
keywords = [
    "Connect to Azure PowerShell",
    "Azure PowerShell",
    "Connect-AzAccount cmdlet",
    "Azure login PowerShell",
    "Azure authentication PowerShell",
    "PowerShell Azure setup",
    "Azure management with PowerShell",
    "Azure Resource Manager",
    "Interactive login Azure PowerShell",
    "PowerShell cloud automation",
    "Sign in to Azure",
    "PowerShell Azure tutorial",
    "Azure subscription management",
    "Azure PowerShell commands",
    "Secure login Azure PowerShell",
]
+++

PowerShell is the Automator's best friend.
I automate everything with PowerShell, and you can also.
When it comes to Azure, I can't remember the last time I created a resource in the Portal.
I either use PowerShell or IaC to provision resources or modify them.

In this post, you will learn how to set up a session with Azure to start to interact with Azure Resources.

## Prerequisites

Before we start, there are few global prerequisites that you need to meet:

+ Active Azure Subscription, if you don't have one, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
+ Reader permissions to the subscription.
+ PowerShell V7, if you don't have it installed, install it from the [GitHub Repository.](https://github.com/PowerShell/PowerShell)

## Getting to know the tools

### PowerShell V7

PowerShell V7 is a cross-platform task automation and configuration management framework, consisting of a command-line shell and scripting language.
Make sure you installed it on your system.

### Az Module

Azure PowerShell Az module is a PowerShell module for interacting with Azure.
Az offers shorter commands, improved stability, and cross-platform support.

To install it, you run the following command:

```PowerShell
Install-Module az -AllowClobber -Scope CurrentUser
```

### Splatting

In most of the code examples, I use "splatting" to pass the parameters.
Splatting makes your commands shorter and easier to read.
You can read more about it [here.](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-7)

## Connect to Azure with PowerShell

You also need to set up a session to Azure from PowerShell, and you can create one with the Az module.

You need to get your Tenant ID and Subscription ID from the Azure Portal.

With this information, you can use the `Connect-AzAccount` to create a session with Azure:

```PowerShell
$TenantID = 'XXXX-XXXX-XXXX-XXXX-XXXX'
$SubscriptionID = 'XXXX-XXXX-XXXX-XXXX'
Connect-AzAccount -TenantId $TenantID -SubscriptionId $SubscriptionID
```

You can now interact with Azure from PowerShell and start your journey to automate everything.
