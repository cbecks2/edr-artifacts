# edr-artifacts
This repository is meant to catalog network and host artifacts associated with various EDR products "shell" and response functionalities.

This goal of this project is to assist incident responders, detection engineers, and threat hunters in finding signs of non-approved EDRs being leveraged in their environment outside enumerating installed software.

# Repository Architecture

This repo is mostly split in two sections: EDRs and Sigma.

The EDRs folder contains a single markdown (.md) file for each EDR with information about this product's shell and response functionnalities.

The Sigma folder contains all the Sigma rules related to the usage of these products' shell and response functionnalities.

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
- Tanium
  - Windows
  - macOS

# Contributions

As many of this products are expensive and do not have free trials (for good reason!), this project is best effort and community driven. All suggestions and commits to improve it are welcomed and encouraged.
