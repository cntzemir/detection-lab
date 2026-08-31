# Lab Setup

## Installed
- VMware Workstation Pro 26H1
- Windows 11 Pro (x64) VM
- Sysmon v15.21

## Verified
- Sysmon64 service is running
- Events are flowing into the Sysmon/Operational log
- Logs looked stale at first; the cause was the VM being set to the Pacific
  timezone while the host is on CEST. Timestamps are stored in UTC and rendered
  in local time, so the 9-hour gap was a display issue, not a telemetry failure.