# Lab 3 - Implement Intersite Connectivity

## What This Lab Was About

This lab was about connecting two completely separate Azure networks and then controlling exactly how traffic flows between them. I also had to prove the networks could not talk to each other before I connected them, and prove they could after. On top of that I set up a custom route to redirect traffic through a specific path instead of letting it go directly.

## The Networks I Set Up

I created two virtual networks that represent two separate parts of a company.

**CoreServicesVnet (10.0.0.0/16)** was the main IT network. It had two subnets. One called Core for the main VM and one called Perimeter that I set aside for a future firewall or network appliance.

**ManufacturingVnet (172.16.0.0/16)** was a separate network for the manufacturing side. It had one subnet with its own Windows Server VM.

The two networks were completely isolated from each other to start. That is actually the default in Azure. Nothing can cross between networks unless you set it up yourself.

## Proving They Could Not Talk

Before connecting anything I ran a connectivity test using Azure Network Watcher. I tested whether the VM in the manufacturing network could reach the VM in the core network. Every single probe failed. 316 out of 316. Azure showed the next hop as None, meaning there was no path at all between the two networks.

I did this first on purpose. If you skip this step you have no way of knowing whether your connection setup actually did anything or if the traffic was already getting through some other way.

## Connecting the Networks

I connected both networks using VNet Peering. Azure needs two connections set up, one from each side, and both have to show as Synchronized before traffic can flow. Once that was done the networks could communicate with each other over Microsoft's private network without going through the public internet.

## Testing That It Worked

After setting up peering I tested the connection by running a PowerShell command directly inside the manufacturing VM using Azure Run Command. I did not need to remote into the machine or open any ports. Azure ran the script for me right from the portal.

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

It came back as True, which confirmed the peering was working and traffic was crossing between the two networks.

## Setting Up a Custom Route

The last part was creating a User Defined Route (UDR). By default Azure decides how traffic gets from one place to another on its own. A UDR lets you override that and say exactly where you want traffic to go.

I created a route table called `rt-CoreServices` with one route that says any traffic headed toward the manufacturing network should first go through the perimeter subnet at IP address `10.0.1.4`. That address is where a firewall or network appliance would sit in a real environment. In this lab the appliance is not actually there yet, but the routing is in place so that when it is added traffic will pass through it automatically for inspection.

| Setting | Value |
|---|---|
| Destination | 172.16.0.0/16 (ManufacturingVnet) |
| Next Hop | Virtual Appliance at 10.0.1.4 |

I attached the route table to the Core subnet to make it active. A route table sitting on its own does nothing until you attach it to a subnet.

## Architecture

```
CoreServicesVnet (10.0.0.0/16)            ManufacturingVnet (172.16.0.0/16)
├── Core subnet (10.0.0.0/24)              └── Manufacturing subnet (172.16.0.0/24)
│   CoreServicesVM (10.0.0.4)                  ManufacturingVM (172.16.0.4)
│   custom route applied here
│   traffic to 172.16.0.0/16 redirected to 10.0.1.4
│
└── Perimeter subnet (10.0.1.0/24)
    future firewall sits here at 10.0.1.4

        connected via VNet Peering (both sides synchronized)
```

## Screenshots

| What It Shows | Screenshot |
|---|---|
| Network Watcher test showing no connection before peering | ![Screenshot 1](Screenshot%20Lab3_1.png) |
| VNet Peering with both links showing Synchronized | ![Screenshot 2](Screenshot%20Lab3_2.png) |
| PowerShell test confirming connection worked | ![Screenshot 3](Screenshot%20Lab3_3.png) |

## What I Learned

Azure networks are isolated by default and nothing connects between them unless you set it up. Peering lets you connect networks over Microsoft's private backbone without needing a VPN. Custom routes let you control exactly where traffic goes instead of letting Azure decide. And Azure Run Command is a useful way to run commands inside a VM without having to open remote desktop or a public IP.

## Skills

`VNet Peering` `User Defined Routes` `Route Tables` `Network Watcher` `Network Segmentation` `Azure Run Command` `PowerShell` `Intersite Connectivity`
