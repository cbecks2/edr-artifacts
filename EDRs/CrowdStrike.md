# CrowdStrike

## 1. Network Telemetry
```
cloudsink.net
*.cloudsink.net
```

## 2. Windows
## 2.1 Process Ancestry
CrowdStrike Real Time Response calls the CLSIDs below. At the time of this writing investigation into what causes each one to be invoked has not been determined. Note that the process chain below may differ slightly depending on what and how something was invoked.

I personally have only seen false positives from this CLSID, but it was included in the Elastic rule referenced below.

[AB8902B4-09CA-4BB6-B78D-A8F59079A8D5](https://strontic.github.io/xcyclopedia/library/clsid_AB8902B4-09CA-4bb6-B78D-A8F59079A8D5.html) 

```
Grandparent Process Path: C:\Windows\system32\svchost.exe
Grandparent Process Command Line: C:\Windows\system32\svchost.exe -k DcomLaunch -p


Parent Process Path: C:\Windows\system32\DllHost.exe 
Parent Process CommandLine: C:\WINDOWS\system32\DllHost.exe /Processid:{BD07DDB9-1C61-4DCE-9202-A2BA1757CDB2}
Parent Process CommandLine: C:\WINDOWS\system32\DllHost.exe /Processid:{BFCC80BE-AE9A-4E79-8B13-454A10C3A1B1}
Parent Process CommandLine: C:\WINDOWS\system32\DllHost.exe /Processid:{AB8902B4-09CA-4BB6-B78D-A8F59079A8D5}
Parent Process CommandLine: C:\WINDOWS\system32\DllHost.exe /Processid:{338B40F9-9D68-4B53-A793-6B9AA0C5F63B}


Process Path: *user input dependent*
Process CommandLine: *user input dependent*
```

## 3. macOS
### 3.1 Process Ancestry

This may differ depending on what tool you use to view Mac Process Trees.

Run Script:
```
Grandarent Process Path: /Library/SystemExtensions/<system extension guid>/com.crowdstrike.falcon.Agent.systemextension/Contents/MacOS/com.crowdstrike.falcon.Agent
Grandparent Process CommandLine: /Library/SystemExtensions/<system extension guid>/com.crowdstrike.falcon.Agent.systemextension/Contents/MacOS/com.crowdstrike.falcon.Agent

Parent Process Path: /bin/zsh
Parent Process CommandLine:
/bin/zsh -c #!/bin/zsh
# SYNOPSIS
#    Run a zsh script with specified command line and timeout
# DESCRIPTION
#   CrowdStrike Real Time Response command
# PARAMETER WD
#    Current working directory
# PARAMETER Param1
#    Script body - Required
# PARAMETER Param2
#    Command line sent to script
# PARAMETER Param3
#    Local path - path to local file to be used as script body
# PARAMETER Param4
#    Timeout in seconds
# NOTES
#    File Name  : runscript.sh
#    Contact    : support@crowdstrike.com
#    Copyright  : CrowdStrike 2020
# LINK
#    https://www.crowdstrike.com/

set -euo pipefail

function err_exit {
    >&2 echo "$1"
    exit
}

# following Windows behavior where HostPath clobbers Raw
if [ -n "$3" ]; then
    if ! [ -f "$3" ] || ! [ -r "$3" ]; then
        err_exit "Cannot read file specified by -HostPath."
    fi
    if ! grep -qI '' "$3"; then
        err_exit "File specified by -HostPath must not be binary. Use run command instead."
    fi
    SCRIPT=$(<${3})
elif [ -n "$1" ]; then
    SCRIPT="$1"
else
    err_exit "Either -Raw or -HostPath is required."
fi

COMMAND_LINE="$2"

TIMEOUT="60"
if [ -n "$4" ]; then
    case $4 in
    ''|*[!0-9]*)
        err_exit "-Timeout must be a positive integer."
        ;;
     *)
        TIMEOUT=$4
        ;;
    esac
fi

# "z" will ensure that arguments are separated, respecting escape sequences
# "Q" will strip off outer quotes added when we set COMMAND_LINE above
/bin/zsh -c "$SCRIPT" /bin/zsh ${(Q)${(z)COMMAND_LINE}} &
pid=$!

waited=0
wait_interval=0.1
print_interval=5
while kill -0 $pid 2>/dev/null ; do
    if ((waited * wait_interval >= TIMEOUT)); then
        kill -9 $pid >/dev/null 2>&1
        err_exit "Timed out waiting for script to exit."
    fi
    if ((waited > 0 && waited * wait_interval % print_interval == 0)); then
        LC_ALL=en_US.UTF-8 printf %b '\u200b'
    fi
    /bin/sleep $wait_interval
    ((waited += 1))
done /bin/zsh *user input here*


Process Path: *user dependent*
Process CommandLine: *user dependent*
```

Put File:
```
Parent Process Path: /sbin/launchd
Parent Process CommandLine: /sbin/launchd


Process Path: /Library/SystemExtensions/<system extension guid>/com.crowdstrike.falcon.Agent.systemextension/Contents/MacOS/com.crowdstrike.falcon.Agent
Process CommandLine: /Library/SystemExtensions/<system extension guid>/com.crowdstrike.falcon.Agent.systemextension/Contents/MacOS/com.crowdstrike.falcon.Agent


Files are created in the first directory below. They are then moved and the permissions are changed to the path in RTR.
File Create: /System/Volumes/Data/Library/Application Support/CrowdStrike/Falcon/*user input dependent*
File Create: /System/Volumes/Data/*user input dependent*

For example:
File Create: /System/Volumes/Data/Library/Application Support/CrowdStrike/Falcon/myfile.zip
File Create: /System/Volumes/Data/Users/user/Desktop/myfile.zip
```

## 4. Linux
### 4.1 Process Ancestry

This may differ depending on what Linux distribution is running. More testing needed.

Interactive Shell:
```
Grandparent Process Path: /opt/CrowdStrike/falcond<sensor-version>
Grandparent Process CommandLine: /opt/CrowdStrike/falcond
Example Grandparent Path: /opt/CrowdStrike/falcond15805

Parent Process Path: /opt/CrowdStrike/falcon-sensor<sensor-version>
Parent Process CommandLine: falcon-sensor
Example Path: /opt/CrowdStrike/falcon-sensor16108

Process Path: *user input dependent*
Process CommandLine: *user input dependent*

```

Run Script:
```
Parent Process Path: /opt/CrowdStrike/falcon-sensor<sensor-version>
Parent Process CommandLine: falcon-sensor
Example Path: /opt/CrowdStrike/falcon-sensor16108

Parent Process Path: /bin/bash
Parent CommandLine: /bin/bash -c #!/bin/bash # SYNOPSIS # Run a bash script with specified command line and timeout # DESCRIPTION # CrowdStrike Real Time Re /bin/bash <user input>
Example Parent CommandLine: /bin/bash -c #!/bin/bash # SYNOPSIS # Run a bash script with specified command line and timeout # DESCRIPTION # CrowdStrike Real Time Re /bin/bash echo "Hello from the otherside!"

Process Path: /bin/bash
CommandLine: /bin/bash -c #!/bin/bash # SYNOPSIS # Run a bash script with specified command line and timeout # DESCRIPTION # CrowdStrike Real Time Re /bin/bash <user input>
Example CommandLine:
/bin/bash -c #!/bin/bash # SYNOPSIS # Run a bash script with specified command line and timeout # DESCRIPTION # CrowdStrike Real Time Re /bin/bash echo "Hello from the otherside!"
```

Put File:
```
Grandparent Process Path: /opt/CrowdStrike/falcond<sensor-version>
Grandparent Process CommandLine: /opt/CrowdStrike/falcond
Example Grandparent Path: /opt/CrowdStrike/falcond15805

Parent Process Path: /opt/CrowdStrike/falcon-sensor<sensor-version>
Parent Process CommandLine: falcon-sensor
Example Path: /opt/CrowdStrike/falcon-sensor16108

Process Path: /bin/bash
Process CommandLine: 
/bin/bash -c #!/bin/bash
# SYNOPSIS
#   Rename a downloaded file from the Falcon cloud
# DESCRIPTION
#   CrowdStrike Real Time Response comma /bin/bash <filename> <current rtr path> <filename>

Process Command Line Example:
/bin/bash -c #!/bin/bash
# SYNOPSIS
#   Rename a downloaded file from the Falcon cloud
# DESCRIPTION
#   CrowdStrike Real Time Response comma /bin/bash file.zip /tmp file.zip       

```

## 5. References

https://github.com/elastic/protections-artifacts/blob/main/behavior/rules/execution_unusual_powershell_engine_imageload.toml
