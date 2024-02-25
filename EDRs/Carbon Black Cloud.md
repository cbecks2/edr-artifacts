# Carbon Black Cloud

Carbon Black Cloud offers two (2) features to interact with an endpoint:\
Live Response, which is a remote shell and available in the Endpoint Standard and Entreprise EDR SKUs.\
Live Query, which is Carbon Black's "wrapper" around osquery which lets you launch osquery queries on an endpoint and get the results back in the console. Available only if you have the Audit & Remediation SKU.


## 1. Network Telemetry
```
*.carbonblack.io
*.confer.net
*.conferdeploy.net
```
## 2. Windows
### 2.1 Process Ancestry

Carbon Black Cloud has two (2) distinct process ancestry depending on the feature used: Live Response or Live Query (osquery).

#### 2.2. Live Response
```
Grandparent Process Path - C:\Program Files\Confer\RepMgr.exe

Parent Process Path - C:\Program Files\Confer\BladeRunner.exe
[For exec command]
Parent Process CommandLine - "C:\Program Files\Confer\BladeRunner.exe" /LiveResponse /doExec /reqId=X /eventUid=X /workingDir="C:\WINDOWS\System32"
[For execfg command]
Parent Process CommandLine - "C:\Program Files\Confer\BladeRunner.exe" /LiveResponse /doExec /reqId=X /eventUid=X /outFile="C:\WINDOWS\system32\output-{0-9}[13].out /outFileMaxSize=26214400 /workingDir="C:\WINDOWS\System32"

Process Path - *user input dependent*
```
Ultimately, the Process Path will be the path of the binary and/or command that is launched through Live Response's exec or execfg command. E.g.: whoami, hostname, ipconfig, net user Aura, etc.

##### 2.2.1. BladeRunner.exe
The /workingDir value depend on which directory the user is in when in the Live Response session, with the default being C:\WINDOWS\System32. That directory can be changed through Live Response's cd command.

The /outFile value is present when an execfg command is used in Live Response. This command returns the output of the Live Response command to the console. The output of the command is saved to that file which is then downloaded by Live Response and the content displayed in the console.

It is unknown (for now) if the /outFileMaxSize value can be changed at a sensor level directly.

#### 2.3. Live Query
```
Grandparent Process Path - C:\Program Files\Confer\RepMgr.exe

Parent Process Path - C:\Program Files\Confer\BladeRunner.exe
Parent Process CommandLine - "C:\Program Files\Confer\BladeRunner.exe" /LiveQuery /queryId=$QUERY_ID /MaxTimeInSeconds=900 /MaxCPUTimeInSeconds=450 /MaxCapCPUPercentage=50 /DynamicCPUCapWindow /MaxMemoryInMb=500

Process Path - C:\Program Files\Confer\Blades\LiveQuery\osqueryi.exe
Process CommandLine - "C:\Program Files\Confer\Blades\LiveQuery\osqueryi.exe" --flagfile="C:\Program Files\Confer\Blades\LiveQuery\$QUERY_ID.flg "$QUERY"

Child Process Path - C:\Program Files\Confer\Blades\LiveQuery\Ext\cbc_plugin_extension.ext.exe
Child Process CommandLine - "C:\Program Files\Confer\Blades\LiveQuery\Ext\cbc_plugin_extension.ext.exe" --socket "\\.\pipe\shell.em" --timeout 30 --interval 1

SMB Pipe - \\.\pipe\shell.em.{0-9}[5]
SMB Pipe Process - C:\Program Files\Confer\Blades\LiveQuery\Ext\cbc_plugin_extension.ext.exe

Created File - C:\Windows\Temp\etilqs_{aA-zZ0-9}[15]
Created by - C:\Program Files\Confer\Blades\LiveQuery\osqueryi.exe
```
##### 2.3.1. BladeRunner.exe
$QUERY_ID refers to the ID of the query that is being launched.

It is unknown (for now) if the various CPU, RAM and execution time parameters can be changed at a sensor level directly.

##### 2.3.2. osqueryi.exe
For osqueryi.exe, $QUERY_ID refers to the ID of the query that is being launched. It should be the same as one in BladeRunner.exe.

For osqueryi.exe, $QUERY refers to the query that is being executed through Live Query. For instance, it could be: SELECT * FROM file WHERE path LIKE "C:\Tools\%%";

For the etilqs file, it has not been investigated yet. It is suspected to hold the results of the Live Query query.

### 2.4. Host Artifacts
Every Carbon Black Cloud Live Response command is logged in the console Audit log.
```
C:\ProgramData\CarbonBlack\Logs\LiveQuery.log
C:\ProgramData\CarbonBlack\Logs\LiveResponse.log
```
LiveQuery.log is a log that holds all the queries that were executed on the endpoint through Live Query.

LiveResponse.log is a log that holds all the commands that were executed on the endpoint through Live Response.

These logs can be obtained by using the "C:\Program Files\Confer\RepCLI.exe" capture command, which creates an archive called psc_sensor.zip. This archive holds the Logs folder in which these logs are located.

## 3. Linux

The information below was gathered on CentOS running the Carbon Black Cloud sensor.

### 3.1. Process Ancestry

Carbon Black Cloud has two (2) distinct process ancestry depending on the feature used: Live Response or Live Query (osquery). On Linux, both the exec and execfg shell commands have the same process ancestry.

#### 3.2. Live Response
```
Grandparent Process Path - /opt/carbonblack/psc/bin/cbagentd

Parent Process Path - /opt/carbonblack/psc/bin/cbagentd
Parent Process CommandLine - /opt/carbonblack/psc/bin/cbagentd --foreground

Process Path - /usr/bin/bash
Process CommandLine - sh -c "*user input dependent*"

Child Process Path - *user input dependent*
Child Process CommandLine - *user input dependent*
```
Ultimately, the Process Path will be the path of the binary and/or command that is launched through Live Response's exec or execfg command. E.g.: whoami, hostname, uname, ip addr, etc.

#### 3.3. Live Query
```
Grandparent Process Path - /opt/carbonblack/psc/bin/cbagentd

Parent Process Path - /opt/carbonblack/psc/bin/cbagentd
Parent Process CommandLine - /opt/carbonblack/psc/bin/cbagentd --foreground

Process Path - /opt/carbonblack/psc/blades/$CLSID/osqueryi
Process CommandLine - /opt/carbonblack/psc/blades/$CLSID/osqueryi --json --logger_min_status 2 "$QUERY"
```
For osqueryi, $CLSID refers to a CLSID that is probably tied/linked to the version of the Carbon Black Cloud agent that is deployed on the system.

For osqueryi, $QUERY refers to the query that is being executed through Live Query. For instance, it could be: SELECT * FROM file WHERE path LIKE "/root/%";

## 4. Sigma

The following Sigma rule(s) can be used to detect the use of Carbon Black Cloud Live Response or Live Query functionalities on a system.

### 4.1. Carbon Black Cloud Live Response on Windows
```
title: Carbon Black Cloud Live Response Usage on Windows
id: 55308b44-289a-4d0e-bb88-5f3a446a2e75
status: experimental
description: This rule detects the usage of Live Response exec and execfg commands from Carbon Black Cloud Live Response on a Windows host.
references:
    - https://docs.vmware.com/en/VMware-Carbon-Black-Cloud/services/carbon-black-cloud-user-guide/GUID-7399B083-D65C-40ED-A831-460F7216F748.html
tags:
    - attack.execution
    - attack.t1059
author: @SecurityAura
date: 2024/02/25
logsource:
    category: process_creation
    product: windows
detection: # TO DO
    selection:
        FieldName: 'StringValue'
        FieldName: IntegerValue
        FieldName|modifier: 'Value'
    condition: selection
fields:
    - fields in the log source that are important to investigate further
falsepositives:
    - Legitimate usage of Carbon Black Cloud Live Response on an endpoint
level: informational
```

## 5. References

https://docs.vmware.com/en/VMware-Carbon-Black-Cloud/services/cbc-sensor-installation-guide/GUID-06D2CB73-968A-466E-BD69-B7480CBA800A.html
https://docs.vmware.com/en/VMware-Carbon-Black-Cloud/services/carbon-black-cloud-user-guide/GUID-7399B083-D65C-40ED-A831-460F7216F748.html#:~:text=To%20use%20Live%20Response%2C%20users,policy%20with%20Live%20Response%20enabled.&text=Click%20Enforce%2C%20then%20Policies.,Select%20a%20policy%20group.
https://docs.vmware.com/en/VMware-Carbon-Black-Cloud/services/carbon-black-cloud-user-guide/GUID-129D4F84-1BF0-49F3-BF95-83002FD63217.html
