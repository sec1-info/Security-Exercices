### **Exercice 02: Network and ARP Spoofing**

#### **1. What is the role of the ARP protocol? Explain how ARP tables are updated?**

**Role of ARP**: The Address Resolution Protocol (ARP) maps IP addresses to MAC addresses in a local network. When a device needs to send a packet to an IP address, it uses ARP to find the corresponding MAC address.

**How ARP tables are updated**:
- A device sends an **ARP Request** (broadcast) asking, “Who has IP X.X.X.X? Tell me your MAC address.”
- The device with the requested IP responds with an **ARP Reply** containing its MAC address.
- The sender updates its **ARP table** (a cache of IP-to-MAC mappings) with this pair.
- ARP tables are also updated when a device receives unsolicited ARP replies (e.g., gratuitous ARP) or during ARP spoofing attacks.
- Entries expire after a timeout (e.g., 20 minutes) to ensure freshness.

#### **2. What frames can machine C sniff? Explain.**

Machine C can sniff **all frames** on the network segment connected to interface `eth0` of router R1, assuming:
- The network uses a **shared medium** (e.g., a hub) or C has enabled **promiscuous mode** on a switch.
- Frames include:
  - **Broadcast frames** (e.g., ARP requests, DHCP).
  - **Multicast frames** (if subscribed).
  - **Unicast frames** destined to C’s MAC address.
  - If C is in promiscuous mode, it captures **all unicast frames** on the segment, including those not addressed to it.

**Explanation**: In a hub-based network, all devices receive all frames. In a switched network, switches forward frames only to the intended recipient, but promiscuous mode allows C to capture all frames reaching its interface. Without additional context (e.g., switch vs. hub), assume C can sniff all traffic on the `eth0` segment.

#### **3. What is the principle of ARP spoofing?**

**ARP spoofing** (or ARP poisoning) is an attack where a malicious device (e.g., C) sends fake ARP replies to associate its MAC address with the IP address of another device (e.g., B or R1). This tricks other devices into sending packets to the attacker instead of the legitimate device.

**Principle**:
- C sends unsolicited ARP replies claiming, “IP X.X.X.X has MAC C.”
- Victims update their ARP tables with the false IP-to-MAC mapping.
- Packets intended for the victim’s IP are sent to C’s MAC, allowing C to intercept, modify, or forward them (e.g., in a man-in-the-middle attack).

#### **4. Which machines can launch an ARP spoofing attack on the network connected to interface `eth0` of router R1? Explain.**

**Answer**: Any machine connected to the network segment attached to `eth0` of R1 can launch an ARP spoofing attack. This includes machines like B, C, and any others on the same LAN.

**Explanation**:
- ARP operates at the link layer and is confined to a single broadcast domain (e.g., the LAN on `eth0`).
- To perform ARP spoofing, a machine must be able to send ARP replies to devices on the same segment.
- Machines outside this segment (e.g., on another interface of R1) cannot directly send ARP packets to devices on `eth0`, as ARP does not cross routers.

#### **5. What is the state of the ARP tables of machines B and R1 before the attack?**

Assuming a typical network setup with B, C, and R1 on the same LAN:
- **Machine B’s ARP table**:
  - Contains the IP-to-MAC mapping for R1’s `eth0` interface (e.g., `IP_R1: MAC_R1`), as B communicates with the router for external traffic.
  - May include C’s mapping (`IP_C: MAC_C`) if B has recently communicated with C.
- **Router R1’s ARP table** (for `eth0`):
  - Contains mappings for devices on the `eth0` LAN, e.g., `IP_B: MAC_B`, `IP_C: MAC_C`.

**Example** (hypothetical IPs and MACs):
- B’s ARP table: `{ IP_R1: MAC_R1, IP_C: MAC_C }`
- R1’s ARP table: `{ IP_B: MAC_B, IP_C: MAC_C }`

#### **6. What steps must machine C perform to execute the ARP spoofing attack to intercept all traffic on the `eth0` network?**

To intercept traffic between B and R1 (man-in-the-middle), C performs these steps:
1. **Enable IP forwarding** (optional): If C wants to forward packets to avoid detection, enable IP forwarding on its system.
2. **Poison B’s ARP table**:
   - Send fake ARP replies to B, claiming: “R1’s IP (IP_R1) has MAC_C.”
   - B updates its ARP table to map `IP_R1` to `MAC_C`.
3. **Poison R1’s ARP table**:
   - Send fake ARP replies to R1, claiming: “B’s IP (IP_B) has MAC_C.”
   - R1 updates its ARP table to map `IP_B` to `MAC_C`.
4. **Repeat periodically**: ARP entries expire, so C sends spoofed ARP replies at regular intervals (e.g., every few seconds).
5. **Sniff or manipulate traffic**: C receives packets sent to `IP_B` or `IP_R1`, can inspect/modify them, and optionally forward them to the correct destination.

#### **7. Describe the state of the ARP tables of machines B, R1, and C after the attack.**

After C successfully poisons the ARP tables:
- **Machine B’s ARP table**:
  - `IP_R1` now maps to `MAC_C` (instead of `MAC_R1`).
  - Example: `{ IP_R1: MAC_C, IP_C: MAC_C }`
- **Router R1’s ARP table**:
  - `IP_B` now maps to `MAC_C` (instead of `MAC_B`).
  - Example: `{ IP_B: MAC_C, IP_C: MAC_C }`
- **Machine C’s ARP table**:
  - Unchanged, as C does not need to modify its own table to send spoofed replies.
  - Contains legitimate mappings, e.g., `{ IP_B: MAC_B, IP_R1: MAC_R1 }`.

Traffic from B to R1 (and vice versa) now goes through C, enabling interception.
