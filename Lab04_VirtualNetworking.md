# Lab 04 — Virtual Networking in Azure

**Tools:** Azure Portal | **Region:** East US 2 | **RG:** az104-rg4

---

## What I Built

A multi-VNet network environment in Azure simulating a segmented enterprise topology. I deployed two Virtual Networks with defined subnet structures, connected them via peering, locked down traffic using NSGs and Application Security Groups, and configured both public and private DNS zones for name resolution.

---

## Virtual Networks

I created two VNets with non-overlapping address spaces — a requirement for peering and a real-world constraint when designing at org scale.

**CoreServicesVnet (10.20.0.0/16)** — the hub network representing shared infrastructure. I subdivided it into three subnets: GatewaySubnet (10.20.0.0/27) reserved for future VPN/ExpressRoute gateway deployment, SharedServicesSubnet (10.20.10.0/24) for shared workloads, and DatabaseSubnet (10.20.20.0/24) for data tier isolation.

**ManufacturingVnet (10.30.0.0/16)** — a spoke network representing a business unit, with ManufacturingSystemsSubnet (10.30.10.0/24).

Keeping address spaces non-overlapping isn't just a technical requirement — it's a design discipline. In production you'd plan your entire IP schema across VNets, regions, and on-premises ranges before deploying anything to avoid costly re-addressing later.

---

## VNet Peering

I connected both VNets via bidirectional peering, allowing resources in either VNet to communicate over private IPs using Microsoft's backbone — no public internet, no gateways, no encryption overhead.

The key architectural detail here is that peering is **non-transitive**. If CoreServicesVnet peers with ManufacturingVnet, and ManufacturingVnet peers with a third VNet, CoreServicesVnet cannot reach that third VNet through the chain. Solving transitive routing requires either Azure Firewall/NVA in a hub-and-spoke design or Azure Virtual WAN at scale — a distinction that comes up often in cloud architecture discussions.

---

## Network Security — NSG & ASGs

I created NSG `myNSGSecure` and attached it to SharedServicesSubnet. Rather than writing IP-based rules, I used an **Application Security Group** (`asg-web`) to control access by workload role.

**Inbound:** Allowed ports 80/443 from `asg-web` — any VM assigned to that ASG can receive web traffic without touching the NSG rule itself as the environment scales.

**Outbound:** Added `DenyInternetOutbound` to explicitly block all outbound internet traffic from the subnet. Azure's default (priority 65001) allows outbound internet — this rule overrides it at a lower priority number and enforces a zero-trust posture where internet egress must be explicitly approved.

The reason I used an ASG instead of source IPs: if I add five more web servers tomorrow, I assign them to `asg-web` at the NIC level. The security rule doesn't change. This is the scalable pattern for managing security policy in dynamic environments.

---

## DNS — Public Zone

I created a Public DNS Zone for `az104contoso.com` and added a `www` A record pointing to `10.20.0.4`. This makes Azure the authoritative nameserver for the domain — in production you'd update your registrar's NS records to delegate to Azure's nameservers to complete the chain.

One operational detail worth noting: the 1-hour TTL (3600s) means resolvers cache the answer for an hour. In a real migration scenario, you'd lower TTL well before the cutover so DNS changes propagate quickly, then raise it back after stabilizing.

---

## DNS — Private Zone

I created a Private DNS Zone (`private.az104contoso.com`) and linked it to both CoreServicesVnet and ManufacturingVnet with **auto registration enabled** on both links.

Auto registration means any VM deployed into either VNet automatically gets an A record created in this zone using its hostname — and the record is removed when the VM is deleted. No manual DNS management as the environment changes.

The limit worth knowing: a VNet can only have auto registration enabled for **one** private DNS zone at a time. You can link to multiple zones, but only one gets auto reg.

The bigger picture is what private DNS enables: internal service discovery by name instead of hardcoded IPs. In zero-trust GovCon environments, you'd never expose internal workload names publicly — private DNS keeps all resolution scoped to the VNet boundary.

---

## Architecture

```
CoreServicesVnet (10.20.0.0/16)          ManufacturingVnet (10.30.0.0/16)
├── GatewaySubnet (10.20.0.0/27)          └── ManufacturingSystemsSubnet
├── SharedServicesSubnet ← myNSGSecure
└── DatabaseSubnet
          ↕ VNet Peering (bidirectional)

Public DNS:  az104contoso.com → www A → 10.20.0.4
Private DNS: private.az104contoso.com
             ├── CoreServicesVnetLink (auto reg ON)
             └── ManufacturingVnetLink (auto reg ON)
```
