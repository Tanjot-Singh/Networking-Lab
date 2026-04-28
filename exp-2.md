# Practical 2: Configuring VLANs

## Objective
Implement VLANs to segment network traffic on a Cisco switch.

---

## Theory
A VLAN (Virtual Local Area Network) logically segments a physical network into multiple isolated broadcast domains. Devices in the same VLAN can communicate with each other but not with devices in other VLANs without a router or Layer 3 switch. VLANs improve security, performance, and manageability.

---

## Equipment / Tools Required
- Cisco Packet Tracer (or real Cisco Switch)
- 1 Cisco Switch (2960 or similar)
- 4 PCs

---

## Network Topology

```
        [Switch SW1]
       /     |     \      \
     Fa0/1  Fa0/2  Fa0/3  Fa0/4
      |      |      |       |
     PC1    PC2    PC3    PC4
  VLAN 10 VLAN 10 VLAN 20 VLAN 20
```

---

## IP Addressing Table

| Device | VLAN    | IP Address     | Subnet Mask   |
|--------|---------|----------------|---------------|
| PC1    | VLAN 10 | 192.168.10.1   | 255.255.255.0 |
| PC2    | VLAN 10 | 192.168.10.2   | 255.255.255.0 |
| PC3    | VLAN 20 | 192.168.20.1   | 255.255.255.0 |
| PC4    | VLAN 20 | 192.168.20.2   | 255.255.255.0 |

---

## Configuration

### Step 1: Enter Privileged EXEC Mode
```
Switch> enable
Switch# configure terminal
```

### Step 2: Set Hostname
```
Switch(config)# hostname SW1
```

### Step 3: Create VLANs
```
SW1(config)# vlan 10
SW1(config-vlan)# name Sales
SW1(config-vlan)# exit

SW1(config)# vlan 20
SW1(config-vlan)# name HR
SW1(config-vlan)# exit
```

### Step 4: Assign Ports to VLAN 10 (PC1 and PC2)
```
SW1(config)# interface FastEthernet 0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit

SW1(config)# interface FastEthernet 0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit
```

### Step 5: Assign Ports to VLAN 20 (PC3 and PC4)
```
SW1(config)# interface FastEthernet 0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# exit

SW1(config)# interface FastEthernet 0/4
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# exit
```

### Step 6: Save Configuration
```
SW1# write memory
```

---

## Verification Commands

```
SW1# show vlan brief
SW1# show vlan id 10
SW1# show interfaces FastEthernet 0/1 switchport
SW1# show mac address-table
```

### Expected Output of `show vlan brief`:
```
VLAN  Name       Status    Ports
----  ---------  --------  ---------------------------
1     default    active    Fa0/5, Fa0/6 ...
10    Sales      active    Fa0/1, Fa0/2
20    HR         active    Fa0/3, Fa0/4
```

---

## Testing Connectivity

### Within Same VLAN (Should SUCCEED ✅)
```
PC1> ping 192.168.10.2    → Success (PC1 to PC2, same VLAN 10)
PC3> ping 192.168.20.2    → Success (PC3 to PC4, same VLAN 20)
```

### Across Different VLANs (Should FAIL ❌)
```
PC1> ping 192.168.20.1    → Fail (PC1 to PC3, different VLANs)
```

---

## Result
VLANs were successfully configured on the Cisco switch. Devices within the same VLAN communicated successfully, while devices in different VLANs were isolated, demonstrating effective traffic segmentation.

---

## Conclusion
This practical demonstrates how VLANs can be used to logically segment a network, improve security, and reduce broadcast traffic without requiring separate physical hardware.
