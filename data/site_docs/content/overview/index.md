---
date: 2016-03-09T00:11:02+01:00
title: Overview
weight: 11
---
## Quick Start
### Main Menu
Once you hit the main menu, you'll see the number of active agents, listeners, and loaded modules.

```
================================================================
 [Empire]  Post-Exploitation Framework
================================================================
 [Version] 2.3 | [Web] https://github.com/empireProject/Empire
================================================================

   _______ .___  ___. .______    __  .______       _______
  |   ____||   \/   | |   _  \  |  | |   _  \     |   ____|
  |  |__   |  \  /  | |  |_)  | |  | |  |_)  |    |  |__
  |   __|  |  |\/|  | |   ___/  |  | |      /     |   __|
  |  |____ |  |  |  | |  |      |  | |  |\  \----.|  |____
  |_______||__|  |__| | _|      |__| | _| `._____||_______|


       282 modules currently loaded

       0 listeners currently active

       0 agents currently active


(Empire) >
```

The **help** command should work for all menus, and almost everything that can be tab-completable is (menu commands, agent names, local file paths where relevant, etc.).

You can ctrl+C to rage quit at any point. Starting Empire back up should preserve existing communicating agents, and any existing listeners will be restarted (as their config is stored in the sqlite backend database).

### Listeners 101
The first thing you need to do it set up a local listener. The **listeners** command will jump you to the listener management menu. Any active listeners will be displayed, and this information can be redisplayed at any time with the **list** command. The `uselistener` command will allow you to select the type of listener. Hitting TAB after this command will show all available listener types. The info command will display the currently set listener options.

```
(Empire: listeners) > uselistener
dbx           http          http_com      http_foreign  http_hop      http_mapi     meterpreter   
(Empire: listeners) > uselistener http
(Empire: listeners/http) > info

    Name: HTTP[S]
Category: client_server

Authors:
  @harmj0y

Description:
  Starts a http[s] listener (PowerShell or Python) that uses a
  GET/POST approach.

HTTP[S] Options:

  Name              Required    Value                            Description
  ----              --------    -------                          -----------
  SlackToken        False                                        Your SlackBot API token to communicate with your Slack instance.
  ProxyCreds        False       default                          Proxy credentials ([domain\]username:password) to use for request (default, none, or other).
  KillDate          False                                        Date for the listener to exit (MM/dd/yyyy).
  Name              True        http                             Name for the listener.
  Launcher          True        powershell -noP -sta -w 1 -enc   Launcher string.
  DefaultDelay      True        5                                Agent delay/reach back interval (in seconds).
  DefaultLostLimit  True        60                               Number of missed checkins before exiting
  WorkingHours      False                                        Hours for the agent to operate (09:00-17:00).
  SlackChannel      False       #general                         The Slack channel or DM that notifications will be sent to.
  DefaultProfile    True        /admin/get.php,/news.php,/login/ Default communication profile for the agent.
                                process.php|Mozilla/5.0 (Windows
                                NT 6.1; WOW64; Trident/7.0;
                                rv:11.0) like Gecko
  Host              True        http://172.17.0.2:80             Hostname/IP for staging.
  CertPath          False                                        Certificate path for https listeners.
  DefaultJitter     True        0.0                              Jitter in agent reachback interval (0.0-1.0).
  Proxy             False       default                          Proxy to use for request (default, none, or other).
  UserAgent         False       default                          User-agent string to use for the staging request (default, none, or other).
  StagingKey        True        TtcRbe!5{h#VWyNr1Sm47paJ8Czo?Mn@ Staging key for initial agent negotiation.
  BindIP            True        0.0.0.0                          The IP to bind to on the control server.
  Port              True        80                               Port for the listener.
  ServerVersion     True        Microsoft-IIS/7.5                Server header for the control server.
  StagerURI         False                                        URI for the stager. Must use /download/. Example: /download/stager.php


(Empire: listeners/http) >
```

The info command will display the currently configured listener options. Set your host/port by doing something like set Host http://192.168.52.142:8081. This is tab-completable, and you can also use domain names here). The port will automatically be pulled out, and the backend will detect if you're doing a HTTP or HTTPS listener. For HTTPS listeners, you must first set the CertPath to be a local .pem file. The provided **./setup/cert.sh** script will generate a self-signed cert and place it in **./data/empire.pem**.

Set optional and WorkingHours, KillDate, DefaultDelay, and DefaultJitter for the listener, as well as whatever name you want it to be referred to as. You can then type **execute** to start the listener. If the name is already taken, a nameX variant will be used, and Empire will alert you if the port is already in use.

### Stagers 101
Empire implements various stagers in a modular format in **./lib/stagers/* **. These include dlls, macros, one-liners, and more. To use a stager, from the main, listeners, or agents menu, use usestager [tab] to tab-complete the set of available stagers, and you'll be taken to the individual stager's menu. The UI here functions similarly to the post module menu, i.e set/unset/info and generate to generate the particular output code.

For UserAgent and proxy options, default uses the system defaults, none clears that option from being used in the stager, and anything else is assumed to be a custom setting (note, this last bit isn't properly implemented for proxy settings yet). From the Listeners menu, you can run the **launcher [listener ID/name]** alias to generate the stage0 launcher for a particular listener (this is the stagers/launcher module in the background). This command can be run from a command prompt on any machine to kick off the staging process.

### Agents 101
You should see a status message when an agent checks in (i.e. [+] Initial agent CGUBKC1R3YLHZM4V from 192.168.52.168 now active). Jump to the Agents menu with **agents**. Basic information on active agents should be displayed. Various commands can be executed on specific agent IDs or **all** from the agent menu, i.e. **kill all**. To interact with an agent, use **interact AGENT_NAME**. Agent names should be tab-completable for all commands.

```
[+] Initial agent P20XQ4XJ from 172.17.0.2 now active (Slack)


(Empire: stager/multi/launcher) > agents

[*] Active agents:

  Name            Lang  Internal IP     Machine Name    Username            Process             Delay    Last Seen
  ---------       ----  -----------     ------------    ---------           -------             -----    --------------------
  P20XQ4XJ        py    172.17.0.2      e7059c42661b    *root               python/59           5/0.0    2017-12-05 22:29:53

(Empire: agents) > interact P20XQ4XJ
(Empire: P20XQ4XJ) >
```

In an Agent menu, **info** will display more detailed agent information, and help will display all agent commands. If a typed command isn't resolved, Empire will try to interpret it as a shell command (like ps). You can **cd** directories, **upload/download** files, and **rename NEW_NAME**.

For each registered agent, a **./downloads/AGENT_NAME/** folder is created (this folder is renamed with an agent rename). An ./agent.log is created here with timestamped commands/results for agent communication. Downloads/module outputs are broken out into relevant folders here as well.

When you're finished with an agent, use **exit** from the Agent menu or **kill NAME/all** from the Agents menu. You'll get a red notification when the agent exits, and the agent will be removed from the interactive list after.

### Modules 101
To see available modules, type **usemodule [tab]**. To search module names/descriptions, use **searchmodule privesc** and matching module names/descriptions will be output.

To use a module, for example share finder from PowerView, type **usemodule situational_awareness/network/sharefinder** and press enter. info will display all current module options.

```
(Empire: P20XQ4XJ) > usemodule situational_awareness/network/port_scan
(Empire: python/situational_awareness/network/port_scan) > info

              Name: Port Scanner.
            Module: python/situational_awareness/network/port_scan
        NeedsAdmin: False
         OpsecSafe: True
          Language: python
MinLanguageVersion: 2.6
        Background: True
   OutputExtension: None

Authors:
  @424f424f

Description:
  Simple Port Scanner.

Comments:
  CIDR Parser credits to http://bibing.us.es/proyectos/abrepro
  y/12106/fichero/ARCHIVOS%252Fservidor_xmlrpc%252Fcidr.py

Options:

  Name   Required    Value                     Description
  ----   --------    -------                   -----------
  Target True                                  Targets to scan in single, range 0-255  
                                               or CIDR format.                         
  Agent  True        P20XQ4XJ                  Agent to execute module on.             
  Port   True        8080                      The port to scan for.                   

```

To set an option, like the port scanner, use `set Port 80`. The Agent argument is always required, and should be auto-filled from jumping to a module from an agent menu. You can also **set Agent [tab]** to tab-complete an agent name. **execute** will task the agent to execute the module, and **back** will return you to the agent's main menu. Results will be displayed as they come back.

### Scripts
In addition to formalized modules, you are able to simply import and use a .ps1 script in your remote empire agent. Use the **scriptimport ./path/** command to import the script. The script will be imported and any functions accessible to the script will now be tab completable using the "scriptcmd" command in the agent. This works well for very large scripts with lots of functions that you do not want to break into a module.

## Architecture
TODO

## Staging
The key-exchange protocol used by Empire is called [Encrypted Key Exchange](https://en.wikipedia.org/wiki/Encrypted_key_exchange) (EKE). There’s a better overview [here](http://stackoverflow.com/questions/15779392/encrypted-key-exchange-understanding).

For Empire, a small launcher (a basic proxy-aware IEX download cradle) is used to download/execute the patched **./data/stager.ps1** script. The URI resource for this request can be specified in **./setup_database.py** under the **STAGE0_URI** paramater. The **stager.ps1** is case-randomized then XOR encrypted with the AES staging key from the database config. This means the key-negotiation stager delivered to each agent will be randomized/different per server, but will be static for each server instance. The staging key is sent with the launcher in order to decrypt the stager, so is assumed to be "burned" by network defenders.

This stager generates a randomized RSA private/public key pair in memory, uses the AES staging key to post the encrypted RSA public key to the **STAGE1_URI** resource (also specifiable in **./setup_database.py**). A random 12-character SESSIONID is also generated at this point, which is the initial name the agent uses to check in. After this post, the server returns the server’s epoch time and a randomized AES session key, encrypted in the agent’s public key.

The agent decrypts the values, gathers basic system information, and posts this information to the server encrypted with the new AES session key to **STAGE2_URI**. The server then returns the patched **./data/agent.ps1**, which can be thought of as the standard API. From here, the agent starts its beaconing behavior.

![alt text](/images/empire_staging_process.png)

## Multi-Platform Support
TODO
One of the powerhouse parts of 
