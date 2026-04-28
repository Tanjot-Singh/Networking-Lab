# Practical 5: Setting Up a DHCP Server

## Objective
Configure a DHCP server on a Cisco router to dynamically assign IP addresses to network devices.

---

## Theory
DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses and other network parameters (subnet mask, default gateway, DNS server) to client devices. This eliminates the need for manual IP configuration.

**DHCP Process (DORA):**
1. **Discover** – Client broadcasts to find a DHCP server.
2. **Offer** – Server offers an IP address.
3. **Request** – Client requests the offered IP.
4. **Acknowledge** – Server confirms the assignment.

---

## Equipment / Tools Required
- Cisco Packet Tracer
- 1 Cisco Router (acting as DHCP server)
- 1 Cisco Switch
- 3 PCs (set to DHCP mode)

---

## Network Topology

```
         [Router R1]
         Fa0/0: 192.168.1.1
              |
         [Switch SW1]
        /     |     \
      PC1    PC2    PC3
   (DHCP)  (DHCP)  (DHCP)
```

---

## Configuration

### Step 1: Configure Router Interface
```
Router> enable
Router# configure terminal
Router(config)# hostname R1

R1(config)# interface FastEthernet 0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

### Step 2: Exclude Static IP Addresses (Reserved IPs)
```
R1(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
```
> This reserves .1 to .10 for static devices (router, server, printer, etc.)

### Step 3: Create DHCP Pool
```
R1(config)# ip dhcp pool LAN_POOL
R1(dhcp-config)# network 192.168.1.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.1.1
R1(dhcp-config)# dns-server 8.8.8.8
R1(dhcp-config)# lease 7
R1(dhcp-config)# exit
```

> **Parameters explained:**
> - `network` – The pool's subnet
> - `default-router` – Gateway to assign to clients
> - `dns-server` – DNS server IP
> - `lease 7` – IP lease duration in days (default is 1 day)

### Step 4: Save Configuration
```
R1# write memory
```

---

## Optional: DHCP for Multiple VLANs / Subnets

```
R1(config)# ip dhcp excluded-address 192.168.2.1 192.168.2.10

R1(config)# ip dhcp pool VLAN20_POOL
R1(dhcp-config)# network 192.168.2.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.2.1
R1(dhcp-config)# dns-server 8.8.8.8
R1(dhcp-config)# lease 7
R1(dhcp-config)# exit
```

---

## Optional: DHCP Relay Agent
If the DHCP server is on a different subnet, configure an IP helper:
```
R1(config)# interface FastEthernet 0/1
R1(config-if)# ip helper-address 192.168.1.1
R1(config-if)# exit
```

---

## Client Configuration (Packet Tracer)
1. Click on PC → Desktop → IP Configuration
2. Select **DHCP** radio button
3. The PC will automatically receive an IP address

---

## Verification Commands

```
R1# show ip dhcp pool
R1# show ip dhcp binding
R1# show ip dhcp server statistics
R1# show running-config | section dhcp
```

### Expected Output of `show ip dhcp binding`:
```
IP address       Client-ID/            Lease expiration        Type
                 Hardware address
192.168.1.11     0060.2FFF.6A91        --                      Automatic
192.168.1.12     00E0.A3CC.1234        --                      Automatic
192.168.1.13     00D0.BC45.5678        --                      Automatic
```

---

## Testing

From any PC in Packet Tracer:
1. Set IP config to **DHCP**
2. Verify that an IP in range 192.168.1.11 – 192.168.1.254 is assigned
3. Ping the default gateway:
```
PC> ping 192.168.1.1    → Should SUCCEED ✅
```
4. Ping another PC:
```
PC1> ping 192.168.1.12  → Should SUCCEED ✅
```

---

## Result
The DHCP server was successfully configured on the Cisco router. All PCs automatically received IP addresses, subnet masks, default gateway, and DNS server information from the DHCP pool.

---

## Conclusion
This practical demonstrates how a Cisco router can function as a DHCP server to automate IP address assignment in a network, reducing administrative overhead and the risk of IP conflicts.
