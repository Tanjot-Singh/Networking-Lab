# Practical 3: Inter-VLAN Routing

## Objective
Enable communication between VLANs using a router (Router-on-a-Stick method) or a Layer 3 switch.

---

## Theory
Inter-VLAN routing allows devices in different VLANs to communicate with each other. Since VLANs are isolated broadcast domains, a Layer 3 device (router or L3 switch) is needed to route traffic between them.

**Two common methods:**
1. **Router-on-a-Stick** – Uses a single router interface with sub-interfaces for each VLAN.
2. **Layer 3 Switch** – Uses Switched Virtual Interfaces (SVIs) for routing.

This practical uses the **Router-on-a-Stick** method.

---

## Equipment / Tools Required
- Cisco Packet Tracer
- 1 Cisco Router (1841 or similar)
- 1 Cisco Switch (2960 or similar)
- 4 PCs

---

## Network Topology

```
         [Router R1]
              |
           Fa0/0 (Trunk)
              |
         [Switch SW1]
        /    |    \     \
     Fa0/1 Fa0/2 Fa0/3 Fa0/4
       |     |     |     |
      PC1   PC2   PC3   PC4
    VLAN10 VLAN10 VLAN20 VLAN20
```

---

## IP Addressing Table

| Device        | VLAN    | IP Address       | Subnet Mask   | Gateway        |
|---------------|---------|------------------|---------------|----------------|
| PC1           | VLAN 10 | 192.168.10.2     | 255.255.255.0 | 192.168.10.1   |
| PC2           | VLAN 10 | 192.168.10.3     | 255.255.255.0 | 192.168.10.1   |
| PC3           | VLAN 20 | 192.168.20.2     | 255.255.255.0 | 192.168.20.1   |
| PC4           | VLAN 20 | 192.168.20.3     | 255.255.255.0 | 192.168.20.1   |
| R1 Fa0/0.10   | VLAN 10 | 192.168.10.1     | 255.255.255.0 | —              |
| R1 Fa0/0.20   | VLAN 20 | 192.168.20.1     | 255.255.255.0 | —              |

---

## Configuration

### Part A: Switch Configuration

#### Step 1: Create VLANs on Switch
```
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1

SW1(config)# vlan 10
SW1(config-vlan)# name Sales
SW1(config-vlan)# exit

SW1(config)# vlan 20
SW1(config-vlan)# name HR
SW1(config-vlan)# exit
```

#### Step 2: Assign Access Ports
```
SW1(config)# interface FastEthernet 0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit

SW1(config)# interface FastEthernet 0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# exit

SW1(config)# interface FastEthernet 0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# exit

SW1(config)# interface FastEthernet 0/4
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# exit
```

#### Step 3: Configure Trunk Port (Uplink to Router)
```
SW1(config)# interface FastEthernet 0/24
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,20
SW1(config-if)# exit

SW1# write memory
```

---

### Part B: Router Configuration (Router-on-a-Stick)

#### Step 1: Enable Physical Interface
```
Router> enable
Router# configure terminal
Router(config)# hostname R1

R1(config)# interface FastEthernet 0/0
R1(config-if)# no shutdown
R1(config-if)# exit
```

#### Step 2: Create Sub-interface for VLAN 10
```
R1(config)# interface FastEthernet 0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit
```

#### Step 3: Create Sub-interface for VLAN 20
```
R1(config)# interface FastEthernet 0/0.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# exit

R1# write memory
```

---

## Verification Commands

```
R1# show ip interface brief
R1# show ip route
SW1# show vlan brief
SW1# show interfaces trunk
```

### Expected Output of `show ip route` on R1:
```
C    192.168.10.0/24 is directly connected, FastEthernet0/0.10
C    192.168.20.0/24 is directly connected, FastEthernet0/0.20
```

---

## Testing Connectivity

### Same VLAN (Should SUCCEED ✅)
```
PC1> ping 192.168.10.3
```

### Different VLAN via Router (Should SUCCEED ✅)
```
PC1> ping 192.168.20.2
PC3> ping 192.168.10.2
```

---

## Result
Inter-VLAN routing was successfully implemented using the Router-on-a-Stick method. PCs in different VLANs were able to communicate through the router's sub-interfaces.

---

## Conclusion
This practical demonstrates how a single router interface with IEEE 802.1Q sub-interfaces can route traffic between multiple VLANs, enabling inter-VLAN communication without requiring separate physical router interfaces.
