# Practical 7: Installation of NS2 and Basic Node Simulation

## Objective
Install NS2 (Network Simulator 2) and create a basic network with two nodes to simulate data transfer between them.

---

## Theory
NS2 (Network Simulator version 2) is an open-source, discrete event-driven network simulator widely used for research and education. It supports simulation of TCP, UDP, routing, and multicast protocols over wired and wireless networks.

**Key Components:**
- **Tcl/OTcl** – Scripting language used to write NS2 simulation scripts
- **NAM** – Network Animator used to visualize simulations
- **Trace files (.tr)** – Log files for analyzing simulation results

---

## Equipment / Tools Required
- Linux OS (Ubuntu 16.04 / 18.04 recommended)
- Terminal with sudo access
- NS2 package (ns-allinone-2.35)

---

## Part A: Installation of NS2

### Step 1: Update System Packages
```bash
sudo apt-get update
sudo apt-get upgrade -y
```

### Step 2: Install Dependencies
```bash
sudo apt-get install -y build-essential tcl8.5 tcl8.5-dev \
tk8.5 tk8.5-dev libxmu-dev libxmuu-dev gcc g++ libxt-dev \
libx11-dev xorg-dev
```

### Step 3: Download NS2
```bash
cd ~
wget https://sourceforge.net/projects/nsnam/files/allinone/ns-allinone-2.35/ns-allinone-2.35.tar.gz
```

### Step 4: Extract the Archive
```bash
tar -xzvf ns-allinone-2.35.tar.gz
cd ns-allinone-2.35
```

### Step 5: Install NS2
```bash
./install
```
> This process takes 5–15 minutes. Wait for completion.

### Step 6: Set Environment Variables
```bash
nano ~/.bashrc
```

Add the following lines at the end:
```bash
export PATH=$PATH:/home/$USER/ns-allinone-2.35/bin
export PATH=$PATH:/home/$USER/ns-allinone-2.35/tcl8.5.10/unix
export PATH=$PATH:/home/$USER/ns-allinone-2.35/tk8.5.10/unix
export LD_LIBRARY_PATH=/home/$USER/ns-allinone-2.35/otcl-1.14
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/$USER/ns-allinone-2.35/lib
```

Apply changes:
```bash
source ~/.bashrc
```

### Step 7: Verify Installation
```bash
ns
```
Expected output:
```
%
```
Type `exit` to quit.

---

## Part B: Basic Two-Node Simulation (Data Transfer)

### Simulation Script: `basic_transfer.tcl`

```tcl
# =============================================
# Practical 7: Basic Two-Node Data Transfer
# =============================================

# Create a new simulator instance
set ns [new Simulator]

# Open trace file for analysis
set tracefile [open basic.tr w]
$ns trace-all $tracefile

# Open NAM (Network Animator) file
set namfile [open basic.nam w]
$ns namtrace-all $namfile

# Define finish procedure
proc finish {} {
    global ns tracefile namfile
    $ns flush-trace
    close $tracefile
    close $namfile
    exec nam basic.nam &
    exit 0
}

# ---- Create Two Nodes ----
set n0 [$ns node]
set n1 [$ns node]

# ---- Create a Duplex Link Between Nodes ----
# Bandwidth: 1Mb, Delay: 10ms, Queue: DropTail
$ns duplex-link $n0 $n1 1Mb 10ms DropTail

# ---- Set Up UDP Agent on Node 0 ----
set udp [new Agent/UDP]
$ns attach-agent $n0 $udp

# ---- Set Up CBR Traffic Source ----
set cbr [new Application/Traffic/CBR]
$cbr set packetSize_ 500
$cbr set interval_ 0.005
$cbr attach-agent $udp

# ---- Set Up Null Agent (Sink) on Node 1 ----
set null [new Agent/Null]
$ns attach-agent $n1 $null

# ---- Connect UDP Agent to Null Agent ----
$ns connect $udp $null

# ---- Schedule Events ----
$ns at 0.5 "$cbr start"
$ns at 4.5 "$cbr stop"
$ns at 5.0 "finish"

# ---- Run Simulation ----
$ns run
```

### Run the Simulation:
```bash
ns basic_transfer.tcl
```

> NAM window will open showing the two nodes with animated packet transfer.

---

## Trace File Analysis

After simulation, view the trace file:
```bash
cat basic.tr
```

Each line format:
```
[event] [time] [from_node] [to_node] [type] [size] [flags] [fid] [src] [dst] [seq] [pkt_id]
```

**Event codes:**
| Code | Meaning |
|------|---------|
| `+` | Packet enqueued |
| `-` | Packet dequeued |
| `r` | Packet received |
| `d` | Packet dropped |

---

## NAM Visualization
- **Blue circles** = Nodes (n0 and n1)
- **Lines** = Links between nodes
- **Moving packets** = Data being transferred
- Use **Play / Step / Stop** controls to observe the simulation

---

## Result
NS2 was successfully installed. A two-node simulation was created where UDP/CBR traffic was transmitted from Node 0 to Node 1. Packet transfer was observed in NAM and trace logs were generated for analysis.

---

## Conclusion
This practical introduces NS2 as a powerful tool for network simulation. The basic two-node setup demonstrates how to model data transfer, analyze trace files, and visualize network behavior using NAM.

