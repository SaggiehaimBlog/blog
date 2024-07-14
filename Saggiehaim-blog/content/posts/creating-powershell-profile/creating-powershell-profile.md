+++
author = "Saggie Haim"
title = 'Creating Powershell Profile'
date = 2018-08-24
draft = false
description = "In this blog post, learn how to create a PowerShell profile to customize your scripting environment. The article covers step-by-step instructions for setting up a profile, adding custom functions, aliases, and scripts to enhance your productivity and streamline your PowerShell experience."
toc = true
tags = [
    "PowerShell",
    "Automation",
]
categories = [
    "PowerShell",
]
keywords = [
    "Create PowerShell profile",
    "Customize PowerShell environment",
    "PowerShell profile setup",
    "PowerShell scripting tips",
    "PowerShell automation",
    "PowerShell custom functions",
    "PowerShell aliases",
    "PowerShell configuration",
    "Enhance PowerShell productivity",
    "IT scripting with PowerShell"
]
image = "/images/create-Ps-Profile-Cover.png"
+++

I had a conversation with a friend today, talked about work, he told me that he has some scripts that he uses a lot in his day-to-day job, and thought about creating a script that loads all of them at once.
The first thing I told him is “Why don’t you set up a profile?” he waited for a few seconds and told me that he doesn’t know about profiles.
If you are reading this and feel the same, this post is for you 🙂
PowerShell profile is a script that runs every time we start the shell.
They can have customization elements like windows size and colors, load modules functions, and scripts, or create an alias or any PS-providers.
Creating a profile is much easier as you think.
PowerShell has a global variable name $Profile that hold the profile file path.
By default, the file does not exist even if the variable holds it.
You can type $profile to see the location:

```PowerShell
$PROFILE
C:\Users\MyUser\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
```

Almost all PowerShell editors have their own profiles.
**VS Code:** C:\Users\MyUser\Documents\WindowsPowerShell\Microsoft.VSCode_profile.ps1
**PowerShell ISE:** C:\Users\MyUser\Documents\WindowsPowerShell\Microsoft.PowerShellISE_profile.ps1

## Creating PowerShell Profile

I say “profiles” as we have four profiles for each shell, but we won’t dive there.
To create our profile, first we need to check if we have one already:

```PowerShell
Test-Path $PROFILE
False
```

We can see we don’t have one, so we create one with new-item cmdlet:

```PowerShell
New-Item -Type File -Path $PROFILE -Force
 
Directory: C:\Users\administrator\Documents\WindowsPowerShell
 
Mode LastWriteTime Length Name
---- ------------- ------ ----
-a---- 8/24/2018 11:36 AM 0 Microsoft.PowerShell_profile.ps1
```

Now you can verify the creation of the file with the same test-path we used earlier.
if everything went ok, we get “True”.

```PowerShell
Test-Path $PROFILE
True
```

After we created our profile file, all left for us is to edit it, with any app we want, you can quickly open the file with your favorite editor:

```PowerShell
## Open with Powershell ISE
ISE $Profile
## Open with VS Code
code $Profile
## open with notepad
notepad $Profile
```

## Here Some Ideas to Add to Your Profile

1. Change the prompt, and add `[Admin]` if its elevated session:

```PowerShell
function prompt {
    $identity = [Security.Principal.WindowsIdentity]::GetCurrent()
    $principal = [Security.Principal.WindowsPrincipal] $identity
  
    $(if (test-path variable:/PSDebugContext) { '[DBG]: ' }
      elseif($principal.IsInRole([Security.Principal.WindowsBuiltInRole]"Administrator")) { "[ADMIN]: " }
      else { '' }
    ) + 'Saggie Haim PS ' + $(Get-Location) +
      $(if ($nestedpromptlevel -ge 1) { '>>' }) + '> '
  }
```

2. Change the color of the window and font:

```PowerShell
## Background Color
$host.UI.RawUI.BackgroundColor = "black"
## Font color
$host.UI.RawUI.ForegroundColor = "red"
```

3. Add your name to the windows title:

```PowerShell
## Keep the default title and add my name
$host.ui.RawUI.WindowTitle += " Saggie Haim"
```

4. Load modules and add custom functions:

```PowerShell
## Test-Port function
function Test-Port {
    param (
    [Parameter(Mandatory=$true)]$hostname,
    [Parameter(Mandatory=$true)]$port
    )
    $result =(New-Object System.Net.Sockets.TcpClient($hostname, $port)).Connected
    return $result
}
## Import Modules
Import-Module ActiveDirectory
```

There is no limit to what you can add to your profile.
You can always add or remove new things from your profile.
Use it as your toolbox, add some style and make it your own.
If you want to read more about PowerShell profile, Microsoft did a great job with its Documentation.
You can use get-help about_profiles or read the online version [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-6).
