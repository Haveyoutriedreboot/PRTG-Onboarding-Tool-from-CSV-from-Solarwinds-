# PRTG-Onboarding-Tool-from-CSV-from-Solarwinds-
Bulk device onboarding for PRTG Network Monitor via CSV, built on PrtgAPI.

A PowerShell GUI tool for bulk onboarding devices into PRTG Network Monitor using a CSV file. Instead of adding devices one by one through the PRTG web interface, you fill in a CSV with your device names, hosts, groups, credentials, and sensors, then run this tool to create everything automatically.

The tool handles the full onboarding flow: creating missing groups, creating devices, setting SNMP (v1, v2c, v3) or WMI credentials, and adding sensors, including SNMP Traffic sensors with automatic interface discovery and optional interface filtering. It is built on top of [PrtgAPI](https://github.com/lordmilko/PrtgAPI) by lordmilko, which provides the underlying PowerShell cmdlets for talking to PRTG.

## What is included in this repository

Both the PowerShell source (`PRTG-Onboard-GUI.ps1`) and the compiled executable (`PRTG-Onboard.exe`) are published here.

The `.exe` is provided for convenience, so anyone can download and run it immediately without needing to install PowerShell modules for compiling or worry about execution policy for the script itself, beyond unblocking the downloaded file.

The `.ps1` source is provided alongside it so the tool is never a black box. If you want to inspect exactly what it does before running it, change any behavior, add support for additional sensor types, adjust the credential handling, or fix something for your own environment, you can edit the script directly and compile your own `.exe` from it. Instructions for compiling are included further down in this README.

This also means you are not dependent on this repository being updated. If PRTG changes a sensor type name, or you need a feature this version does not have, you can make the change yourself and keep using the tool without waiting on anyone else.

## Features

- Reads device definitions from a CSV file
- Creates missing PRTG groups automatically (defaults to "Unknown" if no group is specified)
- Creates devices and skips ones that already exist (configurable behavior)
- Sets SNMP (v1, v2c, v3) or WMI credentials per device
- Adds standard sensors (ping, CPU, memory, volume) with automatic correction of outdated sensor type names
- Adds SNMP Traffic sensors with automatic interface discovery and optional interface filtering
- Live scrolling log inside the app, with a saved log file per run
- Warning summary at the end of each run, separate from blocking errors
- Cancel button to stop a run safely at the next checkpoint
- Auto-installs the PrtgAPI PowerShell module if it is not already present on the machine

## Requirements

Before running this tool, the machine needs the following.

### PowerShell

Windows PowerShell 5.1 or newer. This comes preinstalled on Windows 10 and Windows 11. Check your version with:

```powershell
$PSVersionTable.PSVersion
```

### PrtgAPI module

The tool checks for this automatically and installs it if missing. If you want to install it yourself ahead of time:

```powershell
Install-Module PrtgAPI -Scope CurrentUser -Force
```

### Execution policy

Running scripts or the compiled .exe may require an adjusted execution policy, since both are unsigned by default. Run PowerShell as Administrator and set:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

This allows locally created or downloaded scripts to run, while still requiring a valid signature for scripts from the internet that have not been unblocked.

### Unblocking downloaded files

Files downloaded from GitHub are flagged by Windows as coming from the internet. Unblock them before running:

```powershell
Unblock-File -Path "C:\path\to\PRTG-Onboard-GUI.ps1"
Unblock-File -Path "C:\path\to\PRTG-Onboard.exe"
```

Alternatively, right click the file, choose Properties, and check the "Unblock" box at the bottom of the General tab.

### PRTG account permissions

The PRTG account used to connect needs permissions to:

- Create and edit groups
- Create and edit devices
- Create and edit sensors
- Read and write object properties (for setting credentials)

A full administrator account is the simplest option. If using a restricted account, it must have write access at the probe level where new groups and devices will be created.

### Network access

The machine running the tool needs:

- Network access to the PRTG core server (default ports 80 or 443 for the API)
- If onboarding SNMP devices, the PRTG probe needs network access to those devices on port 161 (UDP)
- If onboarding WMI devices, the PRTG probe needs the appropriate Windows firewall rules and DCOM/WMI access to the target machines

## Installation

1. Download `PRTG-Onboard-GUI.ps1` and `PRTG-Onboard.exe` from this repository
2. Unblock both files (see above)
3. Run the .exe directly, or run the .ps1 with:

```powershell
.\PRTG-Onboard-GUI.ps1
```

## CSV file format

```csv
Name,Host,Group,Sensors,CredentialType,Community,WMIUser,WMIPassword,SNMPVersion,TrafficInterfaces,SNMPv3User,SNMPv3AuthMethod,SNMPv3AuthPassword,SNMPv3PrivMethod,SNMPv3PrivPassword,SNMPv3SecurityLevel
Router-01,192.168.1.1,Network Devices,"ping|snmptraffic|snmpcpu|snmpmemory",SNMP,public,,,v2c,"001|003",,,,,,
Server-01,server01.domain.local,Windows Servers,"ping|wmiprocessor|wmimemory|wmivolume",WMI,,administrator,MyPassword,,,,,,,,
SecureRouter-01,192.168.1.2,Network Devices,"ping|snmptraffic",SNMP,,,,v3,,snmpuser,SHA1,authpass123,AES,privpass123,authPriv
```

| Column | Description |
|---|---|
| Name | Device name as it will appear in PRTG |
| Host | IP address or hostname |
| Group | PRTG group to place the device in, created automatically if missing |
| Sensors | Sensors to create, separated by the pipe character |
| CredentialType | SNMP, WMI, ICMP, or Agent |
| Community | SNMP v1/v2c community string |
| WMIUser | Windows username for WMI devices |
| WMIPassword | Windows password for WMI devices |
| SNMPVersion | v1, v2c, or v3 |
| TrafficInterfaces | Interface numbers for SNMP Traffic, separated by pipe, blank means all interfaces |
| SNMPv3User | SNMP v3 username |
| SNMPv3AuthMethod | SNMP v3 authentication protocol, MD5 or SHA1 |
| SNMPv3AuthPassword | SNMP v3 authentication password |
| SNMPv3PrivMethod | SNMP v3 privacy protocol, DES or AES |
| SNMPv3PrivPassword | SNMP v3 privacy password |
| SNMPv3SecurityLevel | noAuthNoPriv, authNoPriv, or authPriv |

## Building the .exe yourself

If you prefer to compile the .exe from source rather than using the one in this repo:

```powershell
Install-Module ps2exe -Scope CurrentUser -Force
Invoke-PS2EXE "PRTG-Onboard-GUI.ps1" "PRTG-Onboard.exe" -noConsole -title "PRTG Onboarding Tool"
```

## Security notes

- CSV files may contain SNMP community strings and WMI credentials in plain text. Do not commit real device CSVs to a public repository.
- The .gitignore in this repo excludes common CSV filename patterns to help prevent accidental credential leaks.
- WMI passwords are sent to PRTG over the API connection. Confirm your PRTG server connection uses HTTPS in production environments.

## License

This project is licensed under the MIT License. See the LICENSE file for details.

This tool depends on [PrtgAPI](https://github.com/lordmilko/PrtgAPI), which has its own license terms. Check that repository for details.
