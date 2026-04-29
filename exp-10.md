# Practical 10: Simulate and Analyze Network Security Protocols (IPsec) in NS2

## Objective
Simulate and analyze the performance of network security protocols, specifically IPsec, using NS2. Compare network behavior with and without IPsec in terms of throughput, delay, and overhead.

---

## Theory

### What is IPsec?
IPsec (Internet Protocol Security) is a suite of protocols that secures IP communication by authenticating and encrypting each IP packet in a communication session.

### IPsec Components:

| Component | Full Form | Function |
|-----------|-----------|----------|
| **AH** | Authentication Header | Provides integrity and authentication (no encryption) |
| **ESP** | Encapsulating Security Payload | Provides integrity, authentication, AND encryption |
| **IKE** | Internet Key Exchange | Negotiates security keys and associations |
| **SA** | Security Association | A one-way relationship defining security parameters |

### IPsec Modes:

| Mode | Description | Use Case |
|------|-------------|----------|
| **Transport Mode** | Encrypts only the payload; original IP header intact | Host-to-host communication |
| **Tunnel Mode** | Encrypts entire original packet; new IP header added | VPN gateway-to-gateway |

### IPsec in NS2:
NS2 does not have a built-in IPsec module. We simulate IPsec behavior by:
- Adding **packet overhead** to simulate encryption headers (AH adds ~24 bytes, ESP adds ~50+ bytes)
- Using **smaller packet sizes** to simulate processing delay
- Comparing throughput and delay **with overhead vs without overhead**

---

## Equipment / Tools Required
- Linux OS with NS2 installed (ns-allinone-2.35)
- Terminal
- Text editor
- AWK (for trace analysis)
- GNUplot (optional, for graphs)

---

## Network Topology

```
Node 0 (Sender) ──── Node 1 (Router/Gateway) ──── Node 2 (Receiver)

Without IPsec: Normal UDP packets (512 bytes)
With IPsec:    UDP packets with ESP overhead (~562 bytes = 512 + 50 byte ESP header)
```

---

## Part A: Simulation WITHOUT IPsec

### Script: `without_ipsec.tcl`

```tcl
# =============================================
# Practical 10: Simulation WITHOUT IPsec
# =============================================

set ns [new Simulator]

$ns color 1 Blue

set tracefile [open without_ipsec.tr w]
$ns trace-all $tracefile

set namfile [open without_ipsec.nam w]
$ns namtrace-all $namfile

proc finish {} {
    global ns tracefile namfile
    $ns flush-trace
    close $tracefile
    close $namfile
    exec nam without_ipsec.nam &
    exit 0
}

# Create Nodes
set n0 [$ns node]    ;# Sender
set n1 [$ns node]    ;# Gateway/Router
set n2 [$ns node]    ;# Receiver

# Create Links
$ns duplex-link $n0 $n1 1Mb 10ms DropTail
$ns duplex-link $n1 $n2 1Mb 10ms DropTail

# Queue limit
$ns queue-limit $n1 $n2 20

# NAM layout
$ns duplex-link-op $n0 $n1 orient right
$ns duplex-link-op $n1 $n2 orient right

# ---- UDP Agent (Normal, no IPsec overhead) ----
set udp [new Agent/UDP]
$udp set class_ 1
$ns attach-agent $n0 $udp

set null [new Agent/Null]
$ns attach-agent $n2 $null
$ns connect $udp $null

# ---- CBR Traffic (Standard 512-byte packets) ----
set cbr [new Application/Traffic/CBR]
$cbr set packetSize_ 512
$cbr set interval_ 0.01     ;# 100 packets/sec
$cbr attach-agent $udp

# Schedule
$ns at 0.5 "$cbr start"
$ns at 9.5 "$cbr stop"
$ns at 10.0 "finish"

$ns run
```

---

## Part B: Simulation WITH IPsec (ESP Overhead)

### Script: `with_ipsec.tcl`

```tcl
# =============================================
# Practical 10: Simulation WITH IPsec (ESP)
# =============================================

set ns [new Simulator]

$ns color 2 Red

set tracefile [open with_ipsec.tr w]
$ns trace-all $tracefile

set namfile [open with_ipsec.nam w]
$ns namtrace-all $namfile

proc finish {} {
    global ns tracefile namfile
    $ns flush-trace
    close $tracefile
    close $namfile
    exec nam with_ipsec.nam &
    exit 0
}

# Create Nodes
set n0 [$ns node]    ;# Sender (IPsec enabled)
set n1 [$ns node]    ;# Gateway (IPsec endpoint)
set n2 [$ns node]    ;# Receiver

# Create Links
$ns duplex-link $n0 $n1 1Mb 10ms DropTail
$ns duplex-link $n1 $n2 1Mb 10ms DropTail

$ns queue-limit $n1 $n2 20

$ns duplex-link-op $n0 $n1 orient right
$ns duplex-link-op $n1 $n2 orient right

# ---- UDP Agent with IPsec ESP overhead ----
# ESP Header overhead = ~50 bytes
# New packet size = 512 + 50 = 562 bytes
set udp [new Agent/UDP]
$udp set class_ 2
$ns attach-agent $n0 $udp

set null [new Agent/Null]
$ns attach-agent $n2 $null
$ns connect $udp $null

# ---- CBR Traffic with IPsec overhead ----
set cbr [new Application/Traffic/CBR]
$cbr set packetSize_ 562     ;# 512 data + 50 ESP overhead
$cbr set interval_ 0.012     ;# Slightly slower due to encryption processing delay
$cbr attach-agent $udp

# Schedule
$ns at 0.5 "$cbr start"
$ns at 9.5 "$cbr stop"
$ns at 10.0 "finish"

$ns run
```

### Run Both Simulations:
```bash
ns without_ipsec.tcl
ns with_ipsec.tcl
```

---

## Part C: Combined Comparison Script

### Script: `ipsec_compare.tcl`

```tcl
# =============================================
# Practical 10: IPsec Comparison (Combined)
# =============================================

set ns [new Simulator]

$ns color 1 Blue    ;# Normal traffic
$ns color 2 Red     ;# IPsec traffic

set tracefile [open ipsec_compare.tr w]
$ns trace-all $tracefile

set namfile [open ipsec_compare.nam w]
$ns namtrace-all $namfile

proc finish {} {
    global ns tracefile namfile
    $ns flush-trace
    close $tracefile
    close $namfile
    exec nam ipsec_compare.nam &
    exit 0
}

# ---- Nodes ----
set n0 [$ns node]    ;# Normal sender
set n1 [$ns node]    ;# IPsec sender
set n2 [$ns node]    ;# Shared router
set n3 [$ns node]    ;# Normal receiver
set n4 [$ns node]    ;# IPsec receiver

# ---- Links ----
$ns duplex-link $n0 $n2 1Mb 10ms DropTail
$ns duplex-link $n1 $n2 1Mb 10ms DropTail
$ns duplex-link $n2 $n3 1Mb 10ms DropTail
$ns duplex-link $n2 $n4 1Mb 10ms DropTail

$ns queue-limit $n2 $n3 20
$ns queue-limit $n2 $n4 20

# NAM layout
$ns duplex-link-op $n0 $n2 orient right-up
$ns duplex-link-op $n1 $n2 orient right-down
$ns duplex-link-op $n2 $n3 orient right-up
$ns duplex-link-op $n2 $n4 orient right-down

# ---- Flow 1: Normal UDP (No IPsec) ----
set udp_normal [new Agent/UDP]
$udp_normal set class_ 1
$ns attach-agent $n0 $udp_normal

set null_normal [new Agent/Null]
$ns attach-agent $n3 $null_normal
$ns connect $udp_normal $null_normal

set cbr_normal [new Application/Traffic/CBR]
$cbr_normal set packetSize_ 512
$cbr_normal set interval_ 0.01
$cbr_normal attach-agent $udp_normal

# ---- Flow 2: IPsec UDP (with ESP overhead) ----
set udp_ipsec [new Agent/UDP]
$udp_ipsec set class_ 2
$ns attach-agent $n1 $udp_ipsec

set null_ipsec [new Agent/Null]
$ns attach-agent $n4 $null_ipsec
$ns connect $udp_ipsec $null_ipsec

set cbr_ipsec [new Application/Traffic/CBR]
$cbr_ipsec set packetSize_ 562      ;# ESP overhead added
$cbr_ipsec set interval_ 0.012      ;# Processing delay simulation
$cbr_ipsec attach-agent $udp_ipsec

# ---- Schedule ----
$ns at 0.5 "$cbr_normal start"
$ns at 0.5 "$cbr_ipsec start"
$ns at 9.5 "$cbr_normal stop"
$ns at 9.5 "$cbr_ipsec stop"
$ns at 10.0 "finish"

$ns run
```

```bash
ns ipsec_compare.tcl
```

---

## Trace File Analysis

### Throughput of Normal Traffic (Flow 1):
```bash
awk 'BEGIN{bytes=0}
/^r/ && $8==1 { bytes += $6 }
END { print "Normal Throughput: " bytes*8/9 " bps" }' ipsec_compare.tr
```

### Throughput of IPsec Traffic (Flow 2):
```bash
awk 'BEGIN{bytes=0}
/^r/ && $8==2 { bytes += $6 }
END { print "IPsec Throughput: " bytes*8/9 " bps" }' ipsec_compare.tr
```

### Average End-to-End Delay:
```bash
awk '/^r/ && $8==1 {
    split($4, dst, "")
    recv[$11] = $2
}
/^+/ && $8==1 {
    send[$11] = $2
}
END {
    for (p in recv) {
        if (p in send) {
            delay += recv[p] - send[p]; count++
        }
    }
    if (count > 0) print "Avg Delay (Normal): " delay/count " sec"
}' ipsec_compare.tr
```

### Count Dropped Packets:
```bash
echo "Normal drops:"
grep "^d" ipsec_compare.tr | awk '$8==1' | wc -l

echo "IPsec drops:"
grep "^d" ipsec_compare.tr | awk '$8==2' | wc -l
```

---

## Performance Comparison Table

| Metric | Without IPsec | With IPsec (ESP) |
|--------|--------------|-----------------|
| Packet size | 512 bytes | 562 bytes (+50 overhead) |
| Send interval | 0.01 sec | 0.012 sec |
| Throughput | Higher | Slightly lower |
| End-to-end delay | Lower | Higher (encryption delay) |
| Packet drops | Fewer | Slightly more (larger packets) |
| Security | None | Authentication + Encryption |

---

## IPsec Header Overhead Reference

| Protocol | Mode | Overhead Added |
|----------|------|---------------|
| AH | Transport | ~24 bytes |
| AH | Tunnel | ~24 + 20 bytes (new IP header) |
| ESP | Transport | ~50 bytes (header + trailer + IV) |
| ESP | Tunnel | ~70+ bytes (header + trailer + new IP header) |

---

## NAM Visualization
- **Blue packets (n0→n3)** = Normal traffic, smaller, faster
- **Red packets (n1→n4)** = IPsec traffic, slightly larger, slightly slower
- Compare packet sizes and transmission frequency visually in NAM

---

## Result
Two simulations were run — one with normal UDP traffic and one simulating IPsec/ESP overhead. Analysis of trace files confirmed that IPsec adds packet overhead and slight delay but provides security. Throughput was marginally lower with IPsec due to larger packet sizes and increased processing interval.

---

## Conclusion
This practical demonstrates the performance trade-off introduced by IPsec. While IPsec (especially ESP in Tunnel Mode) adds overhead in terms of packet size and processing delay, it is essential for securing data in transit. The simulation shows that the performance impact is acceptable for the security benefits provided, which is why IPsec is widely used in VPNs and secure enterprise networks.
