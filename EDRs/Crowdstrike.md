# Crowdstrike

## 1. Network Telemetry
```
cloudsink.net
*.cloudsink.net
```


## 2. Process Ancestry
Crowdstrike Real Time Response calls the CLSIDs below. At the time of this writing investigation into what causes each one to be invoked has not been determined. Note that the process chain below may differ slightly depending on what and how something was invoked.

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


## 3. Host Artifacts


## 4. References

* https://github.com/elastic/protections-artifacts/blob/main/behavior/rules/execution_unusual_powershell_engine_imageload.toml
