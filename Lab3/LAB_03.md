# Lab 03 — Implement Intersite Connectivity

**Tools:** Azure Portal · PowerShell · Network Watcher | **Region:** East US 2 | **RG:** Lab05Az

---

## What I Built

Two isolated Azure Virtual Networks simulating separate organizational segments — a core IT environment and a manufacturing environment — then connected them using VNet Peering and demonstrated how to override Azure's default routing with a User Defined Route to steer traffic through a dedicated perimeter. I used Network Watcher to prove isolation before peering and PowerShell via Azure Run Command to validate connectivity after.

---

## Virtual Networks

I created two VNets with non-overlapping address spaces.

**CoreServicesVnet (10.0.0.0/16)** represents a central IT hub. I gave it two subnets: the Core subnet (10.0.0.0/24) for the primary workload VM, and a Perimeter subnet (10.0.1.0/24) reserved for a future Network Virtual Appliance — the inspection point in a hub-and-spoke topology.

**ManufacturingVnet (172.16.0.0/16)** represents an isolated business unit spoke. It has a single Manufacturing subnet (172.16.0.0/24) with a Windows Server VM.

Choosing non-overlapping ranges isn't just a technical checkbox — in production you'd design your entire IP schema across VNets, regions, and on-premises ranges upfront to avoid re-addressing later.

---

## Proving Isolation with Network Watcher

Before configuring anything, I used Network Watcher's Connection Troubleshoot to run a baseline test between the two VMs. Every probe failed — 316 of 316 — with no route found between the VNets. The next hop came back as None.

This is Azure's default security posture: VNets are fully isolated from each other by design. Nothing crosses between them without explicit configuration. Documenting this failure first is important because it proves the peering is actually doing work — you're not inheriting some default connectivity that exists without you.

---

## VNet Peering

I connected both VNets with bidirectional peering. Azure requires two link objects — one in each direction — both must reach Synchronized status before traffic can flow.

Peering routes traffic over Microsoft's private backbone with no public internet exposure, no VPN gateway, and no encryption overhead. It's the lowest-latency path for connecting VNets in the same region.

The architectural constraint worth knowing: peering is non-transitive. If CoreServicesVnet peers with ManufacturingVnet, and ManufacturingVnet also peers with a third VNet, CoreServicesVnet cannot reach that third VNet through the chain. Solving transitive routing requires a hub NVA or Azure Firewall — which is exactly what the UDR in this lab sets up the foundation for.

---

## Validating Peering with PowerShell Run Command

After peering was configured, I ran a connectivity test directly inside ManufacturingVM using Azure Run Command — no RDP, no public IP, no Bastion host needed.

```powershell
Test-NetConnection 10.0.0.4 -port 3389
```

```
ComputerName     : 10.0.0.4
RemoteAddress    : 10.0.0.4
RemotePort       : 3389
SourceAddress    : 10.1.0.4
TcpTestSucceeded : True
```

`TcpTestSucceeded: True` confirmed that peering was functioning — traffic crossed from the Manufacturing subnet into CoreServicesVnet over the private backbone.

---

## User Defined Route — Controlling Traffic Flow

With peering working, I layered in a UDR to demonstrate how to override Azure's default routing. Azure automatically creates system routes based on address spaces — peering adds more. A UDR lets you override those system routes and redirect traffic where you want it to go.

I created route table `rt-CoreServices` with a single route:

| Setting | Value |
|---|---|
| Destination | 172.16.0.0/16 (ManufacturingVnet) |
| Next Hop Type | Virtual Appliance |
| Next Hop Address | 10.0.1.4 (Perimeter subnet — future NVA) |

I then associated the route table to the Core subnet. This means any traffic leaving CoreServicesVM destined for ManufacturingVnet gets redirected to 10.0.1.4 — the IP where a firewall or NVA would sit — instead of going directly. The NVA doesn't exist yet in this lab, but this is the exact pattern used in hub-and-spoke architectures to force east-west traffic through a centralized inspection point.

Route tables only take effect when associated to a subnet. The table itself does nothing until it's attached.

---

## Architecture

```
CoreServicesVnet (10.0.0.0/16)            ManufacturingVnet (172.16.0.0/16)
├── Core subnet (10.0.0.0/24)              └── Manufacturing subnet (172.16.0.0/24)
│   CoreServicesVM (10.0.0.4)                  ManufacturingVM (172.16.0.4)
│   ← rt-CoreServices UDR applied
│   172.16.0.0/16 → 10.0.1.4 (NVA)
│
└── Perimeter subnet (10.0.1.0/24)
    Future NVA at 10.0.1.4

          ↕ VNet Peering (bidirectional, both links Synchronized)
```

---

## Screenshots

| Description | Screenshot |
|---|---|
| Network Watcher baseline — connectivity failure before peering | ![Screenshot 1](Screenshot%20Lab3_1.png) |
| VNet Peering — both links Synchronized | ![Screenshot 2](Screenshot%20Lab3_2.png) |
| PowerShell Run Command — TcpTestSucceeded: True | ![Screenshot 3](Screenshot%20Lab3_3.png) |

---

## Key Concepts

- **VNet isolation is the default** — no connectivity between VNets without explicit configuration
- **Peering requires non-overlapping address spaces** — plan your IP schema before deployment
- **Peering is non-transitive** — multi-VNet routing requires NVA or Azure Firewall in the path
- **UDRs override system routes** — giving you control over where traffic goes next
- **Next hop: Virtual Appliance** redirects traffic through an NVA for inspection before forwarding
- **Route tables must be subnet-associated** to take effect
- **Azure Run Command** lets you execute scripts on VMs without RDP or a public IP

---

## Skills Demonstrated

`VNet Peering` `User Defined Routes` `Route Tables` `Network Watcher` `Hub-and-Spoke Architecture` `Network Segmentation` `IP Address Planning` `Azure Run Command` `PowerShell` `Intersite Connectivity`
