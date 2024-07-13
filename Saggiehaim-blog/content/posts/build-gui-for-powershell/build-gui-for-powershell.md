+++
author = "Saggie Haim"
title = 'Build Gui for Powershell'
date = 2019-02-20
draft = false
description = "Learn how to build PowerShell GUI tools with a real life example"
toc = true
tags = [
    "PowerShell"
]
categories = [
    "PowerShell",
    "Automation",
]
keywords = [
    "PowerShell",
    "PowerShell GUI",
    "Automation"
]
image = "/images/pwshguicover.jpg"
+++

Hey everyone! Today I want to talk about GUI for PowerShell.
You probably wonder why? PowerShell is all about the CLI and moving away from the GUI.
but sometimes, a GUI for our scripts can be very useful.

Every Company has HelpDesk team, other System Admins that don’t know PowerShell at all, or non-technical staff.
While I will encourage them to learn, I can’t make them.
As a result, I need to provide them with a GUI for some of the tools or solutions.

![ ](../images/oldladymeme.jpg  "Old Lady meme")

First of all, let’s talk about PowerShell GUI.
In fact, There is no such thing “PowerShell GUI” in the PowerShell world, The GUI part is part of the .NET world (which PowerShell built upon).
There are Two main ways to provide GUI for our scripts and both of them related to .NET:

## Windows Presentation Foundation – WPF

WPF is a subsystem of the .NET framework and can be used for rendering user interfaces in Windows-based applications.
Due to the dependency of .NET, WPF is supported only in Windows-based clients.

## Windows Forms – WinForms

Similarity to WPF, WinForms is also part of the .NET framework.
WinForms is a graphical (GUI) class library included as a part of the Microsoft .NET Framework or Mono Framework, and also supported only in Windows-based clients.

Soon with .NET Core, we will see solutions for Linux GUI also 😃

## The task

In my company, we use Service Accounts, those accounts are used to run Business Application Services, Tasks and more.
They can also be used for developers teams, needing the account for the systems tests run and so.
In order to create a Service Account, one shall follow the following guidelines:

- SamAccountName must start with: Srvc
- Must have a manager (employee who takes responsibility for the account if any anomaly or issue detected)
- Account must have the following description “Service Account for <Account purpose>”
- Account office attribute: Must include “Managed by <Employee name> (<Employee ID>)

Seems like a easy task right? but when you have a team that needs to do it, a few times a week, it becomes error prone quite fast.
So how can we provide a solution?

## The GUI side

I want to provide the team with a tool that will get the information and perform the creation automatically.
In addition, I want to send emails for confirmations and audit.
First, I like to start with the GUI design, it will make it easier to know what functions I need to build in my script.
For small tasks like this, I love to use [POSHGUI](https://poshgui.com/), it’s a web tool that let you build Winforms GUI for PowerShell in minutes.

![My Service Account creation GUI](../images/ServiceAccountGUI.jpg  "Image of my Service Account creation GUI")

I won’t include the “Office” attribute as I have all the information needed to update it myself.

### GUI code overview

First, we need to Add-Type, this cmdlet adds a .NET class to our Powershell Session.
Then we enable visual styles to our form, this adds theme style.

```PowerShell
Add-Type -AssemblyName System.Windows.Forms
[System.Windows.Forms.Application]::EnableVisualStyles()
```

### Global configuration

Secondly, we need to define global properties, including the form itself, the size of the window, title and more:

```PowerShell
#region begin GUI{ 
 
$Form = New-Object system.Windows.Forms.Form # Create a form
$Form.ClientSize = '495,216' # Window size
$Form.text = "Service Account Creator" # Window title
$Form.TopMost = $false #Always on top?
$Form.StartPosition = "CenterScreen" #loads the window in the center of the screen
$Form.FormBorderStyle = 'Fixed3D' #modifies the window border
```

### The Service Account Name

This ‘TextBox’ will be the SamAccountName of the user, as I said before, it must start with Srvc so I typed it in by default as a reminder.

```PowerShell
$SAccountName = New-Object system.Windows.Forms.TextBox # Declare the Lable
$SAccountName.multiline = $false
$SAccountName.text = "Srvc" # Default string
$SAccountName.width = 309 
$SAccountName.height = 20 
$SAccountName.location = New-Object System.Drawing.Point(156, 25)
$SAccountName.Font = 'Microsoft Sans Serif,10'
 
$AccountName = New-Object system.Windows.Forms.Label
$AccountName.text = "Service Account Name"
$AccountName.AutoSize = $true
$AccountName.width = 20
$AccountName.height = 10
$AccountName.location = New-Object System.Drawing.Point(18, 25)
$AccountName.Font = 'Microsoft Sans Serif,10'
```

### Manager Employee ID

In our company the Employee ID is the unique identifier in the Active Directory, we will use it to find the correct employee.
we will use it to fill in the information in the service account and to send him a confirmation mail.

```PowerShell
$ManagerEmployee = New-Object system.Windows.Forms.Label
$ManagerEmployee.text = "Manger Employee ID"
$ManagerEmployee.AutoSize = $true
$ManagerEmployee.width = 25
$ManagerEmployee.height = 10
$ManagerEmployee.location = New-Object System.Drawing.Point(18, 55)
$ManagerEmployee.Font = 'Microsoft Sans Serif,10'
 
$ManagerEmployeeID = New-Object system.Windows.Forms.TextBox
$ManagerEmployeeID.multiline = $false
$ManagerEmployeeID.width = 309
$ManagerEmployeeID.height = 20
$ManagerEmployeeID.location = New-Object System.Drawing.Point(156, 55)
$ManagerEmployeeID.Font = 'Microsoft Sans Serif,10'
```

### Creator Employee ID

Same as the Manager Employee ID, this time for the creator employee ID, we will use this to audit the process and send him a confirmation mail.

```PowerShell
$CreatorEmployee = New-Object system.Windows.Forms.Label
$CreatorEmployee.text = "Creator Employee ID"
$CreatorEmployee.AutoSize = $true
$CreatorEmployee.width = 25
$CreatorEmployee.height = 10
$CreatorEmployee.location = New-Object System.Drawing.Point(18, 82)
$CreatorEmployee.Font = 'Microsoft Sans Serif,10'
 
$CreatorEmployeeID = New-Object system.Windows.Forms.TextBox
$CreatorEmployeeID.multiline = $false
$CreatorEmployeeID.width = 309
$CreatorEmployeeID.height = 20
$CreatorEmployeeID.location = New-Object System.Drawing.Point(156, 82)
$CreatorEmployeeID.Font = 'Microsoft Sans Serif,10'
```

### Description

What is the purpose of this Service account, same as service account name, it must start with “Service Account for ”

```PowerShell
$Description = New-Object system.Windows.Forms.Label
$Description.text = "Description"
$Description.AutoSize = $true
$Description.width = 25
$Description.height = 10
$Description.location = New-Object System.Drawing.Point(18, 110)
$Description.Font = 'Microsoft Sans Serif,10'
 
$TDescription = New-Object system.Windows.Forms.TextBox
$TDescription.multiline = $false
$TDescription.text = "Service Account For"
$TDescription.width = 309
$TDescription.height = 20
$TDescription.location = New-Object System.Drawing.Point(156, 110)
$TDescription.Font = 'Microsoft Sans Serif,10'
```

### Create Account button

Every program needs an action button. Here we need only one button.

```PowerShell
$Create = New-Object system.Windows.Forms.Button
$Create.text = "Create Account"
$Create.width = 111
$Create.height = 30
$Create.location = New-Object System.Drawing.Point(178, 154)
$Create.Font = 'Microsoft Sans Serif,10'
```

### Status Bar

```PowerShell
$StatusBar = New-Object System.Windows.Forms.Label
$StatusBar.width = 495
$StatusBar.height = 20
$StatusBar.location = New-Object System.Drawing.Point(1, 192)
$StatusBar.Font = 'Microsoft Sans Serif,10'
$StatusBar.Text = 'Ready'
$StatusBar.Enabled = $true
```

### Create the form

After we specify each component in our form, we need to invoke it.

```PowerShell
$Form.controls.AddRange(@($SAccountName, $AccountName, $ManagerEmployee, $ManagerEmployeeID, $CreatorEmployee, $CreatorEmployeeID, $Description, $TDescription, $Create, $StatusBar))
```

### Add events

After we have the form, we need to add an event, there are plenty events to execute and you can explore them in POSHGUI, here in our example we need to add click action to our button, we will put there the code we want to execute each time we press the button.

```PowerShell
#region gui events {
$Create.Add_Click( {
# The script that will execute each time we click the button
    })
#endregion events }
```

### wrapping up

Finally, we need to close the GUI section and invoke the ShowDialog() method to display our form

```PowerShell
#endregion GUI }
[void]$Form.ShowDialog()
```

## The PowerShell side

The GUI we have is nice, but it doing nothing.
In order for the GUI to do something when we click on the button “Create Account” we need to add our PowerShell script in the button click action.
But before we do it we need to write some helper functions for our script:

### Get Employee by employeeID

The first function will help us get the information we need for the Manager and Creator employees, it will look for the employee ID in the Active Directory and return the user if found, or $false if not.

```PowerShell
function Get-account ($EmployeeID) {
    $user = Get-ADUser -Properties EmployeeID, EmailAddress -Filter {EmployeeID -eq $EmployeeID} | Select-Object Name, EmailAddress, EmployeeID
    if ($null -eq $user) {
        return $false
    }
    else {
        return $user
    }
}
```

### Generate a Password

Then we need to be able to generate passwords, to do that I’m using my [New Password Generator](https://github.com/SaggiehaimBlog/PS-NewPassword) core function.

```PowerShell
# Create a new password
function new-Password() {
    $inputRange = 48..122
    $inputRange += 33, 35
    $exclude = 91..96
    $exclude += 58..63
    $randomRange = $inputRange | Where-Object { $exclude -notcontains $_}
    for ($i = 0; $i -lt 12; $i++) {
        $rnd = (Get-Random -InputObject $randomRange) 
        $char = [char]$rnd
        $pass += $char
    }
    return $pass
}
```

### Create Mail Reports

Another task we need to perform is to send emails with the account information.
I’m creating two emails, but it’s up to you to decide.
I create one mail for the user creator and the user manager, with the account info and password.
Second mail is for the security team audit, simple mail that state that account has been created, with all the information except the password.

```PowerShell
## Send Mail on Complition
 
function Send-DoneEmail ($Creatoruser, $Manageruser, $NewUser, $password) {
    ##configure SMTP Settings
    $From = "Admin@saggiehaim.met"
    $SMTPServer = "saggiehaim.net"
    $SMTPPort = "25"
    $AdministratorsSMTPAddress = "contact@saggiehaim.net"
    $usersSMTPAddress = $Manageruser.EmailAddress, $Creatoruser.EmailAddress
 
    #Create email and send it to Users.
    $SubjectAdministrators = "New Service Account Has been created for you"
    $bodyToAD = "Hello,<br><br>"
    $bodyToAD += "New Service account has been created for you by: " + $Creator.name + "<br><br>"
    $bodyToAD += "<b>Service account Name:</b> " + $NewUser.SamAccountName + "<br>" 
    $bodyToAD += "<b>Service account Password:</b> " + $password + "<br>"
    $bodyToAD += "<b>You will be asked to change the password after first logon</b><br><br>"
    $bodyToAD += "For Any further assist please contact " + $Creatoruser.name + " by Mail: " + $Creatoruser.EmailAddress + "<br>"
    $bodyToAD += "<br>" 
    $bodyToAD += "Thanks,<br>"
    $bodyToAD += "ServIT<br>" 
 
    Send-MailMessage -From $From -to $usersSMTPAddress -Subject $SubjectAdministrators ` -Body $bodyToAD -BodyAsHtml -Encoding UTF8 -SmtpServer $SMTPServer -port $SMTPPort
            
    #Create email and send it to administrators.
    $SubjectAdministrators = "New Service Account Has been Created"
    $bodyToAD = "Hello,<br><br>"
    $bodyToAD += "New Service Account has beem created.<br>"
    $bodyToAD += "<b>Service Account Name:</b> " + $NewUser.name + "<br>"
    $bodyToAD += "<b>Service Account Description:</b> " + $NewUser.Description + "<br>"
    $bodyToAD += "<b>Service Account Managed by:</b>" + $NewUser.Office + "<br>"
    $bodyToAD += "<b>Service Account Created for:</b> " + $Manageruser.name + "<br>"
    $bodyToAD += "<b>Service Account Created by:</b>" + $Creatoruser.name + "<br>"
    $bodyToAD += "<br>" 
    $bodyToAD += "Active Directory Team<br>" 
 
    Send-MailMessage -From $From -to $AdministratorsSMTPAddress -Subject $SubjectAdministrators ` -Body $bodyToAD -BodyAsHtml -Encoding UTF8 -SmtpServer $SMTPServer -port $SMTPPort
    
}
```

### Create new Service account

Last but not least, the reason we are here, the Service account creation function.
In this function, we specify the account creation OU, the password and the rest of the attribute needed.

```PowerShell
#Create the Service Account
function new-ADServiceAccount ($SAName, $Description, $Creatoruser, $Manageruser) {
    # OU to create the Service Account
    $path = 'OU=Global Service Accounts,DC=Saggiehaim,DC=net'
    $password = new-Password
    $ManagedBy = 'Managed By ' + $Manageruser.Name + ' (' + $Manageruser.EmployeeID + ')' 
    try {
        New-ADUser -SamAccountName $SAName -UserPrincipalName $SAName -DisplayName $SAName -GivenName $SAName -Name $SAName -Description $Description -Office $ManagedBy -AccountPassword (ConvertTo-SecureString -AsPlainText $password -Force) -Enabled $true -ChangePasswordAtLogon $true -Path $path  
    }
    catch {
        return $false
    }
    $NewUser = Get-ADUser -Identity $SAName -Properties Office, Description
    Send-DoneEmail $Creatoruser $Manageruser $NewUser $password
    return $true
    
    
}
```

### The Script

We built the GUI and the functions needed to perform the creation task.
If you remember, when we created the GUI, we add an Action event.
The script we build now is the script we want to execute each time someone presses the key.

### Content validation

We will start with content validation, to make sure that the Help Desk operator is following the Service Account creation guideline.

```PowerShell
if ($SAccountName.Text -eq $null -or $SAccountName.Text -eq '' -or $SAccountName.Text -eq "Srvc") {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Please Make sure you set Service Account name"
            return
        
        }
        elseif ($SAccountName.Text -notlike "Srvc*") {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Please Make sure Service Account Name start with 'Srvc'"
            return
        }
        if ($TDescription.Text -eq $null -or $TDescription.Text -eq '' -or $TDescription.Text -eq "Service Account For") {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Please Make sure to insert Service Account Description!"
            return
        }
        elseif ($TDescription.Text -notlike "Service Account For*") {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Please Start Description with 'Service Account For'"
            return
        }
        if ($ManagerEmployeeID.Text -eq $null -or $ManagerEmployeeID.Text -eq '') {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Please Enter Manager Employee ID"
            return
        }
        if ($CreatorEmployeeID.Text -eq $null -or $creatorEmployeeID.Text -eq '') {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Please Enter Creator Employee ID"
            return
        }
        $Creator = Get-account $CreatorEmployeeID.text
        $Manager = Get-account $ManagerEmployeeID.text
        if ($Creator -eq $false) {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Please Validate Creator Employee ID"
            return 
        }
        if ($Manager -eq $false) {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Please Validate Manager Employee ID"
            return
        }
```

### Creating the Service Account

Now after we finished with the validations, we can create the Service Account:

```PowerShell
        $run = new-ADServiceAccount $SAccountName.text $TDescription.text $Creator $Manager
        if ($run -eq $false) {
            $StatusBar.ForeColor = "#ff0000"
            $StatusBar.Text = "Process Failed. Please verify you have permissions"
            return
        }
        else {
            $StatusBar.ForeColor = "#00b300"
            $StatusBar.Text = "Process Finished with no errors"
        }
```

## Conclusion

We learned how to build GUI for our PowerShell scripts.
How it can help us help others who don’t know yet PowerShell.
And even better, give them the motivation to learn PowerShell and maybe even create their own GUI 😉

I would love to hear your GUI’s Ideas!

You can see the complete script [here](https://github.com/Saggiehaim/Powershell/blob/master/New%20Service%20Account/NewServiceAccount.ps1).
