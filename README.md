# GNS3 Lab: High Availability with VRRP, Object Tracking, and NAT

This lab demonstrates a redundant gateway architecture using **Virtual Router Redundancy Protocol (VRRP)**. It features stateful failover logic where the primary router monitors its uplink (WAN) status and yields control to the backup router if the internet connection is lost.

## 🌐 Topology Architecture

The network is designed with a dual-homed gateway approach:

* **Internal LAN:** `192.168.10.0/24` with a Virtual IP (VIP) of `.254`.
* **External WAN:** `192.168.134.0/24` (in my case) connecting to a simulated Internet/NAT cloud.
* **Redundancy:** R1 is the **Master** (Priority 120), and R2 is the **Backup** (Priority 100).

## 🚀 Key Features

### 1. VRRP Priority & Preemption

R1 is configured with a higher priority to ensure it is the preferred path. **Preemption** is enabled by default so that if R1 reboots, it will reclaim the Master role once it is back online.

### 2. Interface Tracking (The "Smart" Failover)

Unlike basic VRRP which only fails if the router dies, this lab uses **Object Tracking**:

* R1 monitors its WAN interface (`g1/0`).
* If `g1/0` goes down, R1 automatically decrements its priority by **30**.
* This drops R1's priority to **90** (lower than R2’s 100), triggering an immediate failover so traffic doesn't hit a "black hole."

### 3. Redundant PAT (Port Address Translation)

Both routers are configured with NAT Overload. Since the PC points to the VIP (`.254`), only the router currently acting as the VRRP Master will process and translate the traffic.

## 🛠️ Configuration Details

### Router R1 (Master)

```bash
interface g2/0
 ip address 192.168.10.1 255.255.255.0
 vrrp 10 ip 192.168.10.254
 vrrp 10 priority 120
 ip nat inside

interface g1/0
 ip address 192.168.134.5 255.255.255.0
 ip nat outside

```

### Router R2 (Backup)

```bash
interface g2/0
 ip address 192.168.10.2 255.255.255.0
 vrrp 10 ip 192.168.10.254
 vrrp 10 priority 100
 ip nat inside

interface g1/0
 ip address 192.168.134.6 255.255.255.0
 ip nat outside

```

---

## 🧪 Verification & Testing

### 1. Check VRRP Status

Run this on both routers to see who is Master and who is Backup:

```bash
show vrrp brief

```

### 2. Test Failover (The "Uplink Failure" Scenario)

1. Start a continuous ping from **PC-1** to `8.8.8.8`.
2. On **R1**, shut down the WAN interface:
```bash
R1(config)# interface g1/0
R1(config-if)# shutdown

```


3. **Observe:** R1's priority drops, R2 becomes Master, and the ping continues with minimal packet loss.

### 3. Verify NAT Translations

Check which router is currently translating traffic:

```bash
show ip nat translations

```

---

## 📂 Files in this Repo

* `VRRP_Lab.gns3`: The project file.
* `configs/`: Directory containing startup-configs for R1, R2, and Switches.
* `topology.png`: Visual representation of the lab.

---

**Would you like me to add a "Troubleshooting" section to this README, or does this cover everything you need for the GitHub upload?**
