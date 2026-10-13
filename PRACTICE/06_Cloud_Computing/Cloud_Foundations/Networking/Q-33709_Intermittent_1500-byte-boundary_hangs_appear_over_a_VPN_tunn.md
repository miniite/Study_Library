---
type: "practice"
difficulty: "Expert"
---

# Intermittent 1500-byte-boundary hangs appear over a VPN/tunnel. Explain MTU/MSS mismatch and PMTUD blackholing, and how you'd fix it.

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ TCP and Connection Tracking](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/TCP_and_Connection_Tracking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with intermittent network issues where packets of 1500 bytes are experiencing hangs over a Virtual Private Network (VPN). The root cause can be traced back to an MTU/MSS mismatch and possible Path Maximum Transmission Unit (PMTUD) blackholing. This situation often arises when the Maximum Transmission Unit (MTU) of the tunnel is smaller than that of the underlying network, leading to fragmented packets that get stuck or dropped.

#### Triaging the Question

The question primarily seeks an understanding of:

MTU and MSS Mismatch: How the size of data packets being transmitted over a network can conflict with the capabilities of the network infrastructure.
PMTUD Blackholing: A situation where packet fragmentation causes issues due to incorrect MTU settings.

#### Assumptions
The network is using IPv4.
The issue only occurs for 1500-byte packets, suggesting that smaller or larger packets do not cause problems.
The use of a VPN/tunnel introduces an additional layer of complexity in handling packet sizes and fragmentation.

#### Solution
from First Principles
Understanding MTU and MSS

MTU stands for Maximum Transmission Unit, which is the largest size (in bytes) that can be sent over a network without requiring reassembly. MSS (Maximum Segment Size) is related to TCP and refers to the maximum size of data within a single TCP segment.

When a packet larger than the MTU of an interface needs to traverse it, the packet must be fragmented. If there's a mismatch between the sender’s and receiver’s MTUs or MSSs, fragmentation can fail at some point in the network path, causing packets to get stuck or dropped.

PMTUD Blackholing

Path MTU Discovery (PMTUD) is designed to find the smallest Maximum Transmission Unit along any given link in a network. However, if PMTUD fails due to misconfigured firewalls or other network devices, it can lead to "blackholing" where packets are neither delivered nor discarded properly.

To diagnose and resolve this issue:

Verify MTU Settings: Ensure that the MTU settings on both ends of the tunnel and within the network are correctly configured.
Enable PMTUD: Make sure PMTUD is enabled to dynamically discover the path MTU.
Firewall Configuration: Check for any firewall rules or devices that might be causing fragmentation issues.
Steps to Fix the Issue

Check Tunnel and Network MTUs:

BASH
ip link show <interface>

This command will display the current MTU settings on your interfaces.

Enable PMTUD on Both Ends: Ensure that both the sender and receiver systems have PMTUD enabled in their operating system configurations.

Adjust Firewall Rules: Look for any firewall or security devices between the endpoints that might be causing issues with packet fragmentation. Adjusting these rules to allow fragmented packets can resolve blackholing.

Use Tools for Diagnosis: Use tools like ping and traceroute to identify where in the path fragmentation is failing.

BASH
ping -M do -s 1500 <target_ip>
traceroute -f 8 -p 80 <target_host>

The -M do option in ping disables path MTU discovery, which can help identify if the issue is related to PMTUD.

#### Example Scenario

Suppose you have a network with a default MTU of 1500 bytes. A packet larger than this size would need to be fragmented. However, there's a firewall between two endpoints that misconfigures and drops these fragments, leading to blackholing.

To fix it:

Adjust the Firewall: Ensure the firewall allows fragmentation by not dropping small packets.
Set MTU to 1472 Bytes (commonly used for tunnels):
BASH
sudo ifconfig <interface> mtu 1472
Verify PMTUD:
BASH
ping -M do -s 1500 <target_ip>

#### Closing Summary
and Common Mistakes/Follow-Ups

The key to resolving this issue lies in correctly configuring the MTU settings, ensuring that PMTUD is functioning properly, and verifying that there are no firewalls or other network devices blocking fragmented packets. Common mistakes include ignoring firewall configurations or not properly enabling PMTUD.

#### Follow-up
questions might explore deeper aspects such as:

How different operating systems handle fragmentation.
The impact of TCP window size on performance in such scenarios.
Using alternative protocols like VXLAN to mitigate MTU issues.

**Comparison:**

```
Command	Description
ip link show	Shows the MTU settings of the network interface.
ping -M do -s	Disables path MTU discovery and sends a packet with a specified size, testing fragmentation.
```

**Code:**

```bash
# Example commands for troubleshooting and fixing the issue
ip link show <interface>
ping -M do -s 1500 <target_ip>
traceroute -f 8 -p 80 <target_host>
```

**Example input / output:**

```text
Example:
- **Input**: `ping -M do -s 1500 192.168.1.1`
- **Output**: Packet fragmentation fails and blackholing occurs, resulting in no response.
```

**Data:**

```json
{
  "interface": "<network_interface>",
  "mtu": 1472,
  "packet_size": 1500
}
```

**Flow:**

1. Yes
2. No
3. Identify MTU/MSS Mismatch
4. Enable PMTUD?
5. Adjust Firewall Rules
6. Verify MTU Settings
7. Fix Issue
8. Adjust MTU Settings
9. Confirm Resolution

