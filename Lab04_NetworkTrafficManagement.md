# Lab 4 - Implement Network Traffic Management

## What This Lab Was About

This lab was about controlling how traffic flows to virtual machines inside Azure. In a real environment you never send users directly to a single server because if that server goes down everything goes down with it. Instead you put a layer in between that distributes traffic, checks if servers are healthy, and routes requests to the right place. This lab covered two Azure tools that do exactly that: Azure Load Balancer and Azure Application Gateway.

Note: The official lab uses three backend VMs. I completed this with two VMs due to free trial resource limits. The core traffic management concepts are identical.

## The Infrastructure I Built

I created a virtual network with separate subnets: one for the load balancer backend, one for the application gateway backend, and one for the application gateway itself. Subnets keep different layers of the architecture isolated from each other, which is standard practice.

I deployed two virtual machines running web servers as the backend. In production you would have more for higher availability but two is enough to see traffic distribution in action.

## Azure Load Balancer

The Load Balancer sits in front of the backend VMs and distributes incoming traffic between them at the network layer (Layer 4). It works at the TCP/UDP level, which means it does not look at the content of requests, just where they are coming from and where they need to go.

I configured a **backend pool** containing both VMs. Any traffic sent to the load balancer's IP gets distributed across the pool.

I set up a **health probe** that checks each VM on a regular interval. If a VM stops responding the load balancer automatically stops sending traffic to it. Once the VM recovers it gets added back. This is how you get fault tolerance without any manual intervention.

**Load balancing rules** tied the frontend IP to the backend pool and told the load balancer which port to forward traffic on.

## Azure Application Gateway

The Application Gateway is more advanced. It operates at Layer 7, which means it understands HTTP and HTTPS. Because it can read the content of requests it can make smarter routing decisions than a standard load balancer.

I configured:

- A **frontend IP** that receives incoming web traffic
- A **backend pool** pointing to the VMs
- **HTTP settings** that define how the gateway talks to the backend servers
- A **listener** that tells the gateway which port and protocol to accept traffic on
- **Routing rules** that connect listeners to backend pools

The Application Gateway also has built-in WAF (Web Application Firewall) capability, though I did not enable it in this lab. In a real environment you would turn it on to protect against common attacks.

## How Traffic Distribution Works

With the Load Balancer, requests get distributed in round-robin by default. Request 1 goes to VM1, request 2 goes to VM2, request 3 goes back to VM1, and so on. You can configure session persistence if you need a user to always hit the same server.

With the Application Gateway, you can go beyond round-robin. You can route based on URL paths, host headers, or other HTTP attributes. For example, requests to `/api` could go to one backend pool and requests to `/images` could go to another.

## Architecture

```
Internet
    │
    ▼
Azure Load Balancer (Layer 4 - TCP/UDP)
    │
    ├── VM1 (web server)
    └── VM2 (web server)

Internet
    │
    ▼
Azure Application Gateway (Layer 7 - HTTP/HTTPS)
    │
    ├── VM1 (web server)
    └── VM2 (web server)
```

## What I Learned

A Load Balancer is fast and lightweight but it has no awareness of what is inside a request. An Application Gateway is more powerful because it understands HTTP and can route based on content. In practice you would choose between them based on what kind of routing logic you need. Health probes are what make both tools self-healing, the system detects a failed server and reroutes traffic without anyone having to do it manually.

## Skills

`Azure Load Balancer` `Azure Application Gateway` `Backend Pools` `Health Probes` `Load Balancing Rules` `Layer 4 vs Layer 7` `Traffic Distribution` `Virtual Networks` `Azure Portal`
