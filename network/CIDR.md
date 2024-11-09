**CIDR (Classless Inter-Domain Routing)** is a method for allocating IP addresses and routing internet traffic. It was introduced to replace the older system based on **classful IP addressing**, which was inefficient in terms of address space usage.

### 1. **What is CIDR?**
CIDR notation is used to define a range of IP addresses. It consists of an IP address followed by a slash (`/`) and a number (called the **prefix length**). The prefix length indicates how many bits of the IP address are used for the **network part**.

#### Example:
- `192.168.1.0/24`
    - `192.168.1.0`: Network address
    - `/24`: Prefix length (the first 24 bits are used for the network, and the remaining bits are for hosts)

### 2. **IP Address Breakdown**
An IP address is a 32-bit number, divided into 4 octets (8-bit sections) for readability.

#### Example:
- IP Address: `192.168.1.0`
- Binary Representation: `11000000.10101000.00000001.00000000`

### 3. **How CIDR Works**
The prefix length (`/n`) tells you how many bits are used for the **network portion**, and the rest are used for the **host portion**.

#### Example:
- `192.168.1.0/24`
    - **Network bits**: First 24 bits (`11000000.10101000.00000001`)
    - **Host bits**: Last 8 bits (`00000000`)
    - **Address range**: `192.168.1.0` to `192.168.1.255`
    - **Usable hosts**: `192.168.1.1` to `192.168.1.254` (254 hosts; `.0` is the network address and `.255` is the broadcast address)

### 4. **CIDR Notation Table**

| CIDR Notation | Subnet Mask       | Total IPs | Usable IPs | Description          |
|---------------|-------------------|-----------|------------|----------------------|
| /8            | 255.0.0.0         | 16,777,216| 16,777,214 | Very large network   |
| /16           | 255.255.0.0       | 65,536    | 65,534     | Large network        |
| /24           | 255.255.255.0     | 256       | 254        | Common for LANs      |
| /28           | 255.255.255.240   | 16        | 14         | Small subnetwork     |
| /30           | 255.255.255.252   | 4         | 2          | Point-to-point link  |

### 5. **Subnet Mask Conversion**
The **subnet mask** defines which portion of the IP address is the network part and which is the host part.

| Prefix Length | Subnet Mask       |
|---------------|-------------------|
| /8            | 255.0.0.0         |
| /16           | 255.255.0.0       |
| /24           | 255.255.255.0     |
| /25           | 255.255.255.128   |
| /26           | 255.255.255.192   |
| /27           | 255.255.255.224   |
| /28           | 255.255.255.240   |
| /29           | 255.255.255.248   |
| /30           | 255.255.255.252   |

### 6. **Calculating the Network Range**
To determine the range of IP addresses in a CIDR block, follow these steps:

1. **Identify the network bits** (based on the prefix length).
2. **Calculate the number of host bits** (32 - prefix length).
3. **Determine the number of IP addresses**: \( 2^{\text{host bits}} \).
4. **Determine the usable IP addresses**: \( 2^{\text{host bits}} - 2 \) (subtracting the network and broadcast addresses).

#### Example:
- CIDR Block: `10.0.0.0/28`
    - Network Bits: 28
    - Host Bits: 4
    - Total IPs: \( 2^4 = 16 \)
    - Usable IPs: \( 16 - 2 = 14 \)
    - IP Range: `10.0.0.0` to `10.0.0.15`
    - Usable IPs: `10.0.0.1` to `10.0.0.14`

### 7. **Supernetting vs. Subnetting**
- **Subnetting**: Dividing a larger network into smaller sub-networks (e.g., splitting a `/24` into `/28` blocks).
- **Supernetting**: Aggregating smaller networks into a larger network (e.g., combining `/24` networks into a `/22`).

#### Subnetting Example:
- Original Network: `192.168.0.0/24`
- Subnets: `192.168.0.0/26`, `192.168.0.64/26`, `192.168.0.128/26`, `192.168.0.192/26`

#### Supernetting Example:
- Original Networks: `192.168.1.0/24`, `192.168.2.0/24`
- Supernet: `192.168.0.0/22`

### 8. **CIDR in AWS**
In AWS, CIDR blocks are heavily used for **VPCs (Virtual Private Clouds)**, **subnets**, and **security groups**.

#### Example:
- **VPC CIDR**: `10.0.0.0/16` (65,536 IPs)
- **Public Subnet CIDR**: `10.0.1.0/24` (256 IPs)
- **Private Subnet CIDR**: `10.0.2.0/24` (256 IPs)

In a typical AWS architecture:
- **VPC**: Defines the overall IP address range using a large CIDR block (e.g., `/16`).
- **Subnets**: Divide the VPC CIDR block into smaller blocks for public, private, and isolated subnets.
