# Lab 2 - Virtual Networking in Azure

## What This Lab Was About

This lab was about building a network inside Azure from scratch and making sure it was set up securely. In the real world companies split their cloud environments into separate networks so different teams or systems do not have access to each other by default. I built that same kind of setup here, connected the networks together, locked down the traffic, and set up DNS so machines can find each other by name instead of just IP address.

## The Networks I Created

I built two separate virtual networks with different IP address ranges. They have to use different ranges because Azure will not let you connect two networks if their addresses overlap. It is like trying to have two streets with the exact same address in the same city.

**CoreServicesVnet (10.20.0.0/16)** was the main network. I split it into three sections called subnets. One for a future VPN gateway, one for shared services, and one specifically for databases to keep that data separated from everything else.

**ManufacturingVnet (10.30.0.0/16)** was a second network representing a separate business unit. It had one subnet for the manufacturing team's systems.

## Connecting the Networks

I connected both networks using VNet Peering. Once peered, machines in one network can talk to machines in the other over Microsoft's private network, not the public internet. It is fast, private, and does not require any extra hardware or a VPN.

One thing worth knowing is that peering only works between the two networks you connect directly. If Network A is connected to Network B, and Network B is connected to Network C, Network A still cannot reach Network C. Each connection has to be set up explicitly.

## Security Rules

I created a Network Security Group called `myNSGSecure` and attached it to the shared services subnet to control what traffic was allowed in and out.

Instead of writing rules based on specific IP addresses I used something called an Application Security Group. I created one called `asg-web` and assigned VMs to it. That way the security rules apply to the group, not individual IPs. If I add more servers later I just add them to the group and the rules automatically apply. No need to update anything else.

For inbound traffic I allowed web traffic on ports 80 and 443 from that group. For outbound I added a rule that blocks all traffic going out to the internet. Azure allows internet access by default so I had to explicitly override that.

## Public DNS

I set up a public DNS zone for `az104contoso.com` and added a record pointing `www` to an internal IP address. DNS is basically the system that translates a web address like `www.example.com` into an actual IP address a computer can connect to. Setting this up in Azure means Azure is handling that translation for the domain.

## Private DNS

I also set up a private DNS zone called `private.az104contoso.com` and linked it to both networks. With auto registration turned on, any virtual machine that gets created inside either network automatically gets its own DNS record. When the machine is deleted the record goes away too. This means machines can reach each other by name without anyone having to manually update a list.

## Architecture

```
CoreServicesVnet (10.20.0.0/16)          ManufacturingVnet (10.30.0.0/16)
├── GatewaySubnet (10.20.0.0/27)          └── ManufacturingSystemsSubnet
├── SharedServicesSubnet (NSG applied)
└── DatabaseSubnet
          connected via VNet Peering

Public DNS:  az104contoso.com  www points to  10.20.0.4
Private DNS: private.az104contoso.com
             linked to CoreServicesVnet (auto registration on)
             linked to ManufacturingVnet (auto registration on)
```

## What I Learned

Keeping networks separated is a basic security practice in cloud environments. Peering lets you connect them in a controlled way without putting anything on the public internet. Security groups are much easier to manage than IP-based rules when your environment is growing. Private DNS makes it so machines can find each other by name automatically without anyone having to manage it manually.

## Skills

`Virtual Networks` `VNet Peering` `NSG` `Application Security Groups` `Azure DNS` `Private DNS` `Network Segmentation` `Azure Portal`
