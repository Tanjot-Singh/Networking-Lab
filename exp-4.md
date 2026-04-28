
# Practical 4: Configuring Access Control Lists (ACLs)

## Objective
Use Access Control Lists (ACLs) to control network traffic and restrict access on Cisco routers.

---

## Theory
An ACL is an ordered list of permit/deny rules applied to router interfaces. It filters packets based on:
- **Standard ACL (1–99):** Filters based on source IP only.
- **Extended ACL (100–199):** Filters based on source IP, destination IP, protocol, and port.

ACLs are applied either **inbound** or **outbound** on an interface.

---

## Equipment / Tools Required
- Cisco Packet Tracer
- 1 Cisco Router
- 1 Cisco Switch
- 3 PCs

---

## Network Topology

```
PC0 (192.168.1.10) ──┐
                      ├── SW1 ── R1(Fa0/0: 192.168.1.1) ── Fa0/1: 192.168.2.1 ── Server (192.168.2.10)
PC1 (192.168.1.20) ──┘
```

---

## IP Addressing Table

| Device | IP Address     | Subnet Mask   | Gateway       |
|--------|----------------|---------------|---------------|
| PC0    | 192.168.1.10   | 255.255.255.0 | 192.168.1.1   |
| PC1    | 192.168.1.20   | 255.255.255.0 | 192.168.1.1   |
| Server | 192.168.2.10   | 255.255.255.0 | 192.168.2.1   |
| R1 Fa0/0 | 192.168.1.1  | 255.255.255.0 | —           |
| R1 Fa0/1 | 192.168.2.1  | 255.255.255.0 | —           |

---

## Configuration

### Part A: Basic Router Setup
```
Router> enable
Router# configure terminal
Router(config)# hostname R1

R1(config)# interface FastEthernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface FastEthernet 0/1
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

---

### Part B: Standard ACL — Block PC1 from reaching Server

**Goal:** Deny PC1 (192.168.1.20) from accessing the server. Allow all others.

```
R1(config)# access-list 10 deny host 192.168.1.20
R1(config)# access-list 10 permit any
```

**Apply ACL to Fa0/1 (outbound toward server):**
```
R1(config)# interface FastEthernet 0/1
R1(config-if)# ip access-group 10 out
R1(config-if)# exit
```

---

### Part C: Extended ACL — Block Telnet from PC0 to Server

**Goal:** Deny Telnet (TCP port 23) from PC0 (192.168.1.10) to Server, allow all other traffic.

```
R1(config)# access-list 110 deny tcp host 192.168.1.10 host 192.168.2.10 eq 23
R1(config)# access-list 110 permit ip any any
```

**Apply ACL to Fa0/0 (inbound from LAN):**
```
R1(config)# interface FastEthernet 0/0
R1(config-if)# ip access-group 110 in
R1(config-if)# exit

R1# write memory
```

---

### Part D: Named ACL (Alternative Syntax)

```
R1(config)# ip access-list extended BLOCK_TELNET
R1(config-ext-nacl)# deny tcp host 192.168.1.10 host 192.168.2.10 eq telnet
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit

R1(config)# interface FastEthernet 0/0
R1(config-if)# ip access-group BLOCK_TELNET in
R1(config-if)# exit
```

---

## Verification Commands

```
R1# show access-lists
R1# show ip access-lists
R1# show running-config | include access
R1# show ip interface FastEthernet 0/0
R1# show ip interface FastEthernet 0/1
```

### Expected Output of `show access-lists`:
```
Standard IP access list 10
    10 deny   host 192.168.1.20 (X matches)
    20 permit any

Extended IP access list 110
    10 deny tcp host 192.168.1.10 host 192.168.2.10 eq telnet (X matches)
    20 permit ip any any
```

---

## Testing

### Standard ACL Test:
```
PC0> ping 192.168.2.10    → Should SUCCEED ✅
PC1> ping 192.168.2.10    → Should FAIL ❌ (blocked by ACL 10)
```

### Extended ACL Test:
```
PC0> telnet 192.168.2.10  → Should FAIL ❌ (Telnet blocked)
PC0> ping 192.168.2.10    → Should SUCCEED ✅ (only Telnet is blocked)
```

---

## Result
ACLs were successfully configured on the Cisco router. Standard ACL blocked a specific host, and Extended ACL filtered traffic based on protocol and port, demonstrating granular traffic control.

---

## Conclusion
Access Control Lists are a powerful tool for network security. Standard ACLs provide basic source-based filtering, while Extended ACLs offer fine-grained control over traffic using source, destination, protocol, and port criteria.
