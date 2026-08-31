# 01 — Network Basics

Lab: Windows 11 VM (VMware NAT), Wireshark 4.6.8 on Ethernet0.
Capture: `dns-tcp-baseline.pcapng` — 338 packets, 31 Aug 2026.

## DNS resolution

| # | Source | Destination | Info |
|---|---|---|---|
| 18 | 192.168.81.128 | 192.168.81.2 | query 0xeabf A example.com |
| 19 | 192.168.81.2 | 192.168.81.128 | response 0xeabf A 172.66.147.243 |

Query and response are matched by the **Transaction ID** (`0xeabf`), not by
IP or port — several queries can be in flight at once.

`192.168.81.2` is not a real DNS server: it is the VMware NAT gateway.
The query leaves the network with the gateway's address, not mine.
This is why source IPs are unreliable in SOC investigations behind NAT.

Of 21 DNS packets captured, only 2 were mine. The rest was background
noise — Microsoft telemetry and six repeated `wpad.localdomain` lookups.

## TCP 3-way handshake

| # | Direction | Flags |
|---|---|---|
| 20 | me → 172.66.147.243 | SYN |
| 21 | server → me | SYN, ACK |
| 22 | me → server | ACK |

Total elapsed: 18 ms. The exchange proves both sides can send and receive,
and synchronises the initial sequence numbers.

Detection value: **SYN sent, no SYN-ACK returned = filtered or closed port.**
Port scan detection looks for exactly this — many SYNs, few completed
handshakes.

Teardown took four packets (FIN,ACK → ACK → FIN,ACK → ACK), not three:
each direction is closed separately.

## Tool error vs wire truth

`curl http://example.com` in PowerShell returned: