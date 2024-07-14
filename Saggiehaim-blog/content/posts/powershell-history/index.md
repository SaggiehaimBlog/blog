+++
author = "Saggie Haim"
title = 'PowerShell History'
date = 2018-11-28
draft = false
description = "Technically, PowerShell has two type of commands history. One is the command line buffer, which is part of the graphical terminal application. Second is the built-in command history feature, that provides detailed information about the commands you have run."
toc = true
tags = [
    "PowerShell"
]
categories = [
    "PowerShell"
]
keywords = [
    "PowerShell",
    "Command History",
    "PowerShell sessions",
    "PowerShell cmdlets",
    "PowerShell buffer",
    "Get-History",
    "Add-History",
    "Clear-History",
    "Invoke-History",
    "PowerShell automation",
]
image = "/images/historycover.png"
+++

Howdy folks! How are you today?

After my last post, I had to prepare for my trip to the USA, had to visit two of our offices and set up a new one.
Unfortunately, the big California fire started… so things ware little tense during my stay, hope everyone is safe over there!

While working there, my colleague asks me about some cmdlets I ran in the last site we visited.
I had a few PowerShell Sessions open, and I wasn’t sure in which one I ran the cmdlets, so I had to bring my investigator glasses.

![I will see myself out](./images/invest.jpg  "bad eel joke meme")

So, let’s talk about PowerShell history. Technically, PowerShell has two type of commands history.
One is the command line buffer, which is part of the graphical terminal application.
Second is the built-in command history feature, that provides detailed information about the commands you have run.

## So, what are the differences?

### Command Line Buffer

By default, the buffer remembers the last 50 commands typed in.
Also, as we said before because is part of the graphical terminal application, it’s not limited to the current PowerShell session (YES!).
that means that every time we open a new session, we will have the same buffer, but know that if you already have two sessions running, you will have to reopen one of them in order to refresh the buffer.

#### How to use?

**Up and Down keys:**
Navigate thru the buffer list, pressing the Up key will recall the last command you type, continue pressing the up key to move further in the list.
Pressing the Down key will move the other way on the list.
F8: one of the coolest features in my opinion. You can’t view your buffer list, which makes it harder to find what you want, but with the help of F8, we can search thru the buffer.
just type in the cmdlet you are looking for and press F8.
Not what you looked for? press F8 again to further search the list.

#### how to change the buffer size?

We can increase the size of the buffer easily if we want, but pressing the right click on the PowerShell Title Bar, select “Defaults”, under the “Options” tab, change the value under “Buffer Size”

![ ](./images/buffer.png  "PowerShell Terminal buffer setting")

### PowerShell History Cmdlet

Windows PowerShell itself keeps a history of the commands you have typed in.
but this time its unique to the current session. this means that unlike the buffer, it won’t be shared across sessions, and won’t save after you close the session.
the default history size is determined by the parameter $MaximumHistoryCount (from version 3.0 value set to 4096)

#### how to use?

**Get-History**
To view the command line history, use the following cmdlet:

```PowerShell
Get-History
```

We can also use the “h” alias if you want.
Each cmdlet gets an ID in the order they executed.
We can recall specific cmdlet from the history with it ID.

```PowerShell
Get-History -Id 3
```

> cool tip: If we need to copy cmdlet from history to somewhere else, you can pipe it to “clip” and it will be ready in your clipboard.

```PowerShell
(Get-History -Id 1).CommandLine | clip
```

To search the history, we can use the Select-String cmdlet:

```PowerShell
Get-History | Select-String "Saggiehaim"
```

If you need to see detailed information about each command you execute, you can use the Format-List:

```PowerShell
Get-History | Format-List -Property *
```

If we need to save our history list for future purposes, we can export the list to CSV or XML files.
When we export the history, the file includes the data that displayed when we format the history as a list, including all the details.

```PowerShell
Get-History | Export-Csv History.csv
```

#### Add-History

We can use the Get-History cmdlet to import the history list we exported.

```PowerShell
Import-Csv history.csv | Add-History
```

#### Clear History

If we need for some reasons to clear the history and start clean, we can do run the following cmdlet:

```PowerShell
Clear-History
```

We can also delete specific cmdlet from history, using:

```PowerShell
Clear-History -Id #
```

#### Invoke History

If we want to re-run a command from the history list, we can use the invoke history cmdlet.
First, we need to check the ID of the cmdlet we want to run from history, then we can use the following:

```PowerShell
Invoke-History -Id #
```

![Invoke-History in action ](./images/invoke.png  "PowerShell Terminal running the Invoke-History")

If needed, we can run more than one cmdlet:

```PowerShell
## Choosing specific cmdlets
1,5,7 | ForEach {Invoke-History -Id $_ } 

## Running a sequence
10..20 | ForEach {Invoke-History -Id $_ }
```

## Combining everything

So, we can view the PowerShell history, export it, import it and run cmdlets from the list.
That nice, but we can take it one step ahead.
We worked in a session, made some trials and errors.
and ended with a collection of cmdlets that fit our needs, it's not a script material, but it's a sequence of cmdlets we need to complete our task.
We can use the history cmdlets to our advantage.
We can export the PowerShell history list and import it to any session we need,
then we can invoke the cmdlets one by one.
creating a “script” in seconds:

```PowerShell
Import-Csv history.csv | Add-History -PassThru | ForEach-Object -Process {Invoke-History}
```

## Summary

Now that we know how to search the PowerShell history and use the buffer.
We can be much more productive! I encourage you to try and explore all the cmdlets we talked about here.
get familiar with it and add it to your skills set.

See you next time!
