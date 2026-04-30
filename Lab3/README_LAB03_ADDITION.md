# README Addition — Lab 03 Block

Copy and paste this block into your existing README.md after the Lab 02 section.

---

## Lab 03 — Implement Intersite Connectivity

**Skills:** VNet Peering · User Defined Routes · Network Watcher · Hub-and-Spoke Architecture

| Resource | Value |
|---|---|
| Resource Group | Lab05Az |
| Region | East US 2 |
| CoreServicesVnet | 10.0.0.0/16 |
| ManufacturingVnet | 172.16.0.0/16 |

**What I built:**
- Two isolated Azure VNets simulating segmented network environments (core IT and manufacturing)
- Proved VNet isolation using Network Watcher Connection Troubleshoot (Next hop: None)
- Connected VNets using bidirectional VNet Peering over Microsoft's private backbone
- Validated peering with PowerShell `Test-NetConnection` via Azure Run Command (TcpTestSucceeded: True)
- Created a perimeter subnet and User Defined Route to force traffic through a future NVA — the foundation of hub-and-spoke architecture

**Key concepts:** VNet isolation as default security posture · Non-overlapping address spaces for peering · UDRs override system routes · Route tables must be subnet-associated to take effect

📄 [Lab Write-Up](./LAB_03.md) | 📋 [Study Guide PDF](./AZ104_Lab05_StudyGuide.pdf)

---
