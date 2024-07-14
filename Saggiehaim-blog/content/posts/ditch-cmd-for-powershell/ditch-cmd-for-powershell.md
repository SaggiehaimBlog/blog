+++
author = "Saggie Haim"
title = 'Ditch Cmd for Powershell'
date = 2018-09-04
draft = false
description = "In this blog post, discover why you should replace CMD with PowerShell. Learn about the advanced features, enhanced scripting capabilities, and superior functionality that PowerShell offers over the traditional Command Prompt, making it the preferred choice for modern IT professionals."
toc = true
tags = [
    "PowerShell",
    "CMD",
]
categories = [
    "PowerShell",
]
keywords = [
    "Ditch CMD for PowerShell",
    "PowerShell benefits",
    "Command Prompt vs PowerShell",
    "Advanced PowerShell features",
    "Replace CMD with PowerShell",
    "PowerShell scripting advantages",
    "IT automation with PowerShell",
    "PowerShell command example",
    "Why use PowerShell",
    "PowerShell for IT professionals",
]
image = "/images/pstocmd-cover.png"
+++

When I started to learn PowerShell, I said to myself “I’m going to do everything in PowerShell”.
In the beginning, the simpler stuff like unlocking a locked-out account, or resetting a password, took me 5 minutes.
I had to google how to do it on PowerShell, understand what I’m reading and testing it before I’m deleting all organization users.

With the practice and learning, things become much better and most important, faster.
I sometimes even “race” with my friends to show them how things much simpler and faster with PowerShell.
Yet, I always found myself going back to CMD for the most basic things. while you can run native CMD commands in PowerShell, you can’t really leverage the info you get.

I made a list of the CMD commands I use the most:

* ping
* telnet
* tracert
* ipconfig
* nslookup
* ipconfig /flushdns

Luckily for me, The PowerShell team did a great job, and I needed to learn only one cmdlet for the first three command
Let me introduce you Test-NetConnection.
This cmdlet will let us test ports connectivity, ping and traceroute.

## Replacing Ping

The most straightforward use of this command is just Test-NetConnection.
It will send a ping to a default server, see if you have an internet connection:

```PowerShell
PS C:\> Test-NetConnection
 
ComputerName : internetbeacon.msedge.net
RemoteAddress : 13.107.4.52
InterfaceAlias : Wi-Fi
SourceAddress : 10.91.8.20
PingSucceeded : True
PingReplyDetails (RTT) : 5 ms
```

To test a specific computer or address you can use the **-RemoteAddress** or **-ComputerName** parameter:

```PowerShell
PS C:\> Test-NetConnection -RemoteAddress www.saggiehaim.net
```

## Replacing TelNet

To perform port connectivity with Test-NetConnection, all we need to do is add the **-port** parameter and port number: 

```PowerShell
PS C:\> Test-NetConnection -RemoteAddress www.saggiehaim.net -Port 80
 
ComputerName : www.saggiehaim.net
RemoteAddress : 104.31.92.245
RemotePort : 80
InterfaceAlias : Wi-Fi
SourceAddress : 10.91.8.20
TcpTestSucceeded : True
```

> we are looking for is the “TcpTestSucceeded,” in this case, the port is open.

## Replacing Tracert

Same as the previous, same command, different parameter.
This time we will use the **-TraceRoute** parameter

```PowerShell
PS C:\> Test-NetConnection -TraceRoute www.saggiehaim.net
 
ComputerName : www.saggiehaim.net
RemoteAddress : 104.31.93.245
InterfaceAlias : Wi-Fi
SourceAddress : 192.168.1.106
PingSucceeded : True
PingReplyDetails (RTT) : 12 ms
TraceRoute : 192.168.1.1
212.179.37.1
10.250.1.14
212.25.77.10
62.219.189.186
62.219.33.146
104.31.93.245
```

## Replacing IPconfig

To get the IP address, subnet mask and default gateway for each network adapter in the machine, we can use one of the following two cmdlets:

```PowerShell
PS C:\> Get-NetIPConfiguration
 
InterfaceAlias : Local Area Connection
InterfaceIndex : 18
InterfaceDescription : Realtek PCIe GBE Family Controller
NetProfile.Name : Network 6
IPv4Address : 192.168.1.101
IPv6DefaultGateway : fe80::b88f:6eff:fe80:7e6e
IPv4DefaultGateway : 192.168.1.1
DNSServer : 192.168.1.1
0.0.0.0
```

```PowerShell
PS C:\> Get-NetIPAddress
 
IPAddress : fe80::1de1:5d1f:2818:48cc%18
InterfaceIndex : 18
InterfaceAlias : Local Area Connection
AddressFamily : IPv6
Type : Unicast
PrefixLength : 64
PrefixOrigin : WellKnown
SuffixOrigin : Link
AddressState : Preferred
ValidLifetime : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource : False
PolicyStore : ActiveStore
 
IPAddress : 192.168.1.101
InterfaceIndex : 18
InterfaceAlias : Local Area Connection
AddressFamily : IPv4
Type : Unicast
PrefixLength : 24
PrefixOrigin : Dhcp
SuffixOrigin : Dhcp
AddressState : Preferred
ValidLifetime : 17:04:15
PreferredLifetime : 17:04:15
SkipAsSource : False
PolicyStore : ActiveStore
```

## Replacing NSlookup

Many of us IT pro’s, use the nslookup a lot.
So, we can do it on PowerShell with the Resolve-DnsName cmdlet:

```PowerShell
PS C:\> Resolve-DnsName www.saggiehaim.net
 
Name Type TTL Section IPAddress
---- ---- --- ------- ---------
www.saggiehaim.net AAAA 300 Answer 2400:cb00:2048:1::681f:5df5
www.saggiehaim.net AAAA 300 Answer 2400:cb00:2048:1::681f:5cf5
www.saggiehaim.net A 300 Answer 104.31.93.245
www.saggiehaim.net A 300 Answer 104.31.92.245
```

you want to query a specific DNS server, we can use the **-Server** parameter to do it:

```PowerShell
PS C:\> Resolve-DnsName www.saggiehaim.net -Server 1.1.1.1
```

## Ipconfig /flushdns

Before we delete all the DNS client cache, we can view the cache list to see if it’s needed

```PowerShell
PS C:\> Get-DnsClientCache
```

If the list is big, we can look for the particular hostname with the **-Name** or **-Entry** parameters or particular IP address with the **-Data** Parameter

```PowerShell
PS C:\> Get-DnsClientCache -Entry www.saggiehaim.net
 
Entry RecordName Record Status Section TimeTo Data Data
Type Live Length
----- ---------- ------ ------ ------- ------ ------ ----
www.saggiehaim.net www.saggiehaim.net A Success Answer 272 4 104.31.93.245
www.saggiehaim.net www.saggiehaim.net A Success Answer 272 4 104.31.92.245
www.saggiehaim.net www.saggiehaim.net A Success Answer 272 4 104.31.93.245
www.saggiehaim.net www.saggiehaim.net A Success Answer 272 4 104.31.92.245
```

```PowerShell
PS C:\> Get-DnsClientCache -Data 104.31.92.245
 
Entry RecordName Record Status Section TimeTo Data Data
Type Live Length
----- ---------- ------ ------ ------- ------ ------ ----
www.saggiehaim.net www.saggiehaim.net A Success Answer 33 4 104.31.92.245
www.saggiehaim.net www.saggiehaim.net A Success Answer 33 4 104.31.92.245
www.saggiehaim.net www.saggiehaim.net A Success Answer 33 4 104.31.92.245
[/code]
 
To clear the client DNS cache, we can run the Clear-DnsClientCache.
 
[code lang="Powershell"]
Clear-DnsClientCache
```

## Why PowerShell Cmdlets?

So yes, we can do it with PowerShell cmdlets, but if we can also run the native CMD commands inside PowerShell.
What is the difference? The truth is that there are no differences if you run it once, manually.
The real benefits of using the PowerShell cmdlets are when you run them in scripts or doing automation tasks.
When you use the CMD commands, your output is plain text a String.
This means that if you want to take specific info from the command, you will have to do a lot of strings manipulations that won’t be useful for 100% of the cases.

![System.String objects are limited](../images/ipconfig.jpg  "Image PowerShell terminal")

Using the PowerShell cmdlets will return PowerShell Objects, Objects that contain all the information in properties we can query and pass to other cmdlets or functions, or even other scripts.

## The Power of PowerShell Objects

Using the PowerShell cmdlet, we can extract or use specific property.
Not all properties are showing by default and its worth the time exploring them.
we can pipeline the command to get-command cmdlet to see the object information.

![PowerShell objects are the future](../images/psobject.png  "Image PowerShell terminal")

We can choose the properties that interesting to us, for example, I want to see only the interface that performed the ping, and if it’s succeeded or not:

```PowerShell
PS C:\> Test-NetConnection -ComputerName www.saggiehaim.net | select InterfaceAlias,PingSucceeded
 
InterfaceAlias PingSucceeded
-------------- -------------
Wi-Fi True
```

Simple as that 🙂 Go out there, give it a try, and remember, practice, practice, practice.
It’s tough to replace old habits, but it worth the work.

Hope I helped you take a step closer ditching the CMD for good.
