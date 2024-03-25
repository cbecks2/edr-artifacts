# edr-artifacts
This repository is meant to catalog network and host artifacts associated with various EDR products "shell" and response functionalities.

This goal of this project is to assist incident responders, detection engineers, and threat hunters in finding signs of non-approved EDRs being leveraged in their environment outside enumerating installed software.

# Repository Architecture

This repo is mostly split in two sections: EDRs and Sigma.

The EDRs folder contains a single markdown (.md) file for each EDR with information about this product's shell and response functionalities.

The Sigma folder contains all the Sigma rules related to the usage of these products' shell and response functionalities.

# Supported EDRs

The initial release of this project on 2024/03/xx covers the following EDRs:
- Carbon Black Cloud (CBC)
  - Windows
  - Linux 
- CrowdStrike (CS)
  - Windows
  - macOS
  - Linux
- Microsoft Defender for Endpoint (MDE)
  - Windows
  - Linux
- SentinelOne (S1)
  - Windows
  - Linux
- Tanium
  - Windows
  - macOS

# Analysis

The information that was collected and documented in this repository has been obtained in various ways. Mainly by using other event telemetry sources to observe what each EDR was doing. On Windows, this can be done via Sysmon and/or enabling Event ID 4688 (Process Creation) with the commandline logging enabled.

On Linux, this can be done by using the new Sysmon for Linux. Granted that the information it returned could be limited and in some scenarios, a second EDR was deployed on the host to observe the behavior of the first one.

# Contributions

As many of this products are expensive and do not have free trials (for good reason!), this project is best effort and community driven. All suggestions and commits to improve it are welcomed and encouraged.

Even more so if you have access to macOS and are able to help document the EDRs that are already part of the project, but didn't go through any macOS testing!
