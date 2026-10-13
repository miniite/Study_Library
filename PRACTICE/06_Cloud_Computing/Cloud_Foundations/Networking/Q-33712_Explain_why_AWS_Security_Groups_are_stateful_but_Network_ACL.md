---
type: "practice"
difficulty: "Deep"
---

# Explain why AWS Security Groups are stateful but Network ACLs are stateless, and give a concrete bug that this difference causes with ephemeral ports.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ mTLS and Zero Trust](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/mTLS_and_Zero_Trust.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

AWS Security Groups and Network ACLs are both critical components of network security in Amazon Web Services (AWS). However, they operate differently and serve distinct purposes within your VPC infrastructure. Understanding their behavior, especially the difference between stateful vs. stateless operations, is crucial for effective network management.

In this question, we focus on why AWS Security Groups are stateful while Network ACLs are stateless, and we explore a concrete example of how this difference can lead to a bug involving ephemeral ports. Let's break it down step by step from first principles.

#### Triage

The core issue here is the difference in state management between AWS Security Groups and Network ACLs. Specifically:

Security Groups: Are stateful, meaning they keep track of established connections.
Network ACLs: Are stateless, meaning they do not maintain any state information about connections.

To understand this better, we need to consider how these components handle traffic in a VPC environment.

#### First Principles Solution

Statefulness and Statelessness Defined:

Stateful (Security Groups): When a connection is established from the outside world to an internal resource, Security Groups remember that connection. They then allow subsequent packets of that same connection.
Stateless (Network ACLs): Network ACLs do not maintain any state information about connections. Every packet is evaluated independently based on the rules defined in the ACL.

Ephemeral Ports:

Ephemeral ports are dynamically assigned by the operating system and typically used for protocols like TCP, where a new connection needs to be established. These ports can change over time but always remain within a certain range.

#### Example
Bug with Ephemeral Ports:

#### Cons
ider an internal application that is using a web server (HTTP/HTTPS) on port 80 and 443 respectively. The Security Group rules allow incoming traffic from the internet to these ports.
Now, imagine a client initiates a TCP connection from the internet to one of your instances within the VPC. This connection uses an ephemeral port on the client side.

Here is how statefulness and statelessness affect this scenario:

Security Groups:

After the initial connection from the client's ephemeral port to, say, port 80 (HTTP), Security Groups will remember that established connection.
Subsequent packets for the same connection (e.g., ACKs, data, etc.) are allowed through.

Network ACLs:

The Network ACL does not know about the initial connection. Each packet from the client's ephemeral port is evaluated independently based on the rules defined in the ACL.
If the Network ACL allows only a specific range of ports (e.g., 80 and 443) for inbound traffic, it will drop any packets that are not within these well-defined ports.

#### Closing Summary

The key difference between AWS Security Groups and Network ACLs is their state management. Security Groups are designed to be more dynamic and context-aware, remembering established connections. In contrast, Network ACLs treat each packet independently without maintaining any connection state, making them less flexible for certain use cases involving ephemeral ports.

A concrete bug that can arise from this difference involves a scenario where packets from an internal resource using ephemeral ports are dropped by the Network ACL because they don't match the established connections remembered by Security Groups. This discrepancy in behavior can lead to intermittent connectivity issues or complete disconnection of services relying on dynamic port usage.

#### Common Mistakes
and Follow-Ups
Misunderstanding Statefulness: Failing to recognize that Security Groups are stateful and Network ACLs are not can lead to incorrect configuration decisions.
Over-reliance on Network ACLs: Using only Network ACLs for security might miss out on the dynamic nature of certain application flows, leading to potential vulnerabilities or disconnections.
Debugging Challenges: Diagnosing issues related to ephemeral port usage between stateful and stateless components can be complex due to the lack of connection context in Network ACLs.

For a deeper dive into these concepts, you might explore additional scenarios involving specific protocols, custom TCP flows, or multi-tiered architectures.

**Comparison:**

```
Component	Type	Stateful?	Example Scenario
Security Groups	Stateful	Yes	TCP connection from client to instance on port 80, subsequent packets allowed
Network ACLs	Stateless	No	TCP connection from client to instance on ephemeral port, packet dropped if not in range of defined ports
```

**Code:**

```python
# Example code for simulating packet handling with Security Groups and Network ACLs
def handle_packet(packet):
    # Simulate Security Group behavior (stateful)
    if is_established_connection(packet):
        allow_packet(packet)
    else:
        drop_packet(packet)

def handle_packet_network_acl(packet):
    # Simulate Network ACL behavior (stateless)
    if packet_is_in_range(packet, allowed_ports):
        allow_packet(packet)
    else:
        drop_packet(packet)
```

**Example input / output:**

```text
Input (Security Group):
- Initial packet: TCP from external client to internal instance on port 80
- Subsequent packets: ACKs and data

Output (Security Group):
- Initial packet allowed
- Subsequent packets allowed

Input (Network ACL):
- Initial packet: TCP from external client to internal instance on port 80
- Subsequent packets: ACKs and data

Output (Network ACL):
- Initial packet allowed
- Subsequent packets dropped if not in range of defined ports (e.g., ephemeral ports)
```

**Data:**

```json
{
  "SecurityGroupRules": [
    {"port": 80, "protocol": "tcp"},
    {"port": 443, "protocol": "tcp"}
  ],
  "NetworkACLRules": {
    "inbound": [
      {"port": 22, "protocol": "tcp"},
      {"port": 80, "protocol": "tcp"},
      {"port": 443, "protocol": "tcp"}
    ],
    "outbound": []
  }
}
```

**Flow:**

1. Network ACL
2. Security Group
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Initial Packet
10. Is Established Connection?
11. Allow Packet
12. Drop Packet
13. Subsequent Packet
14. Is Within Defined Ports?
15. Allow Packet
16. Drop Packet
17. Subsequent Packets Allowed
18. Subsequent Packets NotAllowed
19. Is Within Defined Ports?
20. Allow Packet
21. Drop Packet
22. Subsequent Packets Allowed
23. Subsequent Packets NotAllowed

