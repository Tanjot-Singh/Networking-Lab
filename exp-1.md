# Practical 1: Basic Network Configuration

## Objective
Learn to configure basic network settings on Cisco devices.

---

## Theory
Basic network configuration on Cisco devices involves setting up hostnames, IP addresses, passwords, and other fundamental settings using the Cisco IOS CLI. This is the foundation for any network setup.

---

## Equipment / Tools Required
- Cisco Packet Tracer (or real Cisco Router/Switch)
- PC with terminal emulator (PuTTY / Packet Tracer terminal)

---

## Network Topology

```
PC0 ──────── Router0 ──────── PC1
192.168.1.2   Fa0/0: 192.168.1.1
              Fa0/1: 192.168.2.1   192.168.2.2
```

---

## Configuration

### Step 1: Access Privileged EXEC Mode
```
Router> enable
Router#
```

### Step 2: Enter Global Configuration Mode
```
Router# configure terminal
Router(config)#
```

### Step 3: Set Hostname
```
Router(config)# hostname R1
R1(config)#
```

### Step 4: Set Enable (Privileged) Password
```
R1(config)# enable secret cisco123
```

### Step 5: Configure Console Password
```
R1(config)# line console 0
R1(config-line)# password console123
R1(config-line)# login
R1(config-line)# exit
```

### Step 6: Configure VTY (Telnet/SSH) Password
```
R1(config)# line vty 0 4
R1(config-line)# password vty123
R1(config-line)# login
R1(config-line)# exit
```

### Step 7: Configure Interface Fa0/0 (LAN Side)
```
R1(config)# interface FastEthernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

### Step 8: Configure Interface Fa0/1 (LAN Side 2)
```
R1(config)# interface FastEthernet 0/1
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

### Step 9: Set Banner Message
```
R1(config)# banner motd # Unauthorized Access is Prohibited! #
```

### Step 10: Disable DNS Lookup (optional but recommended)
```
R1(config)# no ip domain-lookup
```

### Step 11: Save Configuration
```
R1# write memory
```
or
```
R1# copy running-config startup-config
```

---

## PC Configuration

| Device | IP Address    | Subnet Mask     | Default Gateway |
|--------|---------------|-----------------|-----------------|
| PC0    | 192.168.1.2   | 255.255.255.0   | 192.168.1.1     |
| PC1    | 192.168.2.2   | 255.255.255.0   | 192.168.2.1     |

---

## Verification Commands

```
R1# show running-config
R1# show interfaces
R1# show ip interface brief
R1# show version
```

### Expected Output of `show ip interface brief`:
```
Interface         IP-Address      OK?  Method  Status                Protocol
FastEthernet0/0   192.168.1.1     YES  manual  up                    up
FastEthernet0/1   192.168.2.1     YES  manual  up                    up
```

---

## Testing Connectivity

From PC0, open Command Prompt and ping:
```
ping 192.168.1.1   → Should succeed (Gateway)
ping 192.168.2.2   → Should succeed (PC1)
```

---

## Result
Basic network configuration was successfully implemented on a Cisco router. IP addresses were assigned to interfaces, passwords were set, and connectivity between two PCs was verified using ping.

---

## Conclusion
This practical demonstrates the fundamental steps to configure a Cisco router including hostname, passwords, interface IPs, and connectivity verification using the IOS CLI.
