# Practical 8: Basic Network Topology Simulation in NS2

## Objective
Set up and simulate a basic network topology in NS2 to understand the fundamental components, including nodes, links, agents, traffic sources, and the NAM visualizer.

---

## Theory
NS2 uses an event-driven simulation model. A typical NS2 script involves:

| Component | Description |
|-----------|-------------|
| **Simulator** | Core NS2 object managing simulation events |
| **Nodes** | Represent network devices (routers, hosts) |
| **Links** | Connect nodes with configurable bandwidth, delay, queue type |
| **Agents** | Represent transport protocols (TCP, UDP) |
| **Applications** | Generate traffic (CBR, FTP, Telnet) |
| **Trace file** | Records all events for post-analysis |
| **NAM file** | Used by the Network Animator for visualization |

---

## Equipment / Tools Required
- Linux OS with NS2 installed (ns-allinone-2.35)
- Terminal

---

## Network Topology (Star Topology with 4 Nodes)

```
         n1
          |
    n0 - n2 - n3
          |
         n4
```

Central node: **n2** (acts as router)
All other nodes connect to n2.

---

## Simulation Script: `topology.tcl`

```tcl
# =============================================
# Practical 8: Basic Network Topology in NS2
# =============================================

# Step 1: Create Simulator Object
set ns [new Simulator]

# Step 2: Define Colors for Different Flows (NAM)
$ns color 1 Blue
$ns color 2 Red

# Step 3: Open Trace and NAM Files
set tracefile [open topology.tr w]
$ns trace-all $tracefile

set namfile [open topology.nam w]
$ns namtrace-all $namfile

# Step 4: Finish Procedure
proc finish {} {
    global ns tracefile namfile
    $ns flush-trace
    close $tracefile
    close $namfile
    exec nam topology.nam &
    exit 0
}

# Step 5: Create Nodes
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]    ;# Central/Router Node
set n3 [$ns node]
set n4 [$ns node]

# Step 6: Create Links (Star Topology around n2)
$ns duplex-link $n0 $n2 2Mb 10ms DropTail
$ns duplex-link $n1 $n2 2Mb 10ms DropTail
$ns duplex-link $n2 $n3 1Mb 20ms DropTail
$ns duplex-link $n2 $n4 1Mb 20ms DropTail

# Step 7: Set Node Positions for NAM Display
$ns duplex-link-op $n0 $n2 orient right-down
$ns duplex-link-op $n1 $n2 orient right-up
$ns duplex-link-op $n2 $n3 orient right
$ns duplex-link-op $n2 $n4 orient right-down

# Step 8: Set Queue Size on Links
$ns queue-limit $n2 $n3 20
$ns queue-limit $n2 $n4 20

# -------- Flow 1: UDP from n0 to n3 --------
set udp0 [new Agent/UDP]
$udp0 set class_ 1
$ns attach-agent $n0 $udp0

set null3 [new Agent/Null]
$ns attach-agent $n3 $null3
$ns connect $udp0 $null3

set cbr0 [new Application/Traffic/CBR]
$cbr0 set packetSize_ 500
$cbr0 set interval_ 0.01
$cbr0 attach-agent $udp0

# -------- Flow 2: TCP from n1 to n4 --------
set tcp1 [new Agent/TCP]
$tcp1 set class_ 2
$ns attach-agent $n1 $tcp1

set sink4 [new Agent/TCPSink]
$ns attach-agent $n4 $sink4
$ns connect $tcp1 $sink4

set ftp1 [new Application/FTP]
$ftp1 attach-agent $tcp1

# -------- Schedule Events --------
$ns at 0.5  "$cbr0 start"
$ns at 0.5  "$ftp1 start"
$ns at 4.0  "$cbr0 stop"
$ns at 4.0  "$ftp1 stop"
$ns at 5.0  "finish"

# Step 9: Run the Simulation
$ns run
```

### Run the Simulation:
```bash
ns topology.tcl
```

---

## Understanding the Output

### NAM Visualization:
- **Circles** = Nodes (n0–n4)
- **Lines** = Duplex links with direction arrows
- **Blue packets** = UDP/CBR flow from n0 → n2 → n3
- **Red packets** = TCP/FTP flow from n1 → n2 → n4

### Trace File Format (`topology.tr`):
```
r 0.5108 2 3 cbr 500 ------- 1 0.0 3.0 0 0
r 0.5116 2 4 tcp 1000 ------- 2 1.0 4.0 0 0
```

---

## Analyzing the Trace File

### Count total packets received:
```bash
grep "^r" topology.tr | wc -l
```

### Count dropped packets:
```bash
grep "^d" topology.tr | wc -l
```

### Filter UDP packets only:
```bash
grep "cbr" topology.tr
```

### Calculate Throughput (using AWK):
```bash
awk 'BEGIN { bytes=0 }
/^r/ && /cbr/ { bytes += $8 }
END { print "Throughput: " bytes*8/4 " bps" }' topology.tr
```

---

## Key NS2 Concepts Demonstrated

| Concept | Implementation |
|---------|---------------|
| Multi-node topology | 5 nodes in star configuration |
| Multiple traffic flows | UDP-CBR and TCP-FTP simultaneously |
| Link properties | Bandwidth (1–2 Mb), delay (10–20 ms) |
| Queue management | DropTail with limit of 20 packets |
| Flow differentiation | Color-coded flows (Blue = UDP, Red = TCP) |
| Trace analysis | AWK scripts for throughput calculation |

---

## Result
A basic 5-node network topology was successfully simulated in NS2. Two traffic flows (UDP/CBR and TCP/FTP) ran concurrently through the central routing node. Packet transfer was visualized in NAM and analyzed using trace file AWK scripts.

---

## Conclusion
This practical demonstrates the fundamental components of NS2 simulation including node creation, link configuration, agent attachment, traffic generation, and result visualization. Understanding these building blocks is essential for more complex network simulations.
