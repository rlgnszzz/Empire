---
date: 2016-03-09T00:11:02+01:00
title: Empire Core
weight: 12
---

## Agent Types
Within Empire, the implant currently uses two primary languages:

* Python - any platform that supports python 2.5+
* PowerShell - any Windows platform that supports PowerShell v2

Its not always clear what and where something works when you begin to support multi-platform architecture such as Empire. This has always been a design challenge for the development team to keep it clean and logical.
Empire as a project has multiple supported platforms, but simply we have built for the core three operating systems:

1. Linux - Debian, Ubuntu, Red Hat, \*NIX
2. Windows - Windows 7+ (Anything that has the .NET core and PSv2)
3. MacOS - Also commonly referred to as OS X

## Stagers
To reveal the loaded stagers we simply use the `usestager` command and tab.
```
(Empire) > usestager osx/
applescript      ducky            jar              macho            pkg              teensy           
application      dylib            launcher         macro            safari_launcher
```
We can also quick fill the stager module data with the `Listeners` data. Using the `usestager osx/applescript http` while replacing `http` with your listener name.

### OSX Stagers
```
(Empire: stager/osx/applescript) > info

Name: AppleScript

Description:
  Generates AppleScript to execute the Empire stage0
  launcher.

Options:

  Name             Required    Value             Description
  ----             --------    -------           -----------
  Listener         True        http              Listener to generate stager for.
  OutFile          False                         File to output AppleScript to, otherwise
                                                 displayed on the screen.
  SafeChecks       True        True              Switch. Checks for LittleSnitch or a
                                                 SandBox, exit the staging process if
                                                 true. Defaults to True.
  Language         True        python            Language of the stager to generate.
  UserAgent        False       default           User-agent string to use for the staging
                                                 request (default, none, or other).
```


### PowerShell Stagers

##

##

##


##


##


##


##


##
