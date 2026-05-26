+++
author = "Saggie Haim"
title = 'Automating The Network Redesign Process'
date = 2018-08-17
draft = false
description = "In this blog post, learn how to automate network redesign using PowerShell and other tools. The article guides you through the process of automating network configurations, deployments, and management tasks, enhancing efficiency and reducing manual errors in network operations."
toc = true
tags = [
    "PowerShell",
    "Network Redesign",
    "Automation",
]
categories = [
    "PowerShell",
]
keywords = [
    "Automate network redesign",
    "Network automation tools",
    "PowerShell network management",
    "IT network redesign",
    "Automated network deployment",
    "Network management with PowerShell",
    "Efficient in network operations",
    "Reduce manual network errors",
    "IT infrastructure automation",
]
image = "/images/NewSite-977x392.png"
+++

Namaste!
I’m sitting in my hotel room, it’s very late, hoping not to get sick.
It’s my first day in India, as my company has an office here in New Delhi.
If you wonder why I hope not to get sick, well, this was my dinner today:

![Would you eat it?](./images/IndiaDinner.jpeg  "Image of my indian dinner")

To be honest, it was very nice, not the best I had, but far from my worst one.
If you didn’t notice, it had chicken inside.

Part of my job is to take care of the network and NAC.
So, I’m here to replace the switches, set up new access points and implement the NAC solution.
Each site has its own class B LAN network, which is not so good, not for security nor administration.
So, I need to re-design the network.

To do so, I need to apply a new configuration for the switches, the firewall, the Domain Services and the servers (Client will get new configuration from DHCP).
Now, if our LAN was simple, with VLANs for Users and Servers, it was easy.
However, due to the security requirement and the network segregation, we have 17 VLANs to create.

To do it once its ok, to do it in our 15 sites, it's a pain! So, I said to myself as a true lazy guy: “Saggie, there is no chance you are going to do it again.”
So, I decided to automate the process.
First, we need to plan, so I’m sitting down to write the tasks I have to make:

1. Create DHCP Scopes
2. Set DHCP Server options
3. Update Site and services
4. Create DNS reverse lookup zone

Then, I needed to write down the challenges in front of me:

1. Each site has its own DC and DHCP Servers
2. Each site has its own LAN network

Now, I could start writing my script.
So, I divided the script to three sections:

## Gathering Information

It necessary to know which DHCP server I want to modify.
I don’t want to add the scope to the wrong server, it also necessary to know what the Domain Controllers and DNS servers are, the Site name and location and most important, because that why we are here the network segment.
I’m using the Read-host command to populate the variables easily.

```PowerShell
$DCServer = Read-Host -Prompt "Enter The DC Server FQDN or IP"
$DHCPServer = Read-Host -Prompt "Enter The DHCP Server FQDN or IP"
$NetSeg = Read-Host -Prompt "Enter the Network Sufix (if the network is 10.102.0.1, enter 102)"
$DNSServer01 = Read-host -Prompt "Enter Primary DNS Server"
$DnsServer02 = Read-Host -Prompt "Enter Secondary DNS Server"
$SiteName = Read-Host -Prompt "Enter Site And Services site name"
```

## Creating the Scopes

Now, that I have all the information I need for the particular site, I can start building the ‘static’ scopes.
I’m building an array for each scope, so it will be easy to use it later in the script:

```PowerShell
$FirstScope = [Ordered]@{
    Name = "Network-1";
    ScopeId = "10."+$NetSeg+".1.0";
    StartRange = "10."+$NetSeg+".1.1";
    EndRange = "10."+$netSeg+".1.254";
    SubNetMask = "255.255.255.0";
    Description = "Network-2";
    LeaseDuration = "3.00:00:00";
    defaultGateway = "10."+$NetSeg+".1.254"
}
$SecondScope = [Ordered]@{
    Name = "Network-2";
    ScopeId = "10."+$NetSeg+".2.0";
    StartRange = "10."+$NetSeg+".2.1"
    EndRange = "10."+$netSeg+".2.254"
    SubNetMask = "255.255.255.0"
    Description = "Network-2"
    LeaseDuration = "3.00:00:00"
    defaultGateway = "10."+$NetSeg+".2.254"
}
```

## Creating the Scope, Site and Services Subnets and DNS reverse lookup zone

At this point, we have all the information we need.
We can start creating the DHCP scope, update the scope options, set DHCP server options, create DNS reverse lookup zone, and create the subnets for AD Site and Services.
To do that, I’m using the following cmdlets:

* [Add-DhcpServerv4Scope](https://docs.microsoft.com/en-us/powershell/module/dhcpserver/add-dhcpserverv4scope?view=win10-ps) – To create the DHCP scopes
* [Set-DhcpServerv4OptionValue](https://docs.microsoft.com/en-us/powershell/module/dhcpserver/set-dhcpserverv4optionvalue?view=win10-ps) – To add Scope and server options
* [New-ADReplicationSubnet](https://docs.microsoft.com/en-us/powershell/module/addsadministration/new-adreplicationsubnet?view=win10-ps) – To create new subnets in Site And Services
* [Add-DnsServerPrimaryZone](https://docs.microsoft.com/en-us/powershell/module/dnsserver/add-dnsserverprimaryzone?view=win10-ps) – To Create new DNS reverse lookup zone

```PowerShell
ForEach-Object {
    ## creating the Scopes
    Add-DhcpServerV4Scope -Name $_.Name -Description $_.Description -LeaseDuration $_.LeaseDuration -StartRange $_.StartRange -EndRange $_.EndRange -SubnetMask $_.SubNetMask -ComputerName $DHCPServer;
    ## Setting DefaultGateway for each scope
    Set-DhcpServerv4OptionValue -ComputerName $DHCPServer -ScopeId $_.ScopeId -OptionId 03 -Value $_.defaultGateway;
    ## create new subnets in Site And Services
    $name = $_.ScopeID + "/24";
    New-ADReplicationSubnet -Server $DCServer -Name $name -Site $SiteName -Location $SiteName
    ## Create new DNS reverse lookup zone
    Add-DnsServerPrimaryZone -DynamicUpdate Secure -NetworkId $name -ReplicationScope Forest -ComputerName $DCServer
}
 
## Setting DHCP Server Options
Set-DhcpServerV4OptionValue -DnsDomain "Saggiehaim.net" -DnsServer $DnsServer01,$DnsServer02
```

Now it takes less than 3 seconds to redesign the network entirely.
Doing some cleaning and working with the firewall team to finish everything.

It’s important when you write a script, to know who is going to use it.
In this case, I’m the only one who is going to use it, so I’m not entering any validations to my script.
If you are creating the script for someone else, it’s important to add some safe switches to make sure everything running correctly.

I recommend learning the cmdlet, use the links to Microsoft docs, and understand what you are doing.
Don’t run anything from the post without testing it before!
Use it on your own risk.

Hope you enjoyed reading.
