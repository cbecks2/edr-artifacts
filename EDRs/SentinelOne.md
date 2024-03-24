# SentinelOne

SentinelOne offers two (2) features to interact with an endpoint:
* Remote Shell (RSH), which drops the user in a PowerShell console shell. In our testing, even if PowerShell Core (6+, 7+) is installed on Windows, you get dropped in a PowerShell 5+ console.
* Remote Script Orchestration (RSO), which allows the user to execute various scripts on remote endpoint(s). For instance, PowerShell scripts (.ps1) on Windows hosts.

## 1. Network Telemetry

## 2. Windows
### 2.1. Process Ancestry
#### 2.1.1. Remote Shell

When you start a Remote Shell session on an endpoint, on Windows, a user called SentinelRSHUser gets used. Therefore, it is easy to search for any command that may have been executed through Remote Shell. Simply look for any process launched by that user (e.g.: DESKTOP\SentinelRSHUser).
```
Grandparent Process Path - C:\Program Files\SentinelOne\Sentinel Agent $VERSION\SentinelAgent.exe
Grandparent Process CommandLine - "C:\Program Files\SentinelOne\Sentinel Agent $VERSION\SentinelAgent.exe"

Parent Process Path - C:\Program Files\SentinelOne\Sentinel Agent $VERSION\SentinelRemoteShellHost.exe
Parent Process CommandLine - "C:\Program Files\SentinelOne\Sentinel Agent $VERSION\SentinelRemoteShellHost.exe" --rsh {0-9}[2] {0-9}[2] "\"C:\WINDOWS\system32\cmd.exe\" /c C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe -NoProfile -NoExit -EncodedCommand aQBmACAA...

Process Path - C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Process CommandLine - "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -NoExit -EncodedCommand aQBmACAA...

Child Process Path - *user input dependent*
Child Process CommandLine - *user input dependent*
```
$VERSION refers to the version of the SentinelOne Agent that is installed, e.g.: 23.1.5.886.

#### 2.1.2. Remote Script Orchestration
```
Grandparent Process Path - C:\Program Files\SentinelOne\Sentinel Agent $VERSION\SentinelAgent.exe
Grandparent Process CommandLine - "C:\Program Files\SentinelOne\Sentinel Agent $VERSION\SentinelAgent.exe"

Parent Process Path - C:\Program Files\SentinelOne\Sentinel Agent $VERSION\Addons\AddonHost.exe
Parent Process CommandLine - "C:\Program Files\SentinelOne\Sentinel Agent $VERSION\Addons\AddonHost.exe" -e {0-9}[4]

Process Path - C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Process CommandLine - "C:\WINDOWS\system32\WindowsPowerShell\v1.0\powershell.exe" -executionpolicy bypass -NonInteractive "C:\ProgramData\Sentinel\Addons\SentinelRSO\Script\$RSO_SCRIPT.ps1 ; exit $LASTEXITCODE"
```
$RSO_SCRIPT refers to the name of the script that is being launched through RSO, e.g.: Get-WindowsServices.ps1.

### 2.2. Host Artifacts
#### 2.2.1. Remote Shell
```
C:\ProgramData\Sentinel\rshTranscripts
```
As of writing this, in our tests, this folder is empty. The transcripts from the Remote Shell sessions can be downloaded from the SentinelOne management console.

However, if you pull a Support Package from the agent via the console, it includes a log called LogCollectorLog.log which shows the following line:
```
agent.remoteShell.logTranscript false
```
Inferring that there may be a way to set that setting to "true" and therefore, get the rshTranscripts folder populated with the actual Remote Shell transcript logs.

## 3. Linux
### 3.1. Process Ancestry
#### 3.1.1. Remote Shell

When you start a Remote Shell session on an endpoint, you get directly dropped in a shell. Which one: bash, sh, etc. may depend on the system configuration. More testing is required to see if alternate shells (ex: zsh) are supported.
```
Grandparent Process Path - /opt/sentinelone/bin/sentinelone-agent
Grandparent Process CommandLine - /opt/sentinelone/bin/sentinelone-agent s1-orchestrator

Parent Process Path - /opt/sentinelone/bin/sentinelone-agent
Parent Process CommandLine - /opt/sentinelone/bin/sentinelone-agent s1-scanner

Process Path - /usr/bin/bash
Process CommandLine - /usr/bin/bash /bin/bash

Child Process Path - *user input dependent*
Child Process CommandLine - *user input dependent*
```

#### 3.1.2. Remote Script Orchestration
```
Grandparent Process Path - /opt/sentinelone/bin/sentinelone-agent
Grandparent Process CommandLine - /opt/sentinelone/bin/sentinelone-agent s1-orchestrator

Parent Process Path - /opt/sentinelone/bin/sentinelone-agent
Parent Process CommandLine - /opt/sentinelone/bin/sentinelone-agent s1-agent

Process Path - /opt/sentinelone/bin/addon-host
Process CommandLine - /opt/sentinelone/bin/addon-host s1-addon /proc/$PID/fd/$FDID

Child Process Path - /opt/sentinelone/addons/data/SentinelRSO/Script/$SCRIPT_NAME
Child Process CommandLine - /opt/sentinelone/addons/data/SentinelRSO/Script/$SCRIPT_NAME
```
As of writing this, research into the $PID for addon-host is still on-going.

As of writing this, research into the $FDID for addon-host is still on-going.

The $SCRIPT_NAME in the Child Process Path refers to the name of the script that was executed via RSO. For instance, the script is called ps.sh for the default SentinelOne Remote Ops "Process List".

### 3.2. Hosts Artifacts
#### 3.2.1. Remote Shell

```
/opt/sentinelone/log/scanner.log
```
This file seems to "log" when a Remote Shell session is initialized. Exerpt from the log:
```
[2024-03-23 18:03:40.627153] [2344] [info] Starting a remote shell session with user 'root'
[2024-03-23 18:03:40.643618] [2344] [info] Successfully marked remote shell as judgeable group
[2024-03-23 18:03:40.643725] [2344] [info] Sending shell master pty fd to network
```

#### 3.3.2. Remote Script Orchestration

Scripts that are executed via RSO will be dropped in the following folder. Though it looks like they get deleted once the RSO task completes (and they get executed).
```
/opt/sentinelone/addons/data/SentinelRSO/Script
```
Traces of the RSO execution are also present in the log below.
```
/opt/sentinelone/log/agent.log
```

Exerpt from that log:
```
[2024-03-24 00:32:59.658556] [2344] [info] Received addon ipc: addon name: SentinelRSO, trace id: $REMOVED.
[2024-03-24 00:32:59.660245] [7625] [info] addon SentinelRSO trace id $REMOVED: downloading resources
[2024-03-24 00:33:00.618439] [7625] [info] addon SentinelRSO trace id $REMOVED: successfully downloaded resources
[2024-03-24 00:33:00.652020] [7625] [info] addon SentinelRSO trace id $REMOVED: addon-host running with pid 41081
[2024-03-24 00:33:12.242244] [7625] [info] addon SentinelRSO trace id $REMOVED: addon-host exited with 0
[2024-03-24 00:33:12.242284] [7625] [info] addon SentinelRSO trace id $REMOVED: executed successfully!
```

## 4. References

https://assets.sentinelone.com/prod/sentinel-one-remote--1

https://www.sentinelone.com/resources/sentinelone-remote-script-orchestration/
