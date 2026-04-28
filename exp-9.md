# Practical 9: Simulating Network Attacks (DoS) in NS2

## Objective
Simulate a Denial of Service (DoS) attack in NS2 and analyze its effect on legitimate network traffic and overall network performance.

---

## Theory

### What is a DoS Attack?
A **Denial of Service (DoS)** attack floods a target node with excessive traffic, consuming its bandwidth and resources so that legitimate users cannot access the service.

### Types of DoS Simulated in NS2:
| Attack Type | Description |
|-------------|-------------|
| **Flooding Attack** | Attacker sends high-rate UDP packets to overwhelm the victim |
| **SYN Flood** | Attacker sends many TCP SYN packets without completing handshake |
| **Traffic Saturation** | Attacker saturates the link bandwidth |

### Key Metrics to Observe:
- **Packet Drop Rate** – Legitimate packets dropped due to congestion
- **Throughput** – Reduction in legitimate traffic throughput
- **End-to-End Delay** – Increased delay due to queue buildup

---

## Equipment / Tools Required
- Linux OS with NS2 installed
- Terminal
- Text editor (gedit / nano / vim)

---

## Network Topology

```
Legitimate Client (n0) ─────────────────────────┐
                                                  ├── Router (n2) ── Victim Server (n3)
Attacker Node   (n1) ──── Attack Traffic ────────┘
```

---

## Simulation Script: `dos_attack.tcl`

```tcl
# =============================================
# Practical 9: DoS Attack Simulation in NS2
# =============================================

set ns [new Simulator]

# Color coding for flows
$ns color 1 Blue    ;# Legitimate traffic
$ns color 2 Red     ;# Attack traffic

# Trace and NAM files
set tracefile [open dos.tr w]
$ns trace-all $tracefile

set namfile [open dos.nam w]
$ns namtrace-all $namfile

proc finish {} {
    global ns tracefile namfile
    $ns flush-trace
    close $tracefile
    close $namfile
    exec nam dos.nam &
    exit 0
}

# ---- Create Nodes ----
set n0 [$ns node]    ;# Legitimate Client
set n1 [$ns node]    ;# Attacker
set n2 [$ns node]    ;# Router
set n3 [$ns node]    ;# Victim Server

# ---- Create Links ----
# Normal links (1Mb)
$ns duplex-link $n0 $n2 1Mb 10ms DropTail
$ns duplex-link $n1 $n2 1Mb 5ms  DropTail
$ns duplex-link $n2 $n3 1Mb 10ms DropTail

# Bottleneck queue at router (limited to 10 packets)
$ns queue-limit $n2 $n3 10

# NAM layout
$ns duplex-link-op $n0 $n2 orient right-up
$ns duplex-link-op $n1 $n2 orient right-down
$ns duplex-link-op $n2 $n3 orient right

# ========================================
# Legitimate Traffic: UDP/CBR from n0→n3
# ========================================
set udp_legit [new Agent/UDP]
$udp_legit set class_ 1
$ns attach-agent $n0 $udp_legit

set null_legit [new Agent/Null]
$ns attach-agent $n3 $null_legit
$ns connect $udp_legit $null_legit

set cbr_legit [new Application/Traffic/CBR]
$cbr_legit set packetSize_ 512
$cbr_legit set interval_ 0.02     ;# 50 packets/sec = ~200Kbps
$cbr_legit attach-agent $udp_legit

# ========================================
# Attack Traffic: High-rate UDP from n1→n3
# ========================================
set udp_attack [new Agent/UDP]
$udp_attack set class_ 2
$ns attach-agent $n1 $udp_attack

set null_attack [new Agent/Null]
$ns attach-agent $n3 $null_attack
$ns connect $udp_attack $null_attack

set cbr_attack [new Application/Traffic/CBR]
$cbr_attack set packetSize_ 1000
$cbr_attack set interval_ 0.001   ;# 1000 packets/sec = ~8Mbps (flood)
$cbr_attack attach-agent $udp_attack

# ========================================
# Schedule Events
# ========================================
# Legitimate traffic runs throughout
$ns at 0.5  "$cbr_legit start"
$ns at 9.5  "$cbr_legit stop"

# Attack starts at t=2.0 and ends at t=7.0
$ns at 2.0  "$cbr_attack start"
$ns at 7.0  "$cbr_attack stop"

$ns at 10.0 "finish"

$ns run
```

### Run the Simulation:
```bash
ns dos_attack.tcl
```

---

## Trace File Analysis

### Count total received packets (all):
```bash
grep "^r" dos.tr | wc -l
```

### Count packets received at Victim (n3):
```bash
grep "^r" dos.tr | grep " 3 " | wc -l
```

### Count legitimate packets received (flow id 1):
```bash
grep "^r" dos.tr | awk '{if($8==1) count++} END{print "Legit received: " count}'
```

### Count dropped packets:
```bash
grep "^d" dos.tr | wc -l
```

### Throughput of legitimate traffic (AWK):
```bash
awk 'BEGIN{bytes=0; start=0.5; end=9.5}
/^r/ && /cbr/ && $8==1 {
    bytes += $6
}
END {
    throughput = (bytes * 8) / (end - start)
    print "Legitimate Throughput: " throughput " bps"
}' dos.tr
```

### Throughput during attack (t=2.0 to t=7.0) vs before (t=0.5 to t=2.0):
```bash
awk 'BEGIN{before=0; during=0}
/^r/ && $8==1 {
    if($2 >= 0.5 && $2 < 2.0) before += $6
    if($2 >= 2.0 && $2 < 7.0) during += $6
}
END {
    print "Throughput before attack: " before*8/1.5 " bps"
    print "Throughput during attack: " during*8/5.0 " bps"
}' dos.tr
```

---

## Expected Observations

| Metric | Before Attack | During Attack |
|--------|--------------|---------------|
| Legitimate throughput | ~200 Kbps | Significantly reduced |
| Packet drops | Near zero | High (queue overflow) |
| NAM visualization | Blue packets flow freely | Red flood; Blue packets drop |

---

## NAM Visualization Guide
- **Blue packets** = Legitimate traffic from n0
- **Red packets** = Attack flood traffic from n1
- Observe packet drops (X marks) at the n2→n3 link queue during the attack phase (t=2 to t=7)
- After t=7, attack stops and legitimate traffic recovers

---

## Result
The DoS attack simulation was successful. During the attack phase (t=2.0–7.0), the attacker's high-rate UDP flood saturated the bottleneck link, causing significant packet drops for legitimate traffic. After the attack stopped (t=7.0), normal communication resumed.

---

## Conclusion
This practical demonstrates how a DoS attack degrades network performance by flooding the network with excessive traffic. Trace file analysis quantified the throughput drop and packet loss experienced by legitimate users during the attack, highlighting the need for traffic filtering and rate-limiting mechanisms in real networks.
