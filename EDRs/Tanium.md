# Tanium

You are likely to have a hard timing hunting on this behavior given how the tool operates. Nonetheless, we are including it in the project in case you find it useful.

## 1. Network Telemetry
```
*.cloud.tanium.com (if cloud hosted)
*.distribute.tanium.com (if cloud hosted)
```

## 2. Windows
### 2.1. Process Ancestry
```
Grandparent Process Path: C:\Program Files (x86)\Tanium\Tanium Client\TaniumClient.exe
Grandparent Process CommandLine: "C:\Program Files (x86)\Tanium\Tanium Client\TaniumClient.exe" -c


Parent Process Path: C:\Program Files (x86)\Tanium\Tanium Client\TaniumClient.exe
Parent Process CommandLine: "C:\Program Files (x86)\Tanium\Tanium Client\TaniumClient.exe" -c


Process Path: *user input dependent*
Process CommandLine: *user input dependent*
```

### 2.2. Host Artifacts
```
C:\Program Files (x86)\Tanium\Tanium Client\Downloads\[0-9]+\.log
C:\Program Files (x86)\Tanium\Tanium Client\Downloads\Action_[0-9]+\*
```
For instance, if the number of the action is 1247444, the resulting log will be Action_1247444.log and the folder Action_1247444. That folder will then hold anything associated with that Action that was not already on the disk.

## 3. macOS
### 3.1. Process Ancestry

This may differ depending on what tool you use to view Mac Process Trees.
```
Grandparent Process Path: /Library/Tanium/TaniumClient/TaniumClient
Grandparent Process CommandLine: /Library/Tanium/TaniumClient/TaniumClient -d


Parent Process Path: /Library/Tanium/TaniumClient/TaniumClient
Parent Process CommandLine: /Library/Tanium/TaniumClient/TaniumClient -c

Process Path: *user input dependent*
Process CommandLine: *user input dependent*
```

### 3.2. Host Artifacts
```
/Library/Tanium/TaniumClient/Downloads/Action_[0-9]/*
/Library/Tanium/TaniumClient/Downloads/Action_[0-9]+\.log
```

## 4. References
