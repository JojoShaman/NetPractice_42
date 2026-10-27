*This project has been created as part of the 42 curriculum by srosu*

# NetPractice 42

## Description
This 42 project is an introduction to the basics of computer networking.
Here are the main topics of this small practical project:
- IP configuration
- connecting devices through switches and routers
- understanding the role of a gateway within a network

The notions introduced in this project are fairly basic, so instead of giving a simple overview, I wanted to explore the important concepts that help anyone understand the project better, or anyone who wants to dive into computer networking. There are many other important concepts, but this document only covers the general ones.

## Network Concepts

### TCP/IP
TCP/IP, which stands for `Transmission Control Protocol / Internet Protocol`, is a model that describes how two devices communicate over a network and how data reaches its destination correctly.

The TCP/IP model was developed in the 1970s, before the `OSI Model` (published by the ISO in 1984). The OSI model was designed as a more complete, theoretical reference, while TCP/IP is the model actually used on the Internet. People still often use OSI layer names and numbers to talk about a specific layer (for example "a layer 3 device").

Some sources describe TCP/IP with 4 layers (Physical and Data Link merged into a single *Network Access* layer), others with 5. The 5-layer version is the most common:

- **Layer 5 - Application**
  - Produces the actual data exchanged by a service (HTTP, DNS, SSH...).
- **Layer 4 - Transport**
  - First encapsulation: a header with the **source and destination ports** is added, so the data reaches the right application. TCP also adds sequence numbers and acknowledgements to guarantee reliable delivery; UDP doesn't, which makes it faster but unreliable.
  - The data unit is called a `segment` with **TCP** or a `datagram` with **UDP**.
- **Layer 3 - Network**
  - The segment/datagram is wrapped with an IP header containing the **source IP** address, the **destination IP** address and control fields (such as TTL). The data unit is called a `packet`.
- **Layer 2 - Data Link**
  - The packet is wrapped one last time with a header and a trailer. The header mostly contains the **source and destination MAC addresses**. The trailer contains the **FCS** (Frame Check Sequence), a 32-bit value computed with the **CRC-32** algorithm. The receiver computes it again and compares both values to detect corruption.
  - The data unit is called a `frame`.
- **Layer 1 - Physical**
  - The frame is converted into bits and sent through a physical medium (copper cable, fiber, radio waves) to the next device.

When sending, data goes down from layer 5 to layer 1, each layer adding its own header: this is called `encapsulation`. When the destination receives the frame, it goes up from layer 1 to layer 5, each layer removing its header: this is called `decapsulation`.

> Note: encryption is **not** a role of the transport layer. It is handled by protocols such as TLS (used by HTTPS) that sit between the application and the transport layer.

### OSI Model
The OSI (*Open Systems Interconnection*) model splits communication into 7 layers. It is mostly used as a reference to describe and troubleshoot networks.

| OSI layer | Name | TCP/IP equivalent | Example |
|---|---|---|---|
| 7 | Application | Application | HTTP, DNS |
| 6 | Presentation | Application | encoding, TLS |
| 5 | Session | Application | session management |
| 4 | Transport | Transport | TCP, UDP |
| 3 | Network | Network | IP, routers |
| 2 | Data Link | Data Link | Ethernet, MAC, switches |
| 1 | Physical | Physical | cables, signals |

The main difference: OSI splits the TCP/IP application layer into three layers (Application, Presentation, Session).

### IP addresses
An IPv4 address is a 32-bit number, written as 4 numbers from 0 to 255 separated by dots: `192.168.1.10`.

An address is always used with a **subnet mask**, which tells which part of the address identifies the **network** and which part identifies the **host**.

```
IP address : 192.168.1.10   -> 11000000.10101000.00000001.00001010
Mask       : 255.255.255.0  -> 11111111.11111111.11111111.00000000
                               |------- network -------| |-host-|
```

The mask can also be written in **CIDR notation**: `/24` means the first 24 bits are the network part. So `192.168.1.10/24` = `192.168.1.10` with mask `255.255.255.0`.

**Private addresses** (RFC 1918) can be used freely inside a local network but are **not routable on the Internet**:
- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

Other special addresses:
- `127.0.0.0/8`: loopback (the machine itself)
- `0.0.0.0/0`: "any destination", used for default routes

### Subnetting
Subnetting means splitting a network into smaller networks by making the mask longer.

In every subnet, two addresses can't be given to a host:
- the **network address** (all host bits at 0)
- the **broadcast address** (all host bits at 1)

So the number of usable hosts is `2^(32 - prefix) - 2`.

| CIDR | Mask | Addresses | Usable hosts |
|---|---|---|---|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |

**Example:** which subnet does `192.168.1.70/26` belong to?
- A `/26` gives blocks of 64 addresses: `0-63`, `64-127`, `128-191`, `192-255`
- `70` is in the `64-127` block
- Network address: `192.168.1.64`
- Broadcast address: `192.168.1.127`
- Usable hosts: `192.168.1.65` to `192.168.1.126`

Quick trick: block size = `256 - last non-255 byte of the mask` (here `256 - 192 = 64`).

Two hosts can talk directly only if they are in the **same subnet**. Otherwise, they need a router.

### Switch
A switch is a **layer 2** device that connects devices inside the **same network**. It reads the **destination MAC address** of each frame and forwards it only to the right port, using a MAC address table it learns automatically.

A switch doesn't look at IP addresses and has no IP configuration in NetPractice: all hosts connected to the same switch must be in the same subnet.

### Router
A router is a **layer 3** device that connects **different networks** together. Each router interface has its own IP address, which belongs to the network connected to that interface.

To decide where to send a packet, a router reads the **destination IP** and checks its **routing table**. Each route says: "to reach this network, send the packet to this next hop".

```
Destination       Next hop
10.0.2.0/24   =>  192.168.0.2
0.0.0.0/0     =>  192.168.0.1    (default route)
```

The most specific route (longest prefix) is chosen. If no route matches, the packet is dropped.

### Gateway
The gateway (or default gateway) is the address a host sends its packets to when the destination is **outside its own subnet**. It is usually the IP address of the router interface connected to that subnet.

A host's route `default => 192.168.1.1` (or `0.0.0.0/0 => 192.168.1.1`) means: "for anything not in my network, ask `192.168.1.1`". The gateway must be in the **same subnet** as the host, otherwise the host can't reach it.

## Instructions

### Running the training interface
1. Download the archive attached to the project's page on the intranet and extract it into any folder.
2. In that folder, run the launch script:
   ```bash
   ./run.sh
   ```
   It starts a local web server and opens the NetPractice page in your default browser.
3. If `run.sh` doesn't work, start the server manually and open the page yourself:
   ```bash
   python3 -m http.server 49242
   ```
   Then go to `http://localhost:49242` (any other free port works too).

> A local web server is required: browsers block some features when the pages are opened directly as files.

### Solving the levels
- **Training** tab: enter your intranet login to get your personal configuration. Entering the login is mandatory for the exported files to be valid.
- **Evaluation** tab: generates a random configuration, used during the defense.

Each level shows a non-functioning network diagram and one or more goals at the top of the page. Modify the **unshaded fields** (IP addresses, masks, routes) until every goal is reached:
- **Check again**: checks whether the current configuration is correct.
- **Get my config**: downloads the configuration of the current level.
- **Next level**: appears once the level is solved.

The logs at the bottom of the page show the path of each packet and where it gets dropped (missing route, invalid IP, wrong gateway...).

### Exporting the configurations
Before moving to the next level, click **Get my config** to export the configuration of the solved level. Repeat for all 10 levels, then place the 10 files at the root of the Git repository.

### Tips
Tips that solve most levels:
- Hosts and the router interface they are connected to must be in the same subnet, with the same mask.
- Two subnets must never overlap, even if they are on different sides of a router.
- Never use the network or broadcast address of a subnet for a host.
- Check routes in both directions: the reply has to come back too.
- The Internet can't route private addresses: a destination behind the Internet must use a public range.
- Use the logs at the bottom of the page to see where a packet gets lost.

## Resources

### Networking concepts studied
- TCP/IP model and TCP/IP addressing (IPv4)
- OSI layers and encapsulation / decapsulation
- Subnet masks, CIDR notation and subnetting
- Network and broadcast addresses
- Public and private IP ranges
- Default gateways
- Routers and routing tables (default route, next hop)
- Switches and MAC addresses

### References
- [RFC 1918 - Address Allocation for Private Internets](https://datatracker.ietf.org/doc/html/rfc1918)
- [RFC 791 - Internet Protocol](https://datatracker.ietf.org/doc/html/rfc791)
- [Cisco - IP Addressing and Subnetting for New Users](https://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13788-3.html)
- [Cloudflare Learning Center - What is the OSI model?](https://www.cloudflare.com/learning/ddos/glossary/open-systems-interconnection-model-osi/)
- [Professor Messer - Network+ course](https://www.professormesser.com/)

### AI usage
 an AI assistant was used to help structure this README, review the explanations for technical accuracy and complete some sections. All levels of the project were solved without AI.

## Submission details
- The repository must contain **10 exported configuration files** (one per level, downloaded with **Get my config** in training mode with the intranet login entered), placed **at the root of the repository**.
- Only the content of the Git repository is evaluated, so file names should be double-checked before submitting.
- During the defense, **3 random levels** must be solved in the Evaluation tab within a limited amount of time.
- No external tools are allowed during the evaluation, except a simple calculator such as `bc`.