+++
author = "Saggie Haim"
title = 'Sorting Group Policy'
date = 2019-01-10
draft = false
description = "Group Policy. It’s almost the same in every company. it’s historical, contain a lot of Group Policy and if you ever try to touch it, half of the systems stop working."
toc = true
tags = [
    "Group Policy",
    "PowerShell",
    "Automation"
]
categories = [
    "Group Policy",
    "PowerShell",
]
keywords = [
    "Sort GPO",
    "Group Policy",
    "PowerShell",
    "Automation",
    "GPO issue",
    "Sort Group Policy",
    "Organize GPO",
    "Organize Group Policy",
]
image = "/images/GPOFeaturedImage.png"
+++

So, this is the first post for 2019! Happy new year!Today I want to talk about a painful topic, the Group Policy.
It’s almost the same in every company. it’s historical, contain a lot of Group Policy and if you ever try to touch it, half of the systems stop working.

![ ](../images/GPOFeatureImage.jpg  "Its ok burning meme")

Last week I saw a post from my good friend Omer a Microsoft PFE, as he talked about [Common Mistakes in Active Directory and Domain Services](https://blogs.technet.microsoft.com/meamcs/2018/12/31/most-common-mistakes-in-active-directory-and-domain-services-part-1/).
He points out that Removing “Authenticated Users” from the GPO Object Security Filtering is a bad thing.
he also provided a PowerShell script to find GPO’s without “Authenticated Users” permissions.

After running his GPO script, I saw that there few GPO’s that needed to sort.
It gave me the motivation to analyze my entire environment, as a result, I created a few scripts.

## The Target

Firstly, before I start, I’m writing down what I need to look for:

- Disabled GPO’s
- Empty GPO’s
- Unlinked GPO’s
- Missing permissions GPO’s (Based on Omer Script)

I wrote all tests together in one script, but later decided to divide them to functions, so I can run each test individually and in the future create a complete Group Policy Report utilizing those functions.
Therefore let’s deep dive on each test.

## Disabled GPO’s

In this test, we will look for GPO’s with User, Computer or both setting disabled.

### the issue

Technically, there is no issue with GPO with disabled settings.
But if both of them is disabled, then its unused GPO.
We probably don’t need them. If only User or Computer disabled, then it's a “red flag” and we need to make sure we are not “losing” configuration.
Maybe we can merge the settings with another GPO?

### The Function

This will check if there are any settings disabled and return the GPO's.

```PowerShell
function Get-GPODisabled {
    [cmdletbinding()]
    param (
    )
    try {
        Write-Verbose -Message "Importing GroupPolicy module"
        Import-Module GroupPolicy -ErrorAction Stop
    }
    catch {
        Write-Error -Message "GroupPolicy Module not found. Make sure RSAT (Remote Server Admin Tools) is installed"
        exit
    }
    $DisabledGPO = New-Object System.Collections.ArrayList
    try {
        Write-Verbose -Message "Importing GroupPolicy Policies"  
        $GPOs = Get-GPO -All  
        Write-Verbose -Message "Found '$($GPOs.Count)' policies to check"
    }
    catch {
        Write-Error -Message "Can't Load GPO's Please make sure you have connection to the Domain Controllers"
        exit
    }
    ForEach ($gpo  in $GPOs) { 
        Write-Verbose -Message "Checking '$($gpo.DisplayName)' status"
        switch ($gpo.GpoStatus) {
            "ComputerSettingsDisabled" {$DisabledGPO += "in '$($gpo.DisplayName)' the Computer Settings Disabled"}
            "UserSettingsDisabled" {$DisabledGPO += "in '$($gpo.DisplayName)' the User Settings Disabled"}
            "AllSettingsDisabled" {$DisabledGPO += "in '$($gpo.DisplayName)' All Settings Disabled"}
        }
    }
    if (($DisabledGPO).Count -ne 0) {
        Write-Host "The Following GPO's have settings disabled:"
        return $DisabledGPO
    }
    else {
        return "No GPO's with setting disabled Found"
    }
        
}
```

![Result of the Get-GPODisabled](../images/image-1.png  "PowerShell Terminal with the result of running the Get-GPODisabled")

## Empty GPO’s

In this test, we will look for empty GPO’s, that means that the GPO has no setting configured on it.

### The issue

Empty GPO’s are useless and just adding more overhead.
We don’t need them in our environment.

### The Function

This function will check if there is GPO with the User or Computers settings empty.
one common mistake people do, is they check the “User version” and “Computer version”, while it true that a new GPO start with version 0 on both of them and changed in each modification.
but if you take an old GPO and remove all settings, the version won’t reset to 0, and you will miss those GPO’s!

![Empty GPO with computer version 4.](../images/GPO.png  "Example of GPO settings")

```PowerShell
function Get-GPOEmpty {
    [cmdletbinding()]
    param (
    )
    try {
        Write-Verbose -Message "Importing GroupPolicy module"
        Import-Module GroupPolicy -ErrorAction Stop
    }
    catch {
        Write-Error -Message "GroupPolicy Module not found. Make sure RSAT (Remote Server Admin Tools) is installed"
        exit
    }
    $EmptyGPO = New-Object System.Collections.ArrayList
    try {
        Write-Verbose -Message "Importing GroupPolicy Policies"  
        $GPOs = Get-GPO -All  
        Write-Verbose -Message "Found '$($GPOs.Count)' policies to check"
    }
    catch {
        Write-Error -Message "Can't Load GPO's Please make sure you have connection to the Domain Controllers"
        exit
    }
    ForEach ($gpo  in $GPOs) { 
        Write-Verbose -Message "Checking '$($gpo.DisplayName)' status"
        &#91;xml]$GPOXMLReport = $gpo | Get-GPOReport -ReportType xml
        if ($null -eq $GPOXMLReport.gpo.User.ExtensionData -and $null -eq $GPOXMLReport.gpo.Computer.ExtensionData) {
            $EmptyGPO += $gpo
        }
    }
    if (($EmptyGPO).Count -ne 0) {
        Write-Host "The Following GPO's are empty:"
        return $EmptyGPO.DisplayName
    }
    else {
        return "No Empty GPO's Found"
    }
}
```

![Result of the Get-GPOEmpty](../images/image-2.png  "PowerShell Terminal with the result of running the Get-GPOEmpty")

## Unlinked GPO’s

In this test, we will have the most results, as it's the most common case.
Unlinked GPO’s are GPO’s that not linked to any domain, site or OU.

### The issue

Unlinked GPO’s are not an “issue” but most of the time, they are useless overhead to manage and track.
In my company, there was GPO’s from 2008. I guess we won’t use it anymore 😒

### The function

```PowerShell
function Get-GPOUnlinked {
    [cmdletbinding()]
    param ()
    try {
        Write-Verbose -Message "Importing GroupPolicy module"
        Import-Module GroupPolicy -ErrorAction Stop
    }
    catch {
        Write-Error -Message "GroupPolicy Module not found. Make sure RSAT (Remote Server Admin Tools) is installed"
        exit
    }
    $UnlinkedGPO = New-Object System.Collections.ArrayList
    try {
        Write-Verbose -Message "Importing GroupPolicy Policies"  
        $GPOs = Get-GPO -All  
        Write-Verbose -Message "Found '$($GPOs.Count)' policies to check"
    }
    catch {
        Write-Error -Message "Can't Load GPO's Please make sure you have connection to the Domain Controllers"
        exit
    }
    ForEach ($gpo  in $GPOs) { 
        Write-Verbose -Message "Checking '$($gpo.DisplayName)' link"
        &#91;xml]$GPOXMLReport = $gpo | Get-GPOReport -ReportType xml
        if ($null -eq $GPOXMLReport.GPO.LinksTo) { 
            $UnlinkedGPO += $gpo
        }
    }
    if (($UnlinkedGPO).Count -ne 0) {
        return $UnlinkedGPO.DisplayName
    }
    else {
        return Write-Host "No Unlinked GPO found"
    }
}
```

![Result of the Get-GPOUnlinked](../images/image-3.png  "PowerShell Terminal with the result of running the Get-GPOUnlinked")

## Missing Permissions

Here we will test to see if there are GPO’s with missing Read permissions to the “Authenticated Users” or “Domain Computers” groups.

### The Issue

This test is based on Omer post I shared before, so I recommend you give it a read if you haven’t yet.
he explains it better than me 😃
### The Function

```PowerShell
function Get-GPOMissingPermissions {
    [cmdletbinding()]
    param ()
    try {
        Write-Verbose -Message "Importing GroupPolicy module"
        Import-Module GroupPolicy -ErrorAction Stop
    }
    catch {
        Write-Error -Message "GroupPolicy Module not found. Make sure RSAT (Remote Server Admin Tools) is installed"
        exit
    }
    $MissingPermissionsGPOArray = New-Object System.Collections.ArrayList
    try {
        Write-Verbose -Message "Importing GroupPolicy Policies"  
        $GPOs = Get-GPO -All  
        Write-Verbose -Message "Found '$($GPOs.Count)' policies to check"
    }
    catch {
        Write-Error -Message "Can't Load GPO's Please make sure you have connection to the Domain Controllers"
        exit
    }
    ForEach ($gpo  in $GPOs) { 
        Write-Verbose -Message "Checking '$($gpo.DisplayName)' status"
        If ($GPO.User.Enabled) {
            $GPOPermissionForAuthUsers = Get-GPPermission -Guid $GPO.Id -All | Select-Object -ExpandProperty Trustee | Where-Object {$_.Name -eq "Authenticated Users"}
            $GPOPermissionForDomainComputers = Get-GPPermission -Guid $GPO.Id -All | Select-Object -ExpandProperty Trustee | Where-Object {$_.Name -eq "Domain Computers"}
            If (!$GPOPermissionForAuthUsers -and !$GPOPermissionForDomainComputers) {
                $MissingPermissionsGPOArray += $gpo
            }
        }
    }
    if (($MissingPermissionsGPOArray).Count -ne 0) {
        Write-Host "The following GPO's do not grant any permissions to the 'Authenticated Users' Or 'Domain Computers' Group"
        return $MissingPermissionsGPOArray.DisplayName
    }
    else {
        return &#91;string]"No GPO's with missing permissions to the 'Authenticated Users' or 'Domain Computers' groups found "
    }
}
```

![Result of the Get-GPOMissingPermissions](../images/image-4.png  "PowerShell Terminal with the result of running the Get-GPOMissingPermissions")

## Doing it Safe

Before you make any changes, I highly recommend you to backup any GPO before you change or delete.
If you hear people scream and shout outside, just restore the GPO.

### Backup GPO's

While you can easily backup GPO’s over the GUI, we love PowerShell more.
so we will do it with PowerShell.

To backup all GPO’s or specific ones, just run the following cmdlet:

```PowerShell
## Backup All GPO's
Backup-Gpo -All -Path "Location"
 
## Backup specific GPO
Backup-Gpo -Name "GPO NAME" -Path "Location" -Comment "Comment" 
```

### Restore-GPO

If we want to restore deleted or changed GPO, we can use the following cmdlet:

```PowerShell
## Restore All GPO's
Restore-GPO -All -Domain "Domain Name" -Path "Backups Location"
 
## Restore specific GPO
Restore-GPO -Name "GPO NAME" -Path "Backup Location
```

Restore-GPO only support restoring GPO’s of the **same** domain!

## Conclusion

In conclusion, we learned what we need to search.
We saw the functions that will help us find the GPO’s.
And we learned to backup and restore GPO’s.
It’s time to sort the group policy mess! in the future, I hope to release the full GPO environment report, so stay tuned and good luck!

You can find all the code on [GitHub](https://github.com/SaggiehaimBlog/PS-GPO-Tools)