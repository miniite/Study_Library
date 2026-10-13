---
type: "practice"
difficulty: "Basic"
---

# What is a default gateway and what role does it play when a host talks to another network?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Network Devices and Layers](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/Network_Devices_and_Layers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In simple terms, a default gateway acts as a bridge between your local network and external networks, such as the internet. When a host in one network needs to communicate with a device on another network, it uses the default gateway to route its traffic. Let's break down this concept from first principles.

#### Triage
What is Being Asked?

The question asks for an explanation of what a default gateway is and how it functions when a host communicates across networks. The interviewer might be probing your understanding of networking fundamentals, specifically focusing on routing and network connectivity.

#### Assumptions
We are discussing a typical home or small office network where there is a local network (subnet) and the internet.
The default gateway is typically a router that acts as the primary point of contact for all external traffic.
Detailed Solution:

A default gateway, also known as a "gateway IP" or simply "gateway," is a special kind of address assigned to a router within your network. When data needs to be sent out beyond the local network (e.g., from one subnet to another), it uses the default gateway for routing. Here’s how it works:

IP Addresses and Subnets: Every device on a network has an IP address. These addresses are divided into subnets, which logically group devices together. A router separates these subnets.
Routing Decisions: When a host wants to send data to a destination outside its local subnet (e.g., another subnet or the internet), it needs help in figuring out where to forward that packet. The default gateway is configured on each device as part of the routing table, which tells packets where to go next.
Packet Forwarding: When a host sends a packet destined for an IP address outside its local network, the packet’s destination address is compared against the entries in the routing table. If there's no specific route defined, it defaults to the default gateway.

#### Example Scenario

Imagine you have two subnets: Subnet A (192.168.1.0/24) and Subnet B (192.168.2.0/24). The devices in these subnets are connected through a router with the IP address 192.168.1.1. If a device in Subnet A tries to send a packet to an IP address in Subnet B, it will use 192.168.1.1 as the default gateway.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, a default gateway is essential for directing traffic between subnets and connecting them to external networks. Without proper configuration of the default gateway, devices would be unable to communicate outside their local network. Common mistakes include misconfiguring the IP address or forgetting to set up a route to the gateway.

#### Common Mistakes
Misconfigured Default Gateway: If the wrong IP is set as the default gateway, packets may not be routed correctly.
Missing Route Entry: If no specific routes are defined for external networks, traffic might fail to reach its destination.
Insufficient Permissions: In some cases, incorrect permissions on the router can prevent packet forwarding.

#### Follow-Up
Questions:
How do you diagnose and resolve issues with a default gateway?
Can a host communicate directly with an external network without a default gateway? Why or why not?

**Comparison:**

```
IP Address	Subnet Mask	Gateway
192.168.1.5	255.255.255.0	192.168.1.1
192.168.2.10	255.255.255.0	192.168.1.1
```

**Code:**

```bash
# Example of setting up a default gateway in Linux
sudo route add -net 192.168.2.0 netmask 255.255.255.0 gw 192.168.1.1
```

**Example input / output:**

```text
Host in Subnet A (e.g., 192.168.1.5) sends a packet to 192.168.2.10.
- Packet's destination is outside local subnet, so it checks routing table.
- Route entry found for 192.168.2.x/24 via 192.168.1.1 (default gateway).
- Packet forwarded to 192.168.1.1.
```

**Data:**

```json
{
  "subnetA": {
    "ipRange": "192.168.1.0/24",
    "defaultGateway": "192.168.1.1"
  },
  "subnetB": {
    "ipRange": "192.168.2.0/24",
    "defaultGateway": "192.168.1.1"
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Host in Subnet A
5. Is destination within localsubnet?
6. Forward packet to defaultgateway
7. Routable via default gateway?
8. Packet forwarded todestination
9. Packet dropped or redirected

