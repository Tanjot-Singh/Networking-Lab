# Practical 6: Implementing Network Address Translation (NAT)

## Objective
Configure NAT on a Cisco router to allow internal (private) devices to access external networks using public IP addresses.

---

## Theory
NAT (Network Address Translation) translates private IP addresses to a public IP address before packets leave the network. This conserves public IPs and hides the internal network structure.

**Types of NAT:**
| Type | Description |
|------|-------------|
| **Static NAT** | One-to-one mapping of private to public IP |
| **Dynamic NAT** | Maps private IPs to a pool of public IPs |
| **PAT (NAT Overload)** | Multiple private IPs share one public IP using port numbers |

---

## Equipment / Tools Required
- Cisco Packet Tracer
- 1 Cisco Router
- 1 Switch
- 2–3 PCs (inside network)
- 1 Server (outside/internet simulation)

---

## Network Topology

```
[Inside Network]                          [Outside / Internet]
PC1: 192.168.1.10 ──┐
PC2: 192.168.1.20 ──┤── SW1 ── R1(Fa0/0: 192.168.1.1 | Fa0/1: 203.0.113.1) ── Server: 203.0.113.10
PC3: 192.168.1.30 ──┘

           ↑ Inside (Private)                       ↑ Outside (Public)
```

---

## IP Addressing Table

| Device      | IP Address     | Subnet Mask   | Gateway       |
|-------------|----------------|---------------|---------------|
| PC1         | 192.168.1.10   | 255.255.255.0 | 192.168.1.1   |
| PC2         | 192.168.1.20   | 255.255.255.0 | 192.168.1.1   |
| PC3         | 192.168.1.30   | 255.255.255.0 | 192.168.1.1   |
| R1 Fa0/0    | 192.168.1.1    | 255.255.255.0 | —             |
| R1 Fa0/1    | 203.0.113.1    | 255.255.255.0 | —             |
| Server      | 203.0.113.10   | 255.255.255.0 | 203.0.113.1   |

---

## Configuration

### Step 1: Configure Router Interfaces
```
Router> enable
Router# configure terminal
Router(config)# hostname R1

R1(config)# interface FastEthernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# ip nat inside
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface FastEthernet 0/1
R1(config-if)# ip address 203.0.113.1 255.255.255.0
R1(config-if)# ip nat outside
R1(config-if)# no shutdown
R1(config-if)# exit
```

---

### Part A: Static NAT (One PC to One Public IP)

Map PC1's private IP to a dedicated public IP:
```
R1(config)# ip nat inside source static 192.168.1.10 203.0.113.2
```

---

### Part B: Dynamic NAT (Pool of Public IPs)

Define a pool of public IPs:
```
R1(config)# ip nat pool PUBLIC_POOL 203.0.113.2 203.0.113.5 netmask 255.255.255.0
```

Create an ACL to define which inside hosts can use NAT:
```
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255
```

Bind the ACL to the NAT pool:
```
R1(config)# ip nat inside source list 1 pool PUBLIC_POOL
```

---

### Part C: PAT / NAT Overload (Most Common — All PCs Share One Public IP)

```
R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255

R1(config)# ip nat inside source list 1 interface FastEthernet 0/1 overload
```

> This is the most commonly used NAT method in home/office routers.

### Step 4: Save Configuration
```
R1# write memory
```

---

## Verification Commands

```
R1# show ip nat translations
R1# show ip nat statistics
R1# show running-config | include nat
R1# debug ip nat
```

### Expected Output of `show ip nat translations` (PAT):
```
Pro  Inside global      Inside local       Outside local      Outside global
tcp  203.0.113.1:1025   192.168.1.10:1025  203.0.113.10:80    203.0.113.10:80
tcp  203.0.113.1:1026   192.168.1.20:1026  203.0.113.10:80    203.0.113.10:80
```

---

## Testing

From any inside PC, ping the outside server:
```
PC1> ping 203.0.113.10   → Should SUCCEED ✅
PC2> ping 203.0.113.10   → Should SUCCEED ✅
```

From the server, ping a private IP:
```
Server> ping 192.168.1.10  → Should FAIL ❌ (Private IPs not routable)
```

---

## Result
NAT was successfully configured on the Cisco router. Internal devices with private IPs were able to communicate with the external server using the public IP. PAT allowed multiple devices to share a single public IP address using different port numbers.

---

## Conclusion
This practical demonstrates the three types of NAT. PAT (NAT Overload) is the most practical and widely deployed method, allowing an entire internal network to share one public IP address, which conserves IPv4 address space and improves security.
