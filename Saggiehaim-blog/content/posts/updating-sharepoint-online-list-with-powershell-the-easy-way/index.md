+++
author = "Saggie Haim"
title = 'Updating Sharepoint Online List With Powershell the Easy Way'
date = 2018-12-24
draft = false
description = "In this post, I share my journey of automating a weekly update of a SharePoint Online list using PowerShell. Faced with the challenge of importing an HR CSV report, I discovered the PnP PowerShell module, which greatly simplified the process. I cover how to connect to SharePoint, import data, empty the list, and update it, while also addressing the challenge of matching username formats. This guide aims to make your first experience with SharePoint Online and PowerShell automation smoother and more efficient."
toc = true
tags = [
    "SharePoint",
    "PowerShell",
    "Automation",
]
categories = [
    "SharePoint",
    "PowerShell",
]
keywords = [
    "PowerShell",
    "SharePoint Online",
    "PnP PowerShell module",
    "SharePoint automation",
    "CSV import",
    "IT administration",
    "SharePoint list update",
    "PowerShell script",
    "Automating SharePoint tasks",
    "SharePoint integration",
]
image = "/images/SharePointOnlineCover.png"
+++

Hey Guys!
It’s been a long time since my last post, as I been busy with work trips.
But now I’m back! And I want to share with you my experience with PowerShell and SharePoint Online.
To be honest, I never worked with SharePoint before.
I got a request from the SharePoint team to update a list weekly from a CSV report that HR provides (Weekly employees birthdays, nothing exciting).

![ ](../images/2porh9.jpg  "You have no power here meme")

First thing to do? Jump head-on to google and understand what my tools are.
I found this task to be really easy, but it took me a while to find what I needed.
Every post I sew online talked about the SharePoint Online module, and some complex queries and scripts...
But I share the greatest skill of IT Admin, I’m lazy...
and it seems hell lot of work for such a simple task...
So, I move on and keep searching.

After reading Microsoft Docs I notice something called PnP, Jackpot!
PnP stands for Patterns and Practices, and it contains a library of PowerShell commands that allows you to perform complex provisioning and artifact management actions towards SharePoint.

So, I first looked for my “lists” cmdlets and found everything I need to complete my tasks.
I won’t write much about installing and using this module as Microsoft Docs provides a great explanation, you can visit the page [here](https://docs.microsoft.com/en-us/powershell/sharepoint/sharepoint-pnp/sharepoint-pnp-cmdlets?view=sharepoint-ps).

So, to further explain my task, I wrote my steps:

1. Connect to the SharePoint Online
2. Import the CSV
3. Empty the list
4. Update the list from the CSV

## Connect to the SharePoint Online

To connect your SharePoint online using the PnP is really easy:

```PowerShell
$userCredential = Get-Credential -Message "Type the password."
$WebUrl = "https://SaggieHaim.sharepoint.com/sites/Portal"
Connect-PnPOnline –Url $WebUrl –Credentials $userCredential
```

## Import the CSV

After we connected to the SharePoint, it’s pretty easy to import the CSV:

```PowerShell
$employeesBirthday = Import-Csv -Path "D:\Tasks\Update Sharepoint Events list\WeeklyBirthdays.CSV"
```

## Empty the list

Now to empty the list, we can simply use the Get-PnPListItem and Remove-PnPListItem:

```PowerShell
Get-PnPListItem -List "$listName" | foreach { Remove-PnPListItem -List "$listName" -Identity $_.Id -Force}
```

## Update the list from the CSV

So, we passed the halfway mark, but this step gave me a headache!
For some reason I kept getting this message when I tried to update to list:

> add-pnplistItem : The specified user could not be found”

And I’m like, WHAT?? it's a string! not a username!

I decided to try one manually just to understand the fields in the list, and then I notice, that EventAuthor field request a username!
The report I get from the old HR system contains the full name, and luckily for me, it’s enough.
But, unlucky for me… I get Last Name First Name, and SharePoint wants First Name Last Name 🤦‍♀️ so off we go:

We can use the add-PnPListItem cmdlet for each employee in the list:

```PowerShell
    foreach ($employee in $employeesBirthday) 
            {
            ## Replacing first and last names
            $EventAuthor = $employee.name.Split(" ")[1] + " " + $employee.name.Split(" ")[0]
            $expire = $person.Expires
                Add-PnPListItem -List "LumenisGreetingsList" -Values @{
                                "Title" = "IMF" ; 
                                "EventDay" = "$employee.EventDay"; 
                                "EventMonth" = "$employee.EventMonth"; 
                                "EventType" = "birthday";
                                "Role" = "$employee.Role"; 
                                "EventAuthor" = $EventAuthor; 
                                "Expires" = "$employee.expire"
                                }
            }
```

Now that everything is working, tested and ready I can build my first SharePoint and PowerShell automation script 😁
I hope my hard learning will make it easier for your “first time” with SharePoint Online!
