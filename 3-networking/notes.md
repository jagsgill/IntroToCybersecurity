# Networking

## Network Communication Model 
Three components: data sources + medium for data transmission + addressing/locators

1) People + cup&string + names
2) Mobile phones + wireless signal + phone number

There are 3 types of network transmissions: unicast, broadcast, multicast.

### Unicast
1 sender, 1 receiver. 

### Broadcast
1 sender, 0+ receivers

### Multicast
1+ sender, 0+ receivers

## Addressing
IP address is used to uniquely identify a network node - format X.X.X.X (4 octets)
MAC address is used to uniquely identify a network interface - format XX:XX:XX:XX:XX:XX (6 hex digits) 
  - Note: network devices may have multiple network interfaces. Each interface has a *globally* unique MAC address, which is baked into the hardware (sometimes can be modified, or spoofed by attackers)

2 versions: IPv4 (older) and IPv6. IPv4 is much more widely used than IPv6.

### IPv4

#### Structure

32-bit address as a dotted-quad (X.X.X.X). 8-bits per section = octets/"standard" byte,
a.k.a. a set of 4 octets/bytes. So addresses range from 0.0.0.0 to 255.255.255.255

Total # of available IPv4 addresses
= 2^32
= 2^2 * (2^10)^3
~ 4 billion

Note that not all addresses are available for public addressing. Some must be reserved
for private addressing, other services, testing etc. About 1/8th of addresses are
allocated for non-public use. Based on population growth and Internet-connected devices,
IPv4 does not provide sufficient addresses to address global needs.

#### Address space
The first octet can be used to distinguish public from reserved addresses.

- Fully reserved:
  - | 1st octet | Examples | Uses |
    |:---:|:---:|---|
    | 0 | 0.0.0.0 | For a server, 0.0.0.0 is equivalent to all IPv4 addresses on the machine. In routing tables, it usually means default route |
    | 10 | 10.1.2.3 | Local communication within a private network |
    | 127 | 127.0.0.1 | Loopback address to the local host |
- Partially reserved:
  - | 1st octet | Examples | Uses |
    |:---:|:---:|---|
    | 192 | 192.168.0.1 | Local communication within a private network |
    | 192 | 192.0.2.0 | Reserved for use in documentation/examples. This address block is referred to as TEST-NET-1  |
- "Future use"
  - | 1st octet | Examples | Uses |
    |:---:|:---:|---|
    | 224 | 224.0.0.1 | Used for IP multicast | 
    | 240 | 255.255.255.1 | Reserved for future use or experimental features. Effectively they are blacklisted by (nearly all?) network nodes so they are not really usable in the future |

#### CIDR Notation
CIDR is an acronym for Classless Inter-Domain Routing. It's the method used for allocating
addresses.

CIDR provides for 2 groups of bits in an address - network prefix (most significant bits) and host identifier (least significat bits).
It uses Variable-Length Subnet Masking to specify arbitrary-length prefixes. This means
the size of the 2 groups of bits is not static. The prefix size is indicated with a suffix for the number of bits in the prefix.
e.g. 192.168.0.0/16 means the first 16-bits are for the network prefix.

The host identifier identifies a unique network interface on the network. A single device can have multiple interfaces, so different host identifiers mean different network interfaces, not necessarily different devices.

The network prefix defines a netblock (range of consecutive addresses). 
A subnet is when a netblock is broken down further into smaller networks (by having a larger prefix) - e.g. given a /17 network, you can divide it into many /22 subnetworks
A subnet mask can be applied to an address with bitwise AND to obtain the network prefix.
For 192.168.1.1/16, the subnet mask is 255.255.0.0, and the prefix is 192.168.0.0 (think about the bits).

Addresses with identical __routing prefixes__ are in the same subnet. This does not mean addresses with the same prefix.
e.g. Addresses with first octet 192 are not all in the same subnet. 192.0.2.0/24 and 192.168.0.0/24 are not the same subnet. Use the subnet mask to compare prefixes.
So 192.0.2 an 192.0.0 are different network prefixes.

Packets destined for an IP within the same subnet will not leave the network. Only when the IPs are in different networks will
a packet go through the network's router to outside the network.

Note the two typical private network address blocks, 10.x.x.x and 192.168.0.x - more formally, these blocks are
10.0.0.0/8 and 192.168.0.0/16. The former one has a MUCH larger capacity for addresses (24 vs 16 bits). 

Note that the 1st, 2nd and last host identifiers are reserved, so you can subtract 3 from the number of 
possible hosts on a network.
 - .0 for network identity
 - .1 for router
 - (highest) address for broadcast (any data sent to broadcast address is sent to all devices on network)
 
Example:
To broadcast a packet to an entire IPv4 subnet using the private IP address space 172.16.0.0/12,
- subnet mask: (1st 12 bits = 1, last 20 bits = 0) = 255.240.0.0
- network prefix: 172.16.0.0 & (any address in the address space) = 172.16.0.0
- largest host identifer: network prefix | (1st 12 bits = 0, last 20 bits = 1) = 172.31.255.255      
- **Broadcast to 172.31.255.255**

##### Why?
Before CIDR, "classful" network design was used. This allocated one or more octets to different
classes (A, B, C, D, E) of addresses. Basically, this was too coarse as the internet grew - the groups
of bits were so large that they could not be split up among the high number of interested parties.
Finer-grained allocation of addresses was needed, which was achieved using the "classless"
approach.

## OSI Model

### Protocols

Networks may, (or should/must for the Internet in particular) need to support any type of device. How can we allow
many different types of devices to communicate with each other? Standards. Open standards. An open set of standards
describing how to format, send, and receive data facilitates this common method of communication. If all devices
on a network can "speak the same language," then they can communicate with each other.

Protocol: a set of conventions governing the treatment and especially the formatting of data in an electronic communications system.

### OSI

OSI Model = Open Systems Intercommunication Model

Network communication is complicated. This model breaks down network communication into various "layers" that build
on top of each other. Note that it is a *conceptual model* to help understand network systems - it's not exactly how the real systems work. 

Like a building, bottom layer is most fundamental, and the lower layers enable the upper layers to exist.

There are 7 layers:

| Layer | Name | Protocol Data Unit (PDU) | Function |
|:---:|:---:|:---:|---|
| 7 | Application | Data | High level APIs |
| 6 | Presentation | Data | Translation of data between networking service and application. e.g. char encoding, data compression, encryption/decryption |
| 5 | Session | Data | Managing communication sessions (back-and-forth transmissions btw 2 nodes) |
| 4 | Transport | Segment or Datagram | Reliable transmission of data btw network points (segmentation, acknowledgement, multiplexing) |
| 3 | Network | Packet | Structuring & managing a multi-node network (addressing, routing, traffic control) |
| 2 | Data Link | Frame | Reliable transmission of data frames btw 2 nodes connected by physical layer |
| 1 | Physical | Symbol | Transmission & reception of raw bit streams over a physical medium |

Abstractions: each layer only needs to know about the next layer above and below it. For example, the presentation layer
abstracts away the lower layers, so the application layer is not concerned with the physical or transport layers. 
This is because of the data flow, 4->3->2 or 4 <- 3 <- 2

### Data flow
When sending data, it passes down from higher to lower layers. Opposite for receiving data.

SDU = service data unit (the payload)

PDU in level N is composed of an SDU(the PDU of level N+1) + headers/footers for the protocol at level N

1. Top layer converts data into PDU
2. Pass to layer N-1 (the PDU from step 1 is an SDU for this layer)
3. The protocol at layer N-1 adds it's headers/footers to produce the layer N-1 PDU
4. Pass to layer N-2, and so on.
5. Layer 1 transmits the final PDU to a receiver
6. At the receiver, data is passed to successively higher layers after stripping off the current layer's headers/footers.
7. Layer 7 at the receiver receives the original data from step 1.

### Layer 1
Responsible to transmit bits (digital to electrical/radio/optical conversion).
This can be:
- simplex: send data in one direction only (e.g. TV broadcast, radio signals only carry info in one direction)
- half duplex: send data in both directions, but not simultaneously - conserves bandwidth 
e.g. pair of walkie-talkies use one radio frequency to communicate. Simultaneous transmission causes loss of data 
from radio signal interference. Uses single channel
- full duplex: send data in both directions, simultaneously (e.g. landline or mobile phone). Uses 2 channels 
(receive vs send) e.g. 2 radio frequencies, 2 wires

#### Examples: 

##### 802.11 wireless LAN, USB, Ethernet's physical layer

##### Ethernet Hubs
Hubs form a network of directly connected nodes. They have several Ethernet ports, and when a device sends a message
intended for some node, the hub broadcasts the message to all of the other Ethernet ports (but not the src port).
This is a simple but not very efficient/clean way to achieve a network. A more expensive and efficient solution is a
layer 2 device called a "network switch."

### Layer 2
This layer provides a connection between 2 directly connected nodes, and may provide 
- error correction for transmission errors in the physical layer (e.g. parity bit, or cyclic redundancy check, etc) 
  - even or odd parity bit: additional bit is set to 1 or 0 to ensure count of 1s is even/odd, depending on which parity scheme is used). Applied to each byte (1 parity + 7 data bits)
    - e.g. even parity: **1**0101001 / after transmission 10100001. Parity bit is 1 but count of 1s is odd... meaning there was a transmission error. Typically corrected by re-transmitting the octet.
    - TODO CRC
- flow control - the receiver can slow down a fast sender so that it is not overwhelmed with too much data at once

Protocols here define how to establish & terminate connection btw 2 _physically_ connected nodes - A-B vs A-B-C,
the A-C connection is not responsibility of this layer, only the A-B and B-C connections. 
Nodes connected by Ethernet cables and through network switches (within same network) are involved here. 
Nodes connected through gateways or routers (different networks) are not - that is a Layer 3 function.

#### Examples: 

##### Network Switches
A switch is like an intelligent ethernet hub. It maintains a mapping of Ethernet-jack:MAC address in a [MAC Address Table](https://community.cisco.com/t5/networking-documents/overview-of-layer-2-switched-networks-and-communication/ta-p/3128423).
When a device A sends a message to device B, the message is sent only to the Ethernet port connecting to device B.

##### ARP
Address resolution protocol. 

It concerns IP address <-> MAC address resolution within subnetworks.

MAC address (media access control), a.k.a. hardware address, is a *globally* unique identifier, injected by 
manufacturers into network devices. So it uniquely identifies a network device. Is 6 pairs of hex digits, e.g. 3c:97:0e:44:be:ef

Why is ARP needed? 
- Consider a private network/LAN with all nodes connected by a switch. You as a user know the 
src and dst IP addresses. Your system is instructed to send a message to the dst IP address - but which machine is that?
IP and MAC are decoupled because IP address assignments are dynamic - you can assign an IP to some node then change
it later. IP is also at a higher level of abstraction (layer 3), so layer 2 protocols are not concerned about it. For convenience, layer 3 systems like DHCP can be used by nodes to obtain an IP address on the subnetwork.

- Ethernet <-1-> MAC address <-2-> IP address
  - 1: MAC Address Table (MAC : Ethernet Port)
  - 2: ?

Therefore a lookup scheme is needed to translate betweeen IP (layer 3) & MAC (layer 2) addresses.

- When sending a message to a node outside the subnet, the src device sends it to the MAC address of a router/gateway
in the subnet. 

Note that the broadcast ARP message needs a MAC address - the broadcast MAC is ff:ff:ff:ff:ff:ff (see with `arp -a`). Data sent to the broadcast MAC is sent to all connected layer-2 nodes. Cf. broadcast IP address (layer 3), which sends data to all connected layer-3 nodes. 

When performing an IP broadcast, all applicable network switches will perform a MAC broadcast as well.

Basic Process:
- Note: All layer 2 devices maintain a cache of IP:MAC mappings in an ARP Lookup Table

```
if (cache has entry for IP:MAC of interest) {
    - Send message to MAC with key of IP 
} else {
    - Send a *broadcast* ARP message to the broadcast MAC address (ff:ff:ff:ff:ff:ff), "who has X.X.X.X" 

    ** Receive an ARP message from the node with IP X.X.X.X, "I am X.X.X.X. My MAC address is a:b:c:d:e:f"

    - Layer 2 nodes in the subnet cache this IP:MAC entry

    - Send original message to MAC with key of IP 
}
```

Step ** is a vulnerability point - other malicious nodes could respond faster than the real node, causing them
to receive the messages intended for another node. This is called "ARP spoofing."

The ARP table can be manipulated manually as well (add/delete/modify). Entries can also expire automatically.

```text
# arp -a
(192.168.1.64) at f4:e:83:39:9b:3 on en0 ifscope [ethernet]
(192.168.1.66) at 10:78:5b:4b:3a:c0 on en0 ifscope [ethernet]
(192.168.1.255) at ff:ff:ff:ff:ff:ff on en0 ifscope [ethernet]
```

Filter for all ARP traffic, as you delete an ARP entry and ping it:
```text
# sudo tcpdump -lni any arp & ( sleep 1; arp -d 192.168.1.70; ping -c1 -n 192.168.1.70 )
13:00:20.547624 ARP, Request who-has 192.168.1.70 tell 192.168.1.67, length 46
13:00:20.547640 ARP, Reply 192.168.1.70 is-at ac:bc:32:9c:d7:f1, length 28
```

Gratuitous ARP:
ARP is-at replies are normally a response to who-has requests. But, say you change the IP for a device and want to
immediately update the other devices' ARP tables. Then you can broadcast "gratuitous" ARP is-at messages using `arping`
command. When the other devices recieve these messages, they will update their ARP tables. 

### Layer 3
This layer is responsible for connecting nodes across different networks, by packet forwarding. 

This is connectionless (no ack).

All nodes in the network must have a unique host address (for the Internet, IP provides addresses).

Packets are forwarded node to node via the *routing* process, taking them from source node, through 
intermediate nodes, to the destination node. 

The nodes that facilitate packet forwarding across different networks are usually routers & gateways.

Packet forwarding may be unicast, broadcast or multicast.

#### Examples 

##### IPv4 - Internet Protocol v4
Packet format: header + payload

IP header contains
- IP version
- source IPv4 address
- destination IPv4 address
- total length (size of header [min. 20 B] + payload), up to 2^16
- time to live (TTL)
- 16-bit header checksum
- protocol (type of the payload data), e.g. 1 → ICMP, 4 → TCP, 17 → UDP
- other metadata

Note: the source and destination addresses in transmitted packets may be changed by a NAT (Network Address Translation) device. A NAT device is a gateway/router that sits between  its private network's hosts and public networks. For example, a NAT with a single public IP address maintains a NAT table which maps private host IP addresses to the public IP address and a unique port. Destination servers return packets to this external address:port combination, so the NAT can direct these packets to the source host by reversing the mapping.

Note: the header checksum is computed by a router for each incoming packet. If the computed checksum does not match the field, the packet is discarded. The payload checksum is handled by the encapsulated layer (i.e. TCP/UDP in transport layer both have checksum fields as well).

Note: the TTL prevents circular transmission of packets - typically set in units of "network hops."
Routers decrement the TTL field by 1. Once TTL reaches 0, the packet is discarded and the router
sends an ICMP Time Exceeded message back to the originator.

###### traceroute
`traceroute` prints the route packets take to reach a given network host. It does this by incrementing the TTL field of network layer packets to receive ICMP Time Exceeded responses (containing IP addresses) from successive nodes in the route.

```
$ traceroute google.ca
  traceroute to google.ca (172.217.14.195), 64 hops max, 52 byte packets
   1  192.168.1.254 (192.168.1.254)  535.054 ms  316.353 ms  144.089 ms
   2  10.31.54.1 (10.31.54.1)  455.156 ms  466.220 ms  207.993 ms
   3  154.11.12.219 (154.11.12.219)  228.584 ms  204.029 ms
      154.11.12.217 (154.11.12.217)  623.293 ms
   4  74.125.50.110 (74.125.50.110)  454.593 ms  512.744 ms  573.179 ms
   5  74.125.243.177 (74.125.243.177)  389.387 ms  200.582 ms
      74.125.243.193 (74.125.243.193)  203.501 ms
   6  209.85.254.237 (209.85.254.237)  215.466 ms  151.803 ms  213.515 ms
   7  sea30s01-in-f3.1e100.net (172.217.14.195)  198.439 ms  148.848 ms  149.228 ms
 ```



##### IPv6 - Internet Protocol v6

##### ICMP - Internet Control Message Protocol
ping utility is implemented using the ICMP echo request and echo reply messages... explain how

##### IPSec - Internet Protocol Security

### Layer 4

#### Ports

### Layer 5

### Layer 6

### Layer 7

## Switching



## Routing


## Domain Name System


## Load Balancing



## Review Questions
1. Explain how all components of a network function together to deliver packets from one device to another device, which could be on the other side of the world.

*

2. Determine whether an IP address is public or private.

*

3. Understand how the IPv4 address shortage is solved with NAT and IPv6.

*

4. Calculate the network address and subnet size given the address of one device on the network, and the subnet mask.

*

5. List the layers in the OSI Model

*

6. Describe how data moves down the layers in the OSI Model in a process called Encapsulation

*

7. Describe how data moves up the layers in the OSI Model in a process called Decapsulation

*

8. Diagram how data moves from the sending device and across a network to the target device

*

9. Recommend an appropriate switching architecture for a particular network size.

*

10. Use the Wireshark network analysis tool to extract address, FQDN, and DNS information from real network packet transmissions

*

11. Explain how the DNS server resolves an FQDN query to an IP address

*

12. Explain how load balancing works and the different ways of balancing the load of incoming network traffic

*

## Case Study: Describe in detail the process of sending an HTTP request and receiving a response.
