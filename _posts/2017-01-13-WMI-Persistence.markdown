---
layout: post
title:  "WMI Persistence with Cobalt Strike"
date:   2017-01-20 19:45:31 +0530
categories: Archive
author: Julian Catrambone
tags: [WMI, Persistence]
---

Lets be honest implementing persistence on a pentest can be hard, messy, and get you caught.  Fuzzy Security did a great overview of some of the most common techniques used today and how to implement them.  You can find that blog post [here](http://www.fuzzysecurity.com/tutorials/19.html).

Some of the persistence techniques mentioned are:

1. Persistence through the Registry
2. Scheduled Backdoors using Scheduled Tasks
3. Process Resource Hooking
4. Persistence through the MSDTC Service
5. WMI Permanent Event Subscriptions

Personally I like to use WMI as my persistence mechanism.  It is hard to detect, difficult to remove, doesn't require payloads saved on disk, and **_can_** be implemented easily. So how does it work?  Well, at a very high level to establish persistence it is a three step process.

1. Establish an event filter to trigger on system boot
2. Establish an command line consumer to run the payload
3. Establish a Binding to active the command line consumer, when the event filter is activated

To make this process a little easier I decided to make a quick PowerShell script.  The script is available on [Github](https://github.com/jcatrambone94/WMI-Persistence).  Most of the code I used came from research done by [Matt Graeber](https://twitter.com/mattifestation?lang=en) and some help from [Andrew Luke](ttps://twitter.com/Sw4mp_f0x).  Specifically, [here](https://www.fireeye.com/content/dam/fireeye-www/global/en/current-threats/pdfs/wp-windows-management-instrumentation.pdf) and [here](https://www.blackhat.com/docs/us-15/materials/us-15-Graeber-Abusing-Windows-Management-Instrumentation-WMI-To-Build-A-Persistent%20Asynchronous-And-Fileless-Backdoor-wp.pdf) are two resources that I relied on heavily.

To use this script simply edit the Payload to match your current environment.  Import the script and run Install-Persistence.
![Install-Persistence]({{ site.url }}/assets/WMI-Persistence/Install.png)

To make sure it installed correctly, simply run Check-WMI.
![Check-Persistence]({{ site.url }}/assets/WMI-Persistence/Check.png)

Finally, to remove persistence, ensure that the variables for $EventFilterName and $EventConsumerName match the names assigned when it was installed.  By default these values are 'Cleanup' and 'DataCleanup' Respectively.  Then run Remove-Persistence to remove each element of persistence.

There are also a ton of tools that you can use to establish persistence using WMI, like [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) and [PowerLurk](https://github.com/Sw4mpf0x/PowerLurk/blob/master/PowerLurk.ps1).

If you have any feedback, advice, or comments let me know!
