---
published: true
date: '2026-02-04 16:20 +0800'
title: Untitled
author: Fung Lim
excerpt: 'Application Note: Using Crosswork Planning Startup Script'
tags:
  - Crosswork Planning
  - Crosswork Network Controller
---
{% include toc %}

# Application Note: Using Crosswork Planning Startup Script

## Overview

This application note describes how an external script may be used within **Cisco Crosswork Planning 7.2** as a **Startup Script** in the data collection workflow. Our example script `get_plan_cp_cw.py` retrieve the real time network model in the form of a planfile from **Crosswork Network Controller (CNC)** and converts it for use in Crosswork Planning. Since the startup script can be scheduled to execute at a regular cadence as part of a collection workflow, it allows for a automated archival of planfiles for offline planning.

## About Cisco Crosswork Planning

Cisco Crosswork Planning provides toolsets for network operators to create and maintain a model of the current network through the collection and analysis of the network and the traffic demands placed on it. At a given time, this network model contains all relevant information about a network, including topology, traffic and LSPs and their attributes. You may use this information as a basis for analyzing the impact on the network due to changes in traffic demands, paths, node and link failures, link metrics, or others. Crosswork Planning comprises two components: **Crosswork Planning Collector** (component that create, maintain, and archive a network model) and **Crosswork Planning Design** (component for predictive what-if simulation analysis, growth planning, and network optimization and design).

## New Features in Crosswork Planning 7.2 (Startup Script Related)

Cisco Crosswork Planning 7.2 introduces several new capabilities:

| Feature | Description |
|---------|-------------|
| **Startup Script Support** | You can now configure an external script as the first step in the collection configuration chain instead of the existing mandatory IGP/SR-PCE collectors. |
| **Dynamic Data File Access** | Upload data files directly to the Collector. External scripts can access these files at runtime without requiring script repackaging or redeployment. |
| **Import .db Plan Files** | You can now import plan files with a `.db` extension into user space—the format produced by startup scripts. |

For more information, see the *"Run an external script as a startup script"* section in the [Cisco Crosswork Planning 7.2 Collection Setup and Administration](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-collectors-in-cp.html#run-startup-script) document.

## Background: Startup Scripts in Crosswork Planning

As documented in the [Cisco Crosswork Planning 7.2 Collection Setup Guide](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-collectors-in-cp.html#run-startup-script), Crosswork Planning supports running an **external script as the first step** in a collection configuration chain.

Key characteristics of startup scripts:

- Executed **before** any other collectors in the chain
- Only **one startup script** is allowed per collection chain
- When a startup script is configured, the IGP database or SR-PCE collector becomes **optional**
- The startup script output can serve as a **source** for downstream collectors
- Supported languages: **Python, Shell, Perl**
- Valid file formats: `.py`, `.sh`, `.pl`, `.zip`, `.tar`, `.gz`, `.tar.gz`
- Startup script must produce a valid database (.db) file for ingestion to Crosswork Planning

## Script Purpose

`get_plan_cp_cw.py` serves as a startup script that:

1. **Authenticates** to Crosswork Network Controller via SSO
2. **Retrieves** the current network plan file using the CNC RESTCONF API
3. **Converts** the plan file to `.db` format required by Crosswork Planning

This enables Crosswork Planning to use the live network model from CNC as its collection source, rather than performing independent topology discovery. 

## Script Architecture

### Authentication Flow

```
┌─────────────────────┐    ┌─────────────────────────────────┐
│  get_plan_cp_cw.py  │───▶│  CNC SSO Endpoint               │
│                     │    │  https://<IP>:30603/crosswork/  │
│                     │    │  sso/v1/tickets                 │
└─────────────────────┘    └─────────────────────────────────┘
         │                              │
         │  1. POST username/password   │
         │◀─────────────────────────────│
         │     Returns: TGT ticket      │
         │                              │
         │  2. POST TGT + service URL   │
         │◀─────────────────────────────│
         │     Returns: JWT token       │
         ▼
```

### Plan Retrieval Flow

```
┌─────────────────────┐    ┌─────────────────────────────────┐
│  get_plan_cp_cw.py  │───▶│  CNC Optimization Engine API    │
│  (with JWT token)   │    │  /crosswork/nbi/optima/v2/      │
│                     │    │  restconf/operations/...        │
└─────────────────────┘    └─────────────────────────────────┘
         │                              │
         │  POST: get-plan request      │
         │  - version: "7.10" (Default) │
         │  - format: "pln" or "txt"    │
         │◀─────────────────────────────│
         │  Response: base64 planfile   │
         ▼
┌─────────────────────┐
│   mate_convert      │
│   -plan-file X.pln  │
│   -out-file Y.db    │
└─────────────────────┘
         │
         ▼
    [Output .db file for Crosswork Planning]
```

## Command Line Interface

The script conforms to the executable script parameter conventions as defined in the [Cisco Crosswork Planning 7.2 Collection Setup and Administration](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-collectors-in-cp.html#run-external-scripts) documentation. When Crosswork Planning executes an external script, it provides command-line arguments in a predefined order:

| Argument | Description |
|----------|-------------|
| `argv[1]` | Source plan file (baseplan) |
| `argv[2]` | Output plan file (.db format) |
| `argv[3]` | Device access authentication file |
| `argv[4]` | Global network access configuration file |
| `argv[5]` | Home directory |
| `argv[6]` | Path where user uploaded external files are available |
| `argv[7]` | Path to access archive root directory |

The `get_plan_cp_cw.py` script uses `argv[1]` (ignored for startup scripts) and `argv[2]` (output plan file) to conform to this interface:

```bash
python get_plan_cp_cw.py <baseplan> <output_planfile> [options]
```

> **Note**: Since this script retrieves a complete plan file directly from Crosswork Network Controller, it does not use the base planfile (`argv[1]`), device access authentication file (`argv[3]`), or global network access configuration file (`argv[4]`) parameters. Only the **output plan file** (`argv[2]`) parameter is used to specify the name of the output file.

### Arguments

| Argument | Description |
|----------|-------------|
| `baseplan` | First parameter (base planfile) - ignored, required by CP framework |
| `planfile` | Output plan file name (typically `.db` format) |
| `--ip` | Crosswork Network Controller IP address *(optional, for testing only)* |
| `--username`, `-u` | CNC username *(optional, for testing only)* |
| `--password`, `-p` | CNC password *(optional, for testing only)* |
| `--version`, `-v` | Planfile version (default: `7.10`) |

> **Note**: The `--ip`, `--username`, `--password` and `--version` parameters are optional and intended for quick testing purposes only. For production deployments, configure these values as constants within the script or use environment variables.

### Example Usage

```bash
# As standalone script
python get_plan_cp_cw.py ignored output.db --ip 10.58.239.120 -u admin -p mypassword

# The script internally:
# 1. Downloads plan as planfile.pln
# 2. Converts planfile.pln → output.db using mate_convert
```

## Integration with Crosswork Planning Collector

### Configuration Steps

1. **Navigate to Collection Configuration**
   - In Crosswork Planning, create a new collection or edit an existing one

2. **Enable Startup Script**
   - In the **Startup script** section, select **Script**

3. **Upload the Script**
   - **Input file**: Upload `get_plan_cp_cw.py` (or as part of a `.zip` archive if dependencies are needed)
   - **Executable script**: `get_plan_cp_cw.py`
   - **Script language**: Python

4. **Configure Parameters**
   - Modify the hardcoded defaults in the script or pass arguments via the collection configuration

```
CROSSWORK_IP = "198.18.134.219"
CROSSWORK_USERNAME = "admin"
CROSSWORK_PASSWORD = "mypassword"
```

5. **Configure Downstream Collectors (Optional)**
   - Use the startup script output as **Source** for other collectors (LSP, BGP, VPN, etc.)
   - The IGP database or SR-PCE collector becomes optional when a startup script produces a valid network model

### Collection Chain Example

```
┌──────────────────────┐
│   Startup Script     │
│  get_plan_cp_cw.py   │
│  (produces .db)      │
└──────────┬───────────┘
           │ Source
           ▼
┌──────────────────────┐
│   LSP Collector      │
│  (optional)          │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│   Traffic Collector  │
│  (optional)          │
└──────────────────────┘
```

## Key Functions

### `get_auth_ticket(ip, username, password)`

Performs two-step SSO authentication:
1. Obtains TGT (Ticket Granting Ticket) from CNC SSO endpoint
2. Exchanges TGT for JWT token using service URL

### `get_plan(ip, ticket, format, version)`

Calls the CNC Optimization Engine REST API to retrieve the plan file:
- **Endpoint**: `/crosswork/nbi/optima/v2/restconf/operations/cisco-crosswork-optimization-engine-operations:get-plan`
- **Payload**: `{ "input": { "version": "<version>", "format": "<pln|txt>" } }`
- **Returns**: Base64-decoded plan file content

### `main()`

Orchestrates the workflow:
1. Parses command-line arguments
2. Authenticates to CNC
3. Retrieves plan file
4. Saves intermediate file (`.pln` or `.txt`)
5. Converts to `.db` format using `mate_convert`

## Configuration Constants

The script contains hardcoded defaults that should be modified for your environment:

```python
CROSSWORK_IP = "198.18.134.219"      # CNC IP address
CROSSWORK_USERNAME = "admin"         # CNC username  
CROSSWORK_PASSWORD = "mypassword"    # CNC password (update for production!)
TMP_PLANFILE = "planfile.pln"        # Intermediate file name
```

> **Security Note**: For production deployments, consider using environment variables or a secure credential store instead of hardcoded credentials.

## Dependencies

- **Python packages**: `requests`, `urllib3`, `argparse`, `base64`, `subprocess`
- **External tools**: `mate_convert` (Cisco WAE/Crosswork Planning utility)

## Error Handling

The script handles common error scenarios:
- **HTTP errors**: Authentication failures, API errors
- **Connection errors**: Network unreachability
- **File format errors**: Invalid planfile extensions

## Troubleshooting

For troubleshooting script execution issues, logs are available on the Crosswork Planning server at:

```
/mnt/cw_logfs/external-executor-service/1/external-executor.log
```

This log file contains output from the script execution, including any error messages or exceptions that may help diagnose issues with authentication, API calls, or file conversion.

## Relationship to Crosswork Planning Documentation

Per the Cisco documentation:

> "You can provide an external script as the initial step in a data collection chain. When enabled, the startup script is executed before any other collectors in the chain."

This script fulfills that role by providing the network model from CNC, enabling scenarios where:

- Network topology is already managed in Crosswork Network Controller
- You want to synchronize Crosswork Planning with the live CNC network state
- You prefer using CNC's optimization engine data rather than independent SNMP/IGP discovery

## Limitations and Considerations

1. **Single startup script**: Only one startup script per collection chain
2. **Database file requirement**: Downstream collectors fail if the script doesn't produce a valid `.db` file
3. **Credential management**: Hardcoded credentials should be externalized for security
4. **SSL verification**: Script disables SSL verification (`verify=False`) for self-signed certificates
5. **AAA Session Limits**: As a safeguard, it is preferred to use separate CNC credentials for get-plan. Under Admin > AAA Settings, No. of parallel sessions should be set orders higher than No. of parallel sessions per user (e.g. 200 vs 50). 
## References

- [Cisco Crosswork Planning 7.2 Collection Setup Guide - Startup Scripts](https://www.cisco.com/c/en/us/td/docs/cloud-systems-management/crosswork-planning/7-2/setup-guide/cisco-crosswork-planning-7-2-collection-setup-and-administration/m-collectors-in-cp.html#run-startup-script)
- [Cisco Crosswork Network Controller API Documentation](https://developer.cisco.com/docs/crosswork/)

---

*Document Version: 1.0*  
*Script: get_plan_cp_cw.py*  
*Platform: Cisco Crosswork Planning 7.2*
