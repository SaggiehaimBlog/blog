+++
author = "Saggie Haim"
title = 'Assessing New Domain Environment'
date = 2018-09-25
draft = false
description = "n this blog post, I share insights on assessing a new domain environment, highlighting key steps and considerations. The article covers preliminary checks, tools to use, and best practices to ensure a smooth evaluation process. Whether you’re setting up a new network or auditing an existing one, this guide provides practical advice to help you effectively manage and secure your domain environment."
toc = true
tags = [
    "PowerShell",
    "Active Directory",
    "IT Security",
    "Domain assessment"
]
categories = [
    "Active Directory",
    "PowerShell",
]
keywords = [
    "Domain environment assessment",
    "IT Domain evaluation",
    "Network setup audit",
    "Active Directory assessment",
    "IT infrastructure review",
    "Network security check",
    "Domain setup audit tools",
    "Active Directory health check",
    "IT environment analysis",
]
image = "/images/assest-a-new-domain.png"
+++

I had a free day yesterday, and I always love to spend my free time on the Xbox or PlayStation, depends on the mood.
So, while I’m rushing and jumping all over Australia with my Lamborghini, I got a phone call, my first instinct was to mute it and continue with my race, but it was a good friend... so I muted it anyway 😶.
When I finished the race, I called him back (yes, I’m a good friend😜).
after a quick chit chat, he told me that he needs help to assess a new domain environment he needs to work on.

![can you blame me for not answering the phone? ](./images/FH3.png  "Image of Forza Horizon game")

He told me that he needs to get some basic information about those topics:

* Forest information
* Domain information
* GPO information
* DHCP authorized server information
* Users information
* Group information

I asked him to prepare a computer in the domain with Remote Server Administration Tools (RSAT) and get a user credentials in the domain (no special privilege) while I’m writing him a guide.

## Forest Information

When we install the RSAT, we get lot of new Powershell cmdlet.
One of them is Get-ADForest cmdlet. Using this command, we can get information about the active directory forest (surprised?)

```PowerShell
$ForestInformation = Get-ADForest
```

Each cmdlet can give us lot of information, and we don’t want to get lost, so in each cmdlet, we will focus on the information I think most relevant, I encourage you to investigate every cmdlet you see here if you don’t familiar with.

We would like to get information about the forest functional level, the domains in the forest, the root domain and schema master dc:

```PowerShell
$ForestInformation | Select-Object ForestMode,Domains,RootDomain,SchemaMaster | Format-List
ForestMode   : Windows2008R2Forest
Domains      : {saggiehaim.net}
RootDomain   : saggiehaim.net
SchemaMaster : SHDC01.saggiehaim.net
```

> The reason we put it into a variable first is because we will use it later.

## Domain Information

In this part we will use two cmdlets, Get-ADDomain and Get-ADDomainController.
We will use those cmdlets to find the functional level of the domains, the PDC and RID master, and information about the domain controllers.

First, we will use the Get-ADDomain, we will use it to get the following information: Name,Child Domains,Domain functional level,Infrastructure Master, PDC Emulator, RID Master.

```PowerShell
# Single Domain Forest
Get-ADDomain | Select-Object Name,DomainMode,InfrastructureMaster,PDCEmulator,RIDMaster | Format-List
 
# Multi Domain Forest
($ForestInformation).Domains | Get-ADDomain | Select-Object Name,ChildDomains,DomainMode,InfrastructureMaster,PDCEmulator,RIDMaster | Format-List
```

Now we will use the Get-ADDomainController to get information about the Domain Controllers in each domain, it will be useful to get information about: Hostname, IPv4Address, Operating System, the LDAP and SSL Ports, and which site it belongs to:

```PowerShell
# Single Domain Forest
Get-ADDomainController -Filter * -Server saggiehaim.net  | Select-Object Hostname,IPv4Address,OperatingSystem,LDAPport,sslport,site | Format-List
 
# Multi Domain Forest
($ForestInformation).Domains | Get-ADDomainController -Filter * -Server $_  | Select-Object Domain,Hostname,IPv4Address,OperatingSystem,LDAPport,SSLport,site | Format-List
```

This will give us a nice list:

```PowerShell
Hostname    : SHDC01.saggiehaim.net
IPv4Address : 10.10.0.1
OperatingSystem : Windows Server 2012 R2 Standard
LDAPport    : 389
sslport     : 636
site        : Lab
```

## GPO Information

Now GPO is super important, and every Domain Services guy will tell you, it can make or break your environment.
When we start working on a new environment, we would like to know how many GPO we got and how many of them are not in use.

To get the GPO’s quantity, we can just run:

```PowerShell
$GPOs = Get-GPO -All
$GPOs.Count
```

To get the unlinked GPO’s we can use the following script:

```PowerShell
import-module GroupPolicy
 
$unlinkedGPOs = @()
$GPOs = Get-GPO -All  
ForEach($gpo  in $GPOs){ 
    [xml]$GPOXMLReport = $gpo | Get-GPOReport -ReportType xml 
    if ($GPOXMLReport.GPO.LinksTo -eq $null){ 
        $unlinkedGPOs += $gpo 
    }  
 }
 ```

 When it's done, we can get the information from the $unlinkedGPOs variable.

 ```PowerShell
 DisplayName      : Test GPO
DomainName       : saggiehaim.net
Owner            : saggiehaim\Domain Admins
Id               : eb9bf117-81c1-40f9-813a-ce211796189d
GpoStatus        : AllSettingsEnabled
Description      : 
CreationTime     : 9/23/2018 12:38:02 PM
ModificationTime : 9/23/2018 12:40:56 PM
UserVersion      : AD Version: 3, SysVol Version: 3
ComputerVersion  : AD Version: 0, SysVol Version: 0
WmiFilter        : 
```

## DHCP Authorized Server Information

DHCP can be a vulnerability, and it's something we want to control, we don’t want rogue DHCP servers that will serve wrong IP’s or worst, won’t be sync with the rest and will serve duplicated IP’s.

To get the list of the authorized DHCP Servers we can use the cmdlet:

```PowerShell
Get-DhcpServerInDC
```

## Users' Information

It’s all about the users, the most critical resource we have in our environment.
There are so many things we want to know about them, but we will limit it to the most important in my opinion:

* Number of users

```PowerShell
$users = Get-ADUser -Filter *
$users.Count
```

* Count Number of active users

```PowerShell
($users | where {$_.Enabled -eq $true}).count
```

* Get Number of locked out users

```PowerShell
(Search-ADAccount -LockedOut).count
```

## Group Information

Groups can often be a pain, and it easy to lose control over them, those cmdlets will help us get some control over them:

* Number of groups

```PowerShell
(Get-ADGroup -Filter *).count
```

* Getting all empty groups

```PowerShell
Get-ADGroup -Properties Members -Filter * | Where-Object {$_.Members.Count -eq 0 } | Select-Object Name,DistinguishedName | Format-List
```

Today security is everywhere, and we all share the responsibility to keep our environment secure, we also like to get all the members of the Domain Admins and Enterprise Admins groups, we will ask questions about every member.

```PowerShell
# Get Domain Admins members
(Get-ADGroup -Properties Members -Identity 'Domain Admins').Members
 
# Get Enterprise Admins members
(Get-ADGroup -Properties Members -Identity 'Enterprise Admins').Members
```

Don’t take it lightly.
I once saw a company with Domain Users in the Domain Admins group...

## Conclusion

One of the most challenging tasks in my job is taking over a domain environment.
It can be very complicated, and it can take so much time just to understand what’s going on.
There is a lot more information to look for, and lot more topics to investigate, but I think I gave you something to start with.

How will you counter this task? I would love to hear about it!
